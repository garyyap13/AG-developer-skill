# TypeScript Types Reference

All OHCRM types are defined in `types.ts`. This reference covers the core types.

## Core Entity Types

### Customer
```typescript
interface Customer {
  id: string;
  brandId: string;
  name: string;
  phone: string;
  email: string;
  tags: string[];                    // e.g., #Hypertension, #Insomnia
  lastOrderDate: string;
  ltv: number;                       // Lifetime Value
  status: 'Active' | 'Churn Risk' | 'New' | 'Expired';
  address?: string;
  city?: string;
  state?: string;
  postcode?: string;
  nextFollowUp?: string;
  assignedTo?: string;               // Sales Rep ID
  // Membership
  membershipTier: 'Member' | 'Non-Member';
  membershipExpiryDate?: string;
  currentYearSpend: number;          // For RM2000 renewal tracking
  joinDate?: string;
}
```

### Order
```typescript
interface Order {
  id: string;
  brandId: string;
  customerId: string;
  customerName: string;
  customerPhone?: string;
  csRepName?: string;
  marketplaceUsername?: string;
  date: string;
  total: number;
  status: 'Pending Payment' | 'Paid' | 'Processing' | 'Shipped' | 
          'Delivered' | 'Returned' | 'Cancellation Requested' | 'Cancelled';
  items: { sku: string; quantity: number; price?: number }[];
  paymentMethod: 'Credit Card' | 'Bank Transfer' | 'COD' | 
                 'ShopeePay' | 'Lazada Wallet' | 'Cash';
  paymentProofUrl?: string;
  trackingNumber?: string;
  // Platform
  platform: 'Website' | 'Shopee' | 'Lazada' | 'TikTok' | 
            'Facebook' | 'WhatsApp' | 'O2O' | string;
  storeCode?: string;                // O2O dealer
  platformFee?: number;
  netPayout?: number;
  isReconciled: boolean;
  shippingFee?: number;
  packagePrice?: number;             // total - shipping (for VIP)
  // Payment verification
  paymentVerifiedAt?: string;
  paymentVerifiedBy?: string;
  paymentRejectionReason?: string;
  // Shipping
  courier?: string;
  shippedAt?: string;
  shippedBy?: string;
  deliveredAt?: string;
  // Returns
  returnedAt?: string;
  returnReason?: string;
  returnNotes?: string;
  // Cancellation
  cancelReason?: string;
  cancelNotes?: string;
  cancelApprovedAt?: string;
}
```

### Product
```typescript
interface Product {
  sku: string;                       // Primary key
  brandId: string;
  name: string;
  category: 'Single' | 'Bundle' | 'Gift';
  stock: number;                     // Aggregate from warehouses
  price: number;
  status?: 'Active' | 'Inactive';
  bundleItems?: BundleComponent[];   // If category is 'Bundle'
}

interface BundleComponent {
  parentSku: string;
  childSku: string;
  quantity: number;                  // How many of child per parent
}
```

### Dealer
```typescript
interface Dealer {
  id: string;
  brandId?: string;
  name: string;
  state: string;
  city: string;
  address: string;
  sales: number;
  periodSales?: number;
  stock: number;
  latitude?: number;
  longitude?: number;
  status: 'Active' | 'Inactive';
  tier?: string;
  picName?: string;                  // Person In Charge
  picPhone?: string;
  storeCode?: string;
}
```

---

## Enums

### UserRole
```typescript
enum UserRole {
  REP = 'Healthcare Representative',
  SALES_MGR = 'Sales Manager',
  MKT_ONLINE = 'Marketing Exec (Online)',
  MKT_OFFLINE = 'Marketing Exec (O2O)',
  MKT_MGR = 'Marketing Manager',
  FINANCE = 'Finance Executive',
  FULFILLMENT = 'Fulfillment',
  ADMIN = 'Admin',
}
```

### Order Statuses
```typescript
type OrderStatus = 
  | 'Pending Payment' 
  | 'Paid' 
  | 'Processing' 
  | 'Shipped' 
  | 'Delivered' 
  | 'Returned' 
  | 'Cancellation Requested' 
  | 'Cancelled';
```

### Membership Tiers
```typescript
type MembershipTier = 'Member' | 'Non-Member';
```

### Membership Event Types
```typescript
type MembershipEventType = 
  | 'new_member' 
  | 'renewal' 
  | 'lapsed' 
  | 'rejoined' 
  | 'manual_extend' 
  | 'manual_downgrade' 
  | 'upgrade' 
  | 'auto_renew' 
  | 'rejoin' 
  | 'downgrade';
```

---

## History & Logging Types

### MembershipHistory
```typescript
interface MembershipHistory {
  id: string;
  customerId: string;
  brandId: string;
  eventType: MembershipEventType;
  previousTier: string;
  newTier: string;
  previousExpiry?: string;
  newExpiry?: string;
  spendAtEvent: number;
  eventDate: string;
  triggerOrderId?: string;
  triggeredBy: 'system' | 'manual' | 'order';
  notes?: string;
}
```

### OrderStatusHistory
```typescript
interface OrderStatusHistory {
  id: string;
  orderId: string;
  oldStatus: string | null;
  newStatus: string;
  changedBy: string | null;
  changedByName: string | null;
  notes: string | null;
  createdAt: string;
}
```

### ContactLog
```typescript
interface ContactLog {
  id: string;
  customerId: string;
  brandId: string;
  contactedBy: string;
  contactedByName: string;
  contactMethod: 'WhatsApp' | 'Phone' | 'Messenger' | 'Email' | 'In Person';
  outcome: 'No Answer' | 'Interested' | 'Not Interested' | 
           'Callback Requested' | 'Order Placed' | 'Other';
  notes?: string;
  nextActionDate?: string;
  contactedAt: string;
}
```

---

## Stats Types

### MembershipStats
```typescript
interface MembershipStats {
  totalVipMembers: number;
  newVipsThisMonth: number;
  churnedThisMonth: number;
  churnRate: number;
  avgVipClv: number;
  totalVipLtv: number;
  expiring30d: number;
  expiring60d: number;
  expiring90d: number;
  closeToRenewal: number;
  avgCurrentSpend: number;
}
```

---

## Adding New Types

1. Define in `types.ts`
2. Export at file level
3. Import where needed:
   ```typescript
   import { Customer, Order, Product } from '../types';
   ```

### Type Extensions
For optional fields or variations:
```typescript
// Partial for updates
type CustomerUpdate = Partial<Customer>;

// Pick for specific fields
type CustomerSummary = Pick<Customer, 'id' | 'name' | 'phone'>;

// Extend for new fields
interface ExtendedCustomer extends Customer {
  newField: string;
}
```
