# Frontend Patterns

Patterns and conventions for OHCRM React frontend development.

## View Pattern

Views are page-level components in `views/`. They compose contexts, components, and local state.

```tsx
// views/ExampleManagement.tsx
import React, { useState, useEffect } from 'react';
import { useExample } from '../contexts/domainHooks';
import { Example } from '../types';
import { ExampleModal } from '../components/ExampleModal';
import { Plus, Search, Filter } from 'lucide-react';

export function ExampleManagement() {
  // 1. Context hooks
  const { examples, isLoading, addExample, refetch } = useExample();
  
  // 2. Local UI state
  const [searchQuery, setSearchQuery] = useState('');
  const [showModal, setShowModal] = useState(false);
  const [selectedItem, setSelectedItem] = useState<Example | null>(null);
  
  // 3. Derived/filtered data
  const filteredExamples = examples.filter(e => 
    e.name.toLowerCase().includes(searchQuery.toLowerCase())
  );
  
  // 4. Event handlers
  const handleCreate = async (data: Example) => {
    await addExample(data);
    setShowModal(false);
    refetch();
  };
  
  // 5. Render
  return (
    <div className="p-6">
      {/* Header with actions */}
      <div className="flex justify-between items-center mb-6">
        <h1 className="text-2xl font-bold">Examples</h1>
        <button 
          onClick={() => setShowModal(true)}
          className="flex items-center gap-2 bg-blue-600 text-white px-4 py-2 rounded"
        >
          <Plus size={16} /> Add New
        </button>
      </div>
      
      {/* Search/Filter bar */}
      <div className="mb-4">
        <input 
          type="text"
          placeholder="Search..."
          value={searchQuery}
          onChange={(e) => setSearchQuery(e.target.value)}
          className="border rounded px-3 py-2 w-full max-w-md"
        />
      </div>
      
      {/* Data table/grid */}
      {isLoading ? (
        <div>Loading...</div>
      ) : (
        <table className="w-full">
          {/* ... */}
        </table>
      )}
      
      {/* Modals */}
      {showModal && (
        <ExampleModal 
          onClose={() => setShowModal(false)}
          onSave={handleCreate}
        />
      )}
    </div>
  );
}
```

---

## Context Pattern

Contexts manage domain state and provide actions. Located in `contexts/`.

```tsx
// contexts/ExampleContext.tsx
import React, { createContext, useContext, useState, useEffect, ReactNode } from 'react';
import { useAuth } from './AuthContext';
import { Example } from '../types';
import { supabase } from '../lib/supabase';

// 1. Define interface
interface ExampleContextType {
  examples: Example[];
  isLoading: boolean;
  addExample: (example: Example) => Promise<void>;
  updateExample: (id: string, updates: Partial<Example>) => Promise<void>;
  refetch: () => Promise<void>;
}

// 2. Create context
const ExampleContext = createContext<ExampleContextType | undefined>(undefined);

// 3. Provider component
export function ExampleProvider({ children }: { children: ReactNode }) {
  const { user, brandId } = useAuth();
  const [examples, setExamples] = useState<Example[]>([]);
  const [isLoading, setIsLoading] = useState(true);
  
  // Fetch data
  const fetchData = async () => {
    if (!brandId) return;
    setIsLoading(true);
    
    const { data, error } = await supabase
      .from('examples')
      .select('*')
      .eq('brand_id', brandId)
      .order('created_at', { ascending: false });
    
    if (!error && data) {
      setExamples(data);
    }
    setIsLoading(false);
  };
  
  useEffect(() => {
    fetchData();
  }, [brandId]);
  
  // Actions
  const addExample = async (example: Example) => {
    const { error } = await supabase
      .from('examples')
      .insert({ ...example, brand_id: brandId });
    
    if (error) throw error;
    await fetchData(); // Refetch to sync
  };
  
  const updateExample = async (id: string, updates: Partial<Example>) => {
    const { error } = await supabase
      .from('examples')
      .update(updates)
      .eq('id', id);
    
    if (error) throw error;
    await fetchData();
  };
  
  return (
    <ExampleContext.Provider value={{
      examples,
      isLoading,
      addExample,
      updateExample,
      refetch: fetchData
    }}>
      {children}
    </ExampleContext.Provider>
  );
}

// 4. Custom hook
export function useExample() {
  const context = useContext(ExampleContext);
  if (!context) {
    throw new Error('useExample must be used within ExampleProvider');
  }
  return context;
}
```

