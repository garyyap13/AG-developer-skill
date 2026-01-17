# Database Patterns

Patterns for Supabase/PostgreSQL development in OHCRM, with emphasis on triggers.

## Migration File Naming

Format: `YYYYMMDDHHMMSS_description.sql`

```
20260117120000_add_rewards_table.sql
20260117120001_add_rewards_rpc.sql
20260117120002_add_rewards_trigger.sql
```

Use sequential timestamps for related changes.

---

## Table Pattern

```sql
-- Create table with standard columns
CREATE TABLE IF NOT EXISTS public.examples (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    brand_id UUID NOT NULL REFERENCES public.brands(id) ON DELETE CASCADE,
    
    -- Domain columns
    name TEXT NOT NULL,
    status TEXT DEFAULT 'Active',
    amount NUMERIC(10,2) DEFAULT 0,
    
    -- Timestamps
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW(),
    created_by UUID REFERENCES auth.users(id)
);

-- Create updated_at trigger
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_examples_updated_at
    BEFORE UPDATE ON public.examples
    FOR EACH ROW
    EXECUTE FUNCTION update_updated_at_column();

-- RLS
ALTER TABLE public.examples ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users can view own brand examples"
    ON public.examples FOR SELECT
    USING (brand_id = auth.jwt() ->> 'brand_id'::text);

CREATE POLICY "Users can insert own brand examples"
    ON public.examples FOR INSERT
    WITH CHECK (brand_id = auth.jwt() ->> 'brand_id'::text);
```

---

## RPC Function Pattern

```sql
CREATE OR REPLACE FUNCTION public.get_example_stats(
    p_brand_id UUID
)
RETURNS TABLE (
    total_count BIGINT,
    active_count BIGINT,
    total_amount NUMERIC
)
LANGUAGE plpgsql
SECURITY DEFINER
SET search_path = public
AS $$
BEGIN
    RETURN QUERY
    SELECT 
        COUNT(*)::BIGINT AS total_count,
        COUNT(*) FILTER (WHERE status = 'Active')::BIGINT AS active_count,
        COALESCE(SUM(amount), 0) AS total_amount
    FROM examples
    WHERE brand_id = p_brand_id;
END;
$$;
```

### Calling from Frontend
```tsx
const { data, error } = await supabase.rpc('get_example_stats', {
  p_brand_id: brandId
});
```

---

## Trigger Pattern (CRITICAL)

Triggers automate cross-module business logic. Understand before modifying!

### Standard Trigger Template
```sql
-- 1. Create trigger function
CREATE OR REPLACE FUNCTION public.trigger_example_change()
RETURNS TRIGGER
LANGUAGE plpgsql
SECURITY DEFINER
AS $$
BEGIN
    -- INSERT
    IF TG_OP = 'INSERT' THEN
        -- Perform cross-module action
        PERFORM some_side_effect(NEW.id);
        RETURN NEW;
    END IF;
    
    -- UPDATE
    IF TG_OP = 'UPDATE' THEN
        -- Only act on relevant changes
        IF NEW.status IS DISTINCT FROM OLD.status THEN
            PERFORM some_side_effect(NEW.id);
        END IF;
        RETURN NEW;
    END IF;
    
    -- DELETE
    IF TG_OP = 'DELETE' THEN
        PERFORM cleanup_side_effect(OLD.id);
        RETURN OLD;
    END IF;
    
    RETURN NULL;
END;
$$;

-- 2. Create trigger
CREATE TRIGGER trg_example_change
    AFTER INSERT OR UPDATE OR DELETE ON public.examples
    FOR EACH ROW
    EXECUTE FUNCTION trigger_example_change();
```

### Important Triggers in OHCRM

| Trigger | Table | Purpose |
|---------|-------|---------|
| `trg_vip_on_order_change` | orders | Recalculates VIP on order status/total change |
| `trg_sync_product_stock` | warehouse_stock | Syncs warehouse → product stock |
| `trg_sync_bundle_stock` | bundle_components | Recalculates bundle availability |
| `trg_log_order_status` | orders | Logs status changes to history |

### Trigger Gotchas
1. **Triggers fire on ALL changes** - Add conditions to filter
2. **Recursive triggers** - Use `pg_trigger_depth()` to prevent
3. **Performance** - Keep trigger logic fast, defer heavy work
4. **Testing** - Triggers make manual testing tricky

---

## RLS Policy Pattern

OHCRM uses Row-Level Security for multi-tenant data isolation.

```sql
-- Basic brand isolation
CREATE POLICY "brand_isolation" ON public.examples
    FOR ALL
    USING (brand_id = (auth.jwt() ->> 'brand_id')::uuid)
    WITH CHECK (brand_id = (auth.jwt() ->> 'brand_id')::uuid);

-- Role-based access
CREATE POLICY "admin_full_access" ON public.examples
    FOR ALL
    USING (
        EXISTS (
            SELECT 1 FROM public.users
            WHERE id = auth.uid()
            AND role = 'Admin'
        )
    );
```

---

## Common Gotchas

### 1. Always Filter by brand_id
```sql
-- WRONG
SELECT * FROM orders WHERE status = 'Pending';

-- RIGHT  
SELECT * FROM orders WHERE brand_id = p_brand_id AND status = 'Pending';
```

### 2. Phone Normalization
Use the `normalize_phone()` function for consistent matching:
```sql
-- Normalize before insert/update
NEW.phone := normalize_phone(NEW.phone);
```

### 3. SECURITY DEFINER vs INVOKER
- `SECURITY DEFINER` - Runs as function owner, bypasses RLS
- `SECURITY INVOKER` - Runs as calling user, respects RLS

Use DEFINER for RPC functions that need cross-user data.

### 4. NULL Handling
```sql
-- Use COALESCE for defaults
COALESCE(SUM(amount), 0)

-- Use IS DISTINCT FROM for NULL-safe comparison
IF NEW.status IS DISTINCT FROM OLD.status THEN
```

---

## Migration Workflow

1. Create migration file:
   ```
   supabase/migrations/YYYYMMDDHHMMSS_description.sql
   ```

2. Apply locally:
   ```bash
   npx supabase db reset  # Full reset
   # or
   npx supabase migration up  # Apply pending
   ```

3. Generate types:
   ```bash
   npm run db:types
   ```

4. Push to remote (production):
   ```bash
   npx supabase db push
   ```
