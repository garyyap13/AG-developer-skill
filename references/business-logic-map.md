# Business Logic Dependency Map

This is the **most critical** reference for OHCRM development. Before modifying ANY module, consult this document to understand cross-module impacts.

## Cross-Module Dependency Matrix

| When You Touch... | Also Affects... | Mechanism |
|-------------------|-----------------|-----------|
| **Orders** (status → Paid/Shipped/Delivered) | VIP Membership | `trg_vip_on_order_change` trigger |
| **Orders** (total amount changed) | Customer LTV | Trigger recalculates LTV |

| **Orders** (status → Cancelled/Returned) | VIP Membership | Recalculated (order excluded) |

| **Orders** (payment_method = COD) | COD Flag | `trg_set_cod_flag` auto-sets is_cod |
| **Products** (stock change) | Bundle Stock | `sync_bundle_stock_on_component_change` |
| **Warehouse Stock** | Product Stock | `sync_product_stock_from_warehouses` |
| **Customer** (phone field) | Order Matching | Import deduplication uses phone |
| **Dealers** (sales totals) | Dealer Tier | Tier calculation based on period sales |
| **COD Orders** (reconciled) | Order Status | Updates cod_reconciled_at |

---

## Order Lifecycle Effects

```
ORDER STATUS FLOW
━━━━━━━━━━━━━━━━━

[Pending Payment] ─→ Customer pays
        │
        ▼
    [Paid] ─────────→ Finance verifies (online channels)
        │              Marketplace: auto-verified
        ▼
  [Processing] ────→ Fulfillment picks & packs
        │
        ▼
   [Shipped] ──────→ Tracking added
        │
        ├──→ [Delivered] ──→ VIP Recalculated ──→ [Completed]
        │
        └──→ [Returned] ──→ VIP Recalculated (excluded)
                           Stock Restored
```

### VIP Trigger Points

The `trg_vip_on_order_change` trigger fires when:
- Order status changes TO: `Paid`, `Processing`, `Shipped`, `Delivered`, `Completed`
- Order status changes TO: `Cancelled`, `Returned` (recalc to exclude)
- Order total changes by more than RM 10

The trigger calls `recalculate_customer_vip(customer_id)` which:
1. Fetches all qualifying orders for the customer
2. Checks for single order >= RM 668 (VIP qualification)
3. Checks annual spend >= RM 2,000 (renewal)
4. Updates `customers.membership_tier`, `membership_expiry_date`
5. Logs event to `membership_history`

---

## VIP Membership Rules

| Rule | Threshold | Effect |
|------|-----------|--------|
| **Qualification** | Single order >= RM 668 | Become Member, 1-year validity |
| **Renewal** | Annual spend >= RM 2,000 | Extend membership 1 year |
| **Renewal Timing** | At expiry only | Early spending qualifies but doesn't extend early |
| **Lapse** | Spend < RM 2,000 at expiry | Downgrade to Non-Member |

### Important: Early Spending Rule
- Spending >= RM 2,000 DURING membership qualifies for renewal
- But renewal only APPLIES when current period expires
- Example: Member expires Dec 2024, spends RM 3000 in June → Still expires Dec 2024, THEN renews to Dec 2025

---

## Inventory Flow

```
STOCK SYNCHRONIZATION
━━━━━━━━━━━━━━━━━━━━

Warehouse Stock Updated
        │
        ▼
[TRIGGER: sync_product_stock_from_warehouses]
        │
        ▼
SUM(all warehouse stock) → products.stock
        │
        ▼
[TRIGGER: sync_bundle_stock_on_component_change]
        │
        ▼
Bundle stock = MIN(component_stock / required_qty)
```

### Bundle Stock Formula
For a bundle with components:
- Component A: 100 units, requires 2 per bundle
- Component B: 60 units, requires 1 per bundle

Bundle stock = MIN(100/2, 60/1) = MIN(50, 60) = **50 bundles**

---

## Customer Matching Logic (Imports)

When importing orders, customers are matched in this priority:
1. **Phone** (exact match after normalization)
2. **Username** (marketplace username)
3. **Name** (exact match)

If no match found → Auto-create new customer

### Phone Normalization
Malaysian phones normalized to: `601XXXXXXXXX`
- Strips spaces, dashes, parentheses
- Converts `+60` → `60`
- Converts `0` prefix → `60`

---

## COD Flow (Cash on Delivery)

### COD Order Creation

When an order is created with `payment_method = 'COD'` or `'Cash'`:

```
ORDER INSERT/UPDATE (payment_method = COD)
        │
        ▼
[TRIGGER: trg_set_cod_flag]
        │
        ├─→ Sets is_cod = TRUE
        └─→ Sets cod_amount = order.total
```

**Note**: Marketplace orders (Shopee, Lazada, TikTok) do NOT get is_cod flag even if COD.

### COD Reconciliation Flow

```
COURIER REMITTANCE FILE IMPORTED
        │
        ▼
import_cod_reconciliation() RPC
        │
        ├─→ Creates cod_reconciliations record
        ├─→ Matches tracking numbers to orders
        └─→ Creates cod_reconciliation_items

FINANCE VERIFIES RECONCILIATION
        │
        ▼
verify_cod_reconciliation() RPC
        │
        ├─→ Sets status = 'verified'
        └─→ Updates orders: cod_reconciled_at = NOW()
```

