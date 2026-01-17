---
name: ohcrm-developer
description: |
  Comprehensive OHCRM development skill for building and modifying features.
  CRITICAL: Understands cross-module business logic dependencies (VIP, Inventory, etc.).
  Use when: (1) Creating new modules/views/contexts, (2) Modifying existing frontend or 
  backend code, (3) Adding database tables/functions/triggers, (4) Debugging OHCRM issues.
  Always check business-logic-map.md before changes to ensure cross-module compatibility.
---

# OHCRM Developer Skill

Build and modify OHCRM features while respecting cross-module business logic.

## Pre-Flight Checklist

Before ANY change, complete this checklist:

1. [ ] Read `references/business-logic-map.md` to understand module dependencies
2. [ ] Load the relevant OpenSpec spec (see business-logic-map.md for spec paths)
3. [ ] Identify which modules are affected by your change
4. [ ] Check for database triggers that may fire
5. [ ] Plan validation steps

> **CRITICAL**: OHCRM modules are interconnected. Changes to Orders affect VIP Membership. 
> Changes to Stock affect Bundles. Always verify cross-module impact.
>
> **OpenSpec**: Load the capability spec to understand business requirements and scenarios 
> before making changes. This ensures changes align with documented behavior.

---

## Decision Tree

```
What are you building/modifying?
│
├─→ NEW MODULE (view + context + tables)
│   └─→ Go to "New Module Workflow"
│
├─→ MODIFY EXISTING CODE
│   ├─→ Frontend only (UI changes)?
│   │   └─→ Go to "Frontend Modification"
│   │
│   ├─→ Backend only (database)?
│   │   └─→ Go to "Database Modification"
│   │
│   └─→ Full-stack change?
│       └─→ Go to "Full-Stack Modification"
│
├─→ ADD DATABASE FUNCTION/TRIGGER
│   └─→ Go to "Database Modification"
│
└─→ DEBUGGING
    └─→ Go to "Debugging Workflow"
```

---

## New Module Workflow

Creating a complete new module (View + Context + Database).

### Step 1: Plan & Check Dependencies

1. Define the module scope and data requirements
2. Check `references/business-logic-map.md` for related modules
3. Identify if this module affects or is affected by:
   - Orders
   - VIP Membership
   - Inventory/Stock
   - Customer data

### Step 2: Database Layer

1. Create migration file: `supabase/migrations/YYYYMMDDHHMMSS_module_name.sql`
2. Define tables (see `references/database-patterns.md`)
3. Add RLS policies for brand isolation
4. Add RPC functions if needed
5. **If cross-module**: Add triggers to maintain consistency

### Step 3: Types

1. Add interfaces to `types.ts`
2. Export new types

### Step 4: Context

1. Create `contexts/ModuleContext.tsx`
2. Follow pattern in `references/frontend-patterns.md`
3. Wire up Supabase queries
4. Export hook: `useModule()`

### Step 5: View

1. Create `views/ModuleManagement.tsx`
2. Follow view pattern in `references/frontend-patterns.md`
3. Use existing component patterns for modals

### Step 6: Routing

1. Add route in `App.tsx`
2. Add provider to hierarchy if needed (in `CombinedDataProvider.tsx` or `App.tsx`)

### Step 7: Verify

1. Run `/verify-system` workflow
2. Test the module manually
3. Verify no cross-module issues

---

## Frontend Modification

Modifying existing React components or contexts.

### Step 1: Impact Analysis

1. What component/context are you changing?
2. Check if it touches data that triggers database automation
3. Review `references/business-logic-map.md` for dependencies

Example checklist for OrderContext changes:
- [ ] Does this change order status? → VIP trigger will fire
- [ ] Does this change order total? → VIP trigger will fire
- [ ] Does this add new order fields? → May need migration

### Step 2: Implement

1. Follow patterns in `references/frontend-patterns.md`
2. Maintain existing code style
3. Use TypeScript types from `types.ts`

### Step 3: Validate

1. Run `npm run lint`
2. Run `npm run type-check`
3. Test affected workflows

---

## Database Modification

