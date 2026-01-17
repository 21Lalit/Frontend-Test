# Frontend Intern Technical Assessment

**Time Limit:** 90 minutes
**Tools Allowed:** AI assistants, documentation, any resources
**Submission:** JSON file following the schema at the end

---

## Section A: Debug Challenge (25 points)

The following React component has 4 bugs causing incorrect behavior. Identify each bug, explain WHY it's a bug (not just what to fix), and provide the corrected code.

```tsx
// ProductCard.tsx - A card that shows product info with quantity selector
import { useState, useEffect } from 'react';

interface Product {
  id: string;
  name: string;
  price: number;
  stock: number;
}

interface Props {
  productId: string;
  onAddToCart: (productId: string, quantity: number) => void;
}

export function ProductCard({ productId, onAddToCart }: Props) {
  const [product, setProduct] = useState<Product>(null);
  const [quantity, setQuantity] = useState(1);
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    fetchProduct();
  }, []);

  async function fetchProduct() {
    setLoading(true);
    const res = await fetch(`/api/products/${productId}`);
    const data = await res.json();
    setProduct(data);
    setLoading(false);
  }

  function handleQuantityChange(delta: number) {
    const newQty = quantity + delta;
    if (newQty >= 1 && newQty <= product.stock) {
      setQuantity(newQty);
    }
  }

  function handleAddToCart() {
    onAddToCart(productId, quantity);
    setQuantity(1);
  }

  if (loading) return <div className="skeleton h-48 w-full" />;

  return (
    <div className="card bg-base-100 shadow-xl">
      <div className="card-body">
        <h2 className="card-title">{product.name}</h2>
        <p className="text-xl font-bold">${product.price}</p>
        <p className="text-sm text-gray-500">In stock: {product.stock}</p>

        <div className="flex items-center gap-2">
          <button
            className="btn btn-circle btn-sm"
            onClick={() => handleQuantityChange(-1)}
          >-</button>
          <span className="w-8 text-center">{quantity}</span>
          <button
            className="btn btn-circle btn-sm"
            onClick={() => handleQuantityChange(1)}
          >+</button>
        </div>

        <button
          className="btn btn-primary mt-4"
          onClick={handleAddToCart}
          disabled={product.stock == 0}
        >
          Add to Cart
        </button>
      </div>
    </div>
  );
}
```

**For each bug, provide:**
1. Bug location (line reference or code snippet)
2. What goes wrong at runtime
3. Why this is a problem (the underlying concept)
4. The fix

---

## Section B: Performance Analysis (25 points)

A colleague wrote this dashboard component. Users complain it's slow and laggy. The component re-renders frequently and causes jank.

```tsx
// Dashboard.tsx
import { useState, useEffect } from 'react';

interface Order {
  id: string;
  customer: string;
  items: { name: string; price: number; qty: number }[];
  status: 'pending' | 'shipped' | 'delivered';
  createdAt: string;
}

export function Dashboard() {
  const [orders, setOrders] = useState<Order[]>([]);
  const [filter, setFilter] = useState<string>('all');
  const [searchTerm, setSearchTerm] = useState('');

  useEffect(() => {
    const interval = setInterval(async () => {
      const res = await fetch('/api/orders');
      const data = await res.json();
      setOrders(data);
    }, 1000);
    return () => clearInterval(interval);
  }, []);

  const filteredOrders = orders.filter(order => {
    if (filter !== 'all' && order.status !== filter) return false;
    if (searchTerm && !order.customer.toLowerCase().includes(searchTerm.toLowerCase())) return false;
    return true;
  });

  const stats = {
    total: orders.length,
    pending: orders.filter(o => o.status === 'pending').length,
    shipped: orders.filter(o => o.status === 'shipped').length,
    delivered: orders.filter(o => o.status === 'delivered').length,
    revenue: orders.reduce((sum, o) => sum + o.items.reduce((s, i) => s + i.price * i.qty, 0), 0),
  };

  function formatCurrency(amount: number) {
    return new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD' }).format(amount);
  }

  return (
    <div className="p-6">
      <div className="stats shadow mb-6">
        <div className="stat">
          <div className="stat-title">Total Orders</div>
          <div className="stat-value">{stats.total}</div>
        </div>
        <div className="stat">
          <div className="stat-title">Revenue</div>
          <div className="stat-value">{formatCurrency(stats.revenue)}</div>
        </div>
        <div className="stat">
          <div className="stat-title">Pending</div>
          <div className="stat-value text-warning">{stats.pending}</div>
        </div>
      </div>

      <div className="flex gap-4 mb-4">
        <input
          type="text"
          placeholder="Search customer..."
          className="input input-bordered"
          value={searchTerm}
          onChange={(e) => setSearchTerm(e.target.value)}
        />
        <select
          className="select select-bordered"
          value={filter}
          onChange={(e) => setFilter(e.target.value)}
        >
          <option value="all">All</option>
          <option value="pending">Pending</option>
          <option value="shipped">Shipped</option>
          <option value="delivered">Delivered</option>
        </select>
      </div>

      <div className="overflow-x-auto">
        <table className="table">
          <thead>
            <tr>
              <th>Order ID</th>
              <th>Customer</th>
              <th>Items</th>
              <th>Total</th>
              <th>Status</th>
              <th>Date</th>
            </tr>
          </thead>
          <tbody>
            {filteredOrders.map(order => (
              <tr key={order.id}>
                <td>{order.id}</td>
                <td>{order.customer}</td>
                <td>{order.items.length} items</td>
                <td>{formatCurrency(order.items.reduce((s, i) => s + i.price * i.qty, 0))}</td>
                <td><span className={`badge badge-${order.status === 'pending' ? 'warning' : order.status === 'shipped' ? 'info' : 'success'}`}>{order.status}</span></td>
                <td>{new Date(order.createdAt).toLocaleDateString()}</td>
              </tr>
            ))}
          </tbody>
        </table>
      </div>
    </div>
  );
}
```