---

## Component Pattern

Reusable components in `components/`. Modals are the most common.

```tsx
// components/ExampleModal.tsx
import React, { useState } from 'react';
import { X } from 'lucide-react';
import { Example } from '../types';

interface ExampleModalProps {
  example?: Example;           // Optional for edit mode
  onClose: () => void;
  onSave: (data: Example) => Promise<void>;
}

export function ExampleModal({ example, onClose, onSave }: ExampleModalProps) {
  const [formData, setFormData] = useState({
    name: example?.name || '',
    value: example?.value || 0,
  });
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [error, setError] = useState<string | null>(null);
  
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setIsSubmitting(true);
    setError(null);
    
    try {
      await onSave(formData as Example);
    } catch (err) {
      setError(err instanceof Error ? err.message : 'Failed to save');
    } finally {
      setIsSubmitting(false);
    }
  };
  
  return (
    <div className="fixed inset-0 bg-black/50 flex items-center justify-center z-50">
      <div className="bg-white rounded-lg shadow-xl w-full max-w-md">
        {/* Header */}
        <div className="flex justify-between items-center p-4 border-b">
          <h2 className="text-lg font-semibold">
            {example ? 'Edit Example' : 'New Example'}
          </h2>
          <button onClick={onClose}>
            <X size={20} />
          </button>
        </div>
        
        {/* Form */}
        <form onSubmit={handleSubmit} className="p-4 space-y-4">
          {error && (
            <div className="bg-red-100 text-red-700 p-3 rounded">
              {error}
            </div>
          )}
          
          <div>
            <label className="block text-sm font-medium mb-1">Name</label>
            <input
              type="text"
              value={formData.name}
              onChange={(e) => setFormData(prev => ({ ...prev, name: e.target.value }))}
              className="w-full border rounded px-3 py-2"
              required
            />
          </div>
          
          {/* Footer */}
          <div className="flex justify-end gap-2 pt-4">
            <button 
              type="button" 
              onClick={onClose}
              className="px-4 py-2 border rounded"
            >
              Cancel
            </button>
            <button 
              type="submit" 
              disabled={isSubmitting}
              className="px-4 py-2 bg-blue-600 text-white rounded disabled:opacity-50"
            >
              {isSubmitting ? 'Saving...' : 'Save'}
            </button>
          </div>
        </form>
      </div>
    </div>
  );
}
```

---

## UI Conventions

### Icons
Use **Lucide React** for all icons:
```tsx
import { Plus, Search, Filter, X, Edit, Trash2, ChevronRight } from 'lucide-react';
<Plus size={16} className="text-gray-500" />
```

### Colors (Tailwind)
- Primary: `blue-600`
- Success: `green-600`
- Warning: `yellow-600`
- Danger: `red-600`
- Neutral: `gray-*`

### Status Badges
```tsx
const statusColors = {
  'Active': 'bg-green-100 text-green-800',
  'Pending': 'bg-yellow-100 text-yellow-800',
  'Inactive': 'bg-gray-100 text-gray-800',
  'Error': 'bg-red-100 text-red-800',
};
```

### Mobile-First
- Use responsive classes: `md:`, `lg:`
- Default to mobile layout
- Tables become cards on mobile

---

## Domain Hooks

Use `contexts/domainHooks.ts` for cleaner imports:

```tsx
// Instead of:
import { useCustomers } from '../contexts/CustomerContext';
import { useOrders } from '../contexts/OrderContext';

// Use:
import { useCustomers, useOrders } from '../contexts/domainHooks';
```
