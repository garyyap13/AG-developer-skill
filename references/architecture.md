# OHCRM Architecture Reference

Quick reference for OHCRM project structure and architectural patterns.

## Tech Stack

| Layer | Technology |
|-------|------------|
| Frontend | React 19 + TypeScript + Vite |
| Styling | TailwindCSS |
| State | React Context + TanStack Query |
| Backend | Supabase (PostgreSQL + RLS) |
| Auth | Supabase Auth |
| Deployment | Vercel |

---

## Directory Structure

```
OHCRM/
├── App.tsx                 # Router + Provider composition
├── index.tsx               # Entry point
├── types.ts                # All TypeScript interfaces
├── constants.ts            # App constants & config
│
├── views/                  # Page-level components (27 files)
│   ├── OrderManagement.tsx
│   ├── CustomerManagement.tsx
│   ├── MembershipManagement.tsx
│   └── ...
│
├── components/             # Reusable UI components (50+)
│   ├── CreateOrderModal.tsx
│   ├── CustomerDetailsModal.tsx
│   └── ui/                 # Base primitives
│
├── contexts/               # State management (15 providers)
│   ├── AuthContext.tsx
│   ├── OrderContext.tsx
│   ├── CustomerContext.tsx
│   └── ...
│
├── hooks/                  # Custom React hooks
├── lib/                    # Utilities (supabase, query client)
├── utils/                  # Helper functions
│
├── supabase/
│   └── migrations/         # SQL migrations (113+ files)
│
├── docs/                   # Documentation
│   └── technical_specs/    # Per-module specs
│
└── .agent/
    ├── workflows/          # AI agent workflows
    ├── skills/             # Agent skills
    └── openspec/           # Spec-driven changes
```

---

## Provider Hierarchy

```tsx
<ClientProvider>              // Multi-client support
  <QueryClientProvider>       // TanStack Query cache
    <BrowserRouter>
      <AuthProvider>          // Auth + user + brandId
        <CombinedDataProvider>  // Orders, Customers, Products
          <WarehouseProvider>
            <WhatsAppProvider>
              {/* App Routes */}
            </WhatsAppProvider>
          </WarehouseProvider>
        </CombinedDataProvider>
      </AuthProvider>
    </BrowserRouter>
  </QueryClientProvider>
</ClientProvider>
```

**Important**: CombinedDataProvider MUST be inside AuthProvider (needs brandId).

---

## Module Registry

| Module | Route | View | Context | Key Tables |
|--------|-------|------|---------|------------|
| Orders | `/orders` | `OrderManagement.tsx` | `OrderContext.tsx` | `orders`, `order_items` |
| Customers | `/customers` | `CustomerManagement.tsx` | `CustomerContext.tsx` | `customers` |
| VIP Membership | `/membership` | `MembershipManagement.tsx` | `MembershipContext.tsx` | `customers`, `membership_history` |
| Inventory | `/inventory` | `InventoryManagement.tsx` | `ProductContext.tsx` | `products`, `bundle_components` |
| Warehouse | `/warehouse` | `WarehouseManagement.tsx` | `WarehouseContext.tsx` | `warehouses`, `warehouse_stock` |
| Dealers | `/o2o-marketing` | `O2OMarketing.tsx` | `DealerContext.tsx` | `dealers`, `dealer_tiers` |
| Finance | `/finance` | `FinanceView.tsx` | `OrderContext.tsx` | `orders` |
| Fulfillment | `/fulfillment` | `FulfillmentView.tsx` | `OrderContext.tsx` | `orders` |
| Settings | `/settings` | `Settings.tsx` | `SettingsContext.tsx` | `sales_sources` |

---

## Role-Based Access

| Role Key | Dashboard | Access Level |
|----------|-----------|--------------|
| `Admin` | AdminDecisionDashboard | Full access |
| `Sales Manager` | SalesManagerDashboard | Team + reports |
| `Healthcare Representative` | RepDashboard | Own data only |
| `Marketing Manager` | MarketingDashboard | Marketing modules |
| `Finance Executive` | FinanceDashboard | Payment verification |
| `Fulfillment` | FulfillmentDashboard | Shipping operations |

---

## Key Conventions

### Multi-Tenant
- Almost ALL tables have `brand_id` column
- ALWAYS filter by brand_id in queries
- RLS policies enforce brand isolation

### Naming
- Database: `snake_case` (e.g., `order_items`)
- TypeScript: `camelCase` for variables, `PascalCase` for types
- Files: `PascalCase.tsx` for components, `camelCase.ts` for utils

### State Management
- Use Context for shared state
- Use TanStack Query for server state caching
- Local state for UI-only concerns

---

## Quick Navigation

| Looking for... | Check these files |
|----------------|-------------------|
| Types/Interfaces | `types.ts` |
| Constants | `constants.ts` |
| Supabase client | `lib/supabase.ts` |
| Route definitions | `App.tsx` |
| Domain hooks | `contexts/domainHooks.ts` |