### COD-Related Tables

| Table | Purpose |
|-------|---------|
| `orders.is_cod` | Boolean flag for COD orders |
| `orders.cod_amount` | Amount to collect |
| `orders.cod_collected_at` | When courier collected |
| `orders.cod_reconciled_at` | When reconciled with remittance |
| `cod_reconciliations` | Batch reconciliation records |
| `cod_reconciliation_items` | Individual order matches |

---

## Impact Checklist Templates

### Before Modifying Orders Module
```
- [ ] Does this change affect order status? → Check VIP trigger
- [ ] Does this change affect order total? → Check VIP trigger
- [ ] Does this change affect items/SKUs? → Check stock deduction
- [ ] Does this change customer linkage? → Check customer matching
```

### Before Modifying Customer Module
```
- [ ] Does this change affect phone field? → Check import matching
- [ ] Does this change affect LTV calculation? → Check order triggers
- [ ] Does this change membership fields? → Check VIP recalculation
```

### Before Modifying Products/Inventory Module
```
- [ ] Does this change affect stock? → Check bundle sync trigger
- [ ] Does this change bundles? → Check bundle stock calculation
- [ ] Does this change SKU? → Check order items foreign key
```

### Before Modifying Finance Module
```
- [ ] Does this change payment verification? → Check status flow
- [ ] Does this change dealer payments? → Check dealer tier logic
```

### Before Modifying Fulfillment Module
```
- [ ] Does this change shipping status? → Check VIP trigger
- [ ] Does this change returns? → Check stock restoration logic
```

---

## Database Triggers Reference

| Trigger | Table | Events | Function Called |
|---------|-------|--------|-----------------|
| `trg_vip_on_order_change` | orders | INSERT, UPDATE, DELETE | `trigger_vip_on_order_change()` |
| `trg_set_cod_flag` | orders | INSERT, UPDATE (payment_method) | `set_cod_flag()` |
| `trg_sync_product_stock` | warehouse_stock | INSERT, UPDATE, DELETE | `sync_product_stock_from_warehouses()` |
| `trg_sync_bundle_stock` | bundle_components | INSERT, UPDATE, DELETE | `sync_bundle_stock_on_component_change()` |
| `trg_update_customer_ltv` | orders | INSERT, UPDATE | Updates customer.ltv |

---

## Common Cross-Module Bugs

### Bug: VIP not updating after order completion
**Cause**: Order status didn't change to a trigger-eligible status
**Fix**: Ensure status transitions through Paid → Processing → Shipped → Delivered

### Bug: Bundle stock incorrect
**Cause**: Component stock was updated directly without trigger
**Fix**: Always update through `warehouse_stock` table

### Bug: Customer not linked after import
**Cause**: Phone not normalized correctly
**Fix**: Use `normalize_phone()` function

### Bug: Order total changed but VIP not recalculated
**Cause**: Total change was less than RM 10 threshold
**Fix**: Trigger only fires for changes > RM 10 to avoid noise

---

## Files by Business Domain

### VIP Membership
- `supabase/migrations/*vip*.sql` - VIP calculation function
- `contexts/MembershipContext.tsx` - Frontend state
- `views/MembershipManagement.tsx` - Settings UI
- `components/CustomerDetailsModal.tsx` - Manual recalc button

### Order Processing
- `contexts/OrderContext.tsx` - All order operations
- `views/OrderManagement.tsx` - Order list UI
- `views/FinanceView.tsx` - Payment verification
- `views/FulfillmentView.tsx` - Shipping operations

### Inventory
- `contexts/ProductContext.tsx` - Product CRUD
- `contexts/WarehouseContext.tsx` - Stock management
- `views/InventoryManagement.tsx` - Product list
- `views/WarehouseManagement.tsx` - Warehouse operations

---

## OpenSpec Capability References

For detailed requirements and business rules, load the relevant OpenSpec spec:

| Capability | OpenSpec Spec | When to Load |
|------------|---------------|--------------|
| **VIP Membership** | `openspec/specs/membership/spec.md` | Working on membership logic, tier calculation |
| **Order Processing** | `openspec/specs/order-processing/spec.md` | Modifying order workflow, status transitions |
| **Inventory** | `openspec/specs/inventory/spec.md` | Stock management, bundle logic |
| **Customer Management** | `openspec/specs/customer-management/spec.md` | Customer data, linking, imports |
| **O2O Marketing** | `openspec/specs/o2o-marketing/spec.md` | Dealer management, tier logic |
| **User Management** | `openspec/specs/user-management/spec.md` | Roles, permissions, access control |
| **Access Control** | `openspec/specs/access-control/spec.md` | RLS policies, role-based access |

### Master Spec
- **VIP Membership Flow**: `openspec/specs/vip-membership-flow.md` - Complete VIP calculation documentation
- **Database Objects**: `openspec/specs/database-objects.md` - All tables, functions, triggers

### How to Use OpenSpec Specs

When modifying a capability:
1. Load the relevant spec from the table above
2. Review the requirements and scenarios
3. Ensure your changes align with documented behavior
4. Update the spec if you're adding new requirements (via OpenSpec proposal)