Adding tables, columns, functions, or triggers.

### Step 1: Impact Analysis (CRITICAL)

1. Check `references/business-logic-map.md`
2. Identify ALL triggers that may be affected
3. Map out the data flow

For trigger changes, trace the chain:
```
Your Change → Trigger A → Function B → Side Effect C
```

### Step 2: Create Migration

1. File: `supabase/migrations/YYYYMMDDHHMMSS_description.sql`
2. Follow patterns in `references/database-patterns.md`
3. Use `CREATE OR REPLACE` for idempotency

### Step 3: Test Locally

```bash
npx supabase db reset  # Apply migrations
npm run db:types       # Regenerate types
```

### Step 4: Verify Cross-Module

1. Test affected triggers manually
2. Verify dependent modules still work
3. Check VIP recalculation if touching orders
4. Check bundle stock if touching inventory

---

## Full-Stack Modification

Changes spanning frontend and backend.

### Step 1: Plan End-to-End

1. Map the full data flow
2. Identify all affected files
3. Check `references/business-logic-map.md`
4. Consider using `/openspec-proposal` for significant changes

### Step 2: Backend First

1. Create migrations
2. Add RPC functions
3. Test database layer independently

### Step 3: Frontend Second

1. Update types
2. Modify context
3. Update view/components

### Step 4: Integration Test

1. Test the complete flow
2. Verify cross-module effects
3. Run `/verify-system`

---

## Debugging Workflow

Finding and fixing issues in OHCRM.

### Step 1: Identify Scope

Is this a:
- Frontend issue (UI not rendering, state not updating)?
- Backend issue (data not saving, RPC error)?
- Cross-module issue (VIP not calculating, stock wrong)?

### Step 2: For Cross-Module Issues

1. Check `references/business-logic-map.md` for the trigger chain
2. Verify triggers are firing:
   ```sql
   SELECT * FROM information_schema.triggers 
   WHERE trigger_schema = 'public';
   ```
3. Check if conditions trigger the automation

### Step 3: Common Issues

| Symptom | Likely Cause | Check |
|---------|--------------|-------|
| VIP not updating | Order status not in trigger list | Status must be Paid/Processing/Shipped/Delivered |
| Bundle stock wrong | Component not in warehouse_stock | Check warehouse_stock table |
| Customer not linked | Phone not normalized | Check normalize_phone() |
| Data not showing | brand_id filter missing | Check RLS and queries |

---

## Quick Reference

### File Locations

| Need to find... | Look in... |
|-----------------|------------|
| TypeScript types | `types.ts` |
| Route definitions | `App.tsx` |
| Domain hooks | `contexts/domainHooks.ts` |
| Supabase client | `lib/supabase.ts` |
| Constants | `constants.ts` |

### Key Thresholds

| Business Rule | Value |
|---------------|-------|
| VIP Qualification | Single order >= RM 668 |
| VIP Renewal | Annual spend >= RM 2,000 |
| Membership Duration | 1 year |

### Trigger-Eligible Statuses (VIP)

Orders with these statuses count toward VIP:
- Paid
- Processing
- Shipped
- Delivered
- Completed

---

## Reference Documents

Load these when you need detailed information:

| Document | When to Load |
|----------|--------------|
| `references/business-logic-map.md` | **Always** - before any change |
| `references/architecture.md` | Understanding project structure |
| `references/frontend-patterns.md` | Creating/modifying React code |
| `references/database-patterns.md` | Creating/modifying SQL |
| `references/types-reference.md` | Working with TypeScript types |
| `references/workflow-integration.md` | Using existing workflows |

---

## Gotchas & Common Mistakes

1. **Forgetting brand_id** - Almost all queries need brand isolation
2. **Modifying orders without testing VIP** - Always verify VIP recalculates
3. **Direct product.stock updates** - Stock syncs from warehouse_stock
4. **Ignoring trigger chains** - Changes cascade through triggers
5. **Phone format inconsistency** - Always normalize phones
6. **Missing RLS policies** - New tables need brand isolation
7. **Not updating types.ts** - Frontend won't see new fields