**Identify at least 4 performance issues and explain:**
1. What the issue is
2. Why it causes performance problems
3. How you would fix it (describe approach, not full code)
4. What React concepts/APIs would you use

---

## Section C: Component Design (25 points)

Design a `<FileUploader />` component with these requirements:

- Drag & drop zone + click to select files
- Multiple file upload support
- Show upload progress for each file
- Max 5 files, max 10MB each
- Only allow images (jpg, png, webp) and PDFs
- Show preview thumbnails for images
- Allow removing files before final submission
- Accessible (keyboard navigation, screen reader support)
- Mobile-friendly

**Provide:**
1. Component interface (props, types)
2. Internal state structure
3. Key accessibility considerations (ARIA, keyboard)
4. Pseudo-code or skeleton of the component logic
5. How you'd handle errors (file too large, wrong type, upload failure)

*Note: We're evaluating your design thinking, not syntax perfection.*

---

## Section D: Code Review (25 points)

Review this PR diff. The developer says "Added real-time notifications feature."

Find issues related to: correctness, performance, security, accessibility, maintainability.

```tsx
// NotificationBell.tsx
import { useEffect, useState } from 'react';

export function NotificationBell({ userId }) {
  const [notifications, setNotifications] = useState([]);
  const [unreadCount, setUnreadCount] = useState(0);
  const [isOpen, setIsOpen] = useState(false);

  useEffect(() => {
    const ws = new WebSocket(`wss://api.example.com/notifications?user=${userId}`);

    ws.onmessage = (event) => {
      const notification = JSON.parse(event.data);
      setNotifications(prev => [notification, ...prev]);
      setUnreadCount(prev => prev + 1);

      // Play sound for new notifications
      new Audio('/notification.mp3').play();
    };

    ws.onerror = () => console.log('WebSocket error');

    // Load existing notifications
    fetch(`/api/notifications?userId=${userId}`)
      .then(res => res.json())
      .then(data => {
        setNotifications(data.notifications);
        setUnreadCount(data.notifications.filter(n => !n.read).length);
      });
  }, []);

  function markAsRead(notificationId) {
    fetch(`/api/notifications/${notificationId}/read`, { method: 'POST' });
    setNotifications(prev =>
      prev.map(n => n.id === notificationId ? { ...n, read: true } : n)
    );
    setUnreadCount(prev => prev - 1);
  }

  function markAllRead() {
    notifications.forEach(n => {
      if (!n.read) markAsRead(n.id);
    });
  }

  return (
    <div className="relative">
      <button onClick={() => setIsOpen(!isOpen)}>
        <svg>...</svg>
        {unreadCount > 0 && (
          <span className="badge badge-error badge-sm absolute -top-1 -right-1">
            {unreadCount}
          </span>
        )}
      </button>

      {isOpen && (
        <div className="absolute right-0 mt-2 w-80 bg-base-100 shadow-xl rounded-lg">
          <div className="flex justify-between p-3 border-b">
            <h3>Notifications</h3>
            <button onClick={markAllRead}>Mark all read</button>
          </div>
          <div className="max-h-96 overflow-y-auto">
            {notifications.map(n => (
              <div
                key={n.id}
                className={`p-3 border-b cursor-pointer ${!n.read ? 'bg-base-200' : ''}`}
                onClick={() => markAsRead(n.id)}
                dangerouslySetInnerHTML={{ __html: n.message }}
              />
            ))}
          </div>
        </div>
      )}
    </div>
  );
}
```

**For each issue found, specify:**
1. Category (correctness/performance/security/accessibility/maintainability)
2. The problematic code
3. What could go wrong
4. How to fix it

---

## Submission Format

Save your answers as `submission.json`:

```json
{
  "candidate": {
    "name": "Your Name",
    "email": "your.email@nsut.ac.in"
  },
  "section_a": {
    "bugs": [
      {
        "location": "line X or code snippet",
        "runtime_behavior": "what goes wrong",
        "underlying_issue": "why this is a problem",
        "fix": "corrected code"
      }
    ]
  },
  "section_b": {
    "issues": [
      {
        "issue": "description",
        "why_problematic": "explanation",
        "fix_approach": "how to fix",
        "react_concepts": ["useMemo", "useCallback", etc]
      }
    ]
  },
  "section_c": {
    "props_interface": "TypeScript interface",
    "state_structure": "description of state",
    "accessibility": ["consideration 1", "consideration 2"],
    "component_logic": "pseudo-code or description",
    "error_handling": "how errors are handled"
  },
  "section_d": {
    "issues": [
      {
        "category": "security|performance|correctness|accessibility|maintainability",
        "code": "problematic snippet",
        "problem": "what could go wrong",
        "fix": "how to fix"
      }
    ]
  }
}
```

---

## Evaluation Criteria

| Criteria | Weight |
|----------|--------|
| Technical accuracy | 40% |
| Depth of understanding (explains WHY, not just WHAT) | 30% |
| Practical awareness (edge cases, real-world concerns) | 20% |
| Communication clarity | 10% |

**What distinguishes great answers:**
- Explains the underlying concepts, not just the fix
- Considers edge cases and real-world usage
- Shows awareness of trade-offs
- Demonstrates understanding of React's mental model
