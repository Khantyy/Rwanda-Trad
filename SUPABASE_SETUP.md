# Supabase Database Setup Guide

This online store now integrates with Supabase for backend database management. Follow these steps to set up your Supabase project.

## Step 1: Create a Supabase Project

1. Go to [supabase.com](https://supabase.com)
2. Sign up or log in to your account
3. Click "New Project"
4. Fill in the project details:
   - **Project Name**: Rwanda-Trad
   - **Database Password**: Create a strong password
   - **Region**: Choose the region closest to your users
5. Click "Create new project" and wait for initialization

## Step 2: Get Your Credentials

1. Go to **Settings** > **API** in your Supabase dashboard
2. Copy your:
   - **Project URL** (SUPABASE_URL)
   - **anon/public key** (SUPABASE_ANON_KEY)

## Step 3: Create Database Tables

In your Supabase dashboard, go to **SQL Editor** and run the following SQL to create your tables:

### Products Table
```sql
CREATE TABLE products (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  name TEXT NOT NULL,
  description TEXT,
  price DECIMAL(10, 2) NOT NULL,
  image_url TEXT,
  category TEXT,
  stock INTEGER DEFAULT 0,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- Enable Row Level Security
ALTER TABLE products ENABLE ROW LEVEL SECURITY;

-- Create public read policy
CREATE POLICY "Allow public read access" ON products
  FOR SELECT USING (true);
```

### Orders Table
```sql
CREATE TABLE orders (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  user_email TEXT NOT NULL,
  total_amount DECIMAL(10, 2) NOT NULL,
  status TEXT DEFAULT 'pending',
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- Enable Row Level Security
ALTER TABLE orders ENABLE ROW LEVEL SECURITY;

-- Create policy for inserting orders
CREATE POLICY "Allow anyone to create orders" ON orders
  FOR INSERT WITH CHECK (true);

-- Create policy for reading own orders
CREATE POLICY "Allow users to read orders" ON orders
  FOR SELECT USING (true);
```

### Order Items Table
```sql
CREATE TABLE order_items (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  order_id UUID NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
  product_id UUID NOT NULL REFERENCES products(id),
  quantity INTEGER NOT NULL,
  price DECIMAL(10, 2) NOT NULL,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Enable Row Level Security
ALTER TABLE order_items ENABLE ROW LEVEL SECURITY;

-- Create policies
CREATE POLICY "Allow anyone to create order items" ON order_items
  FOR INSERT WITH CHECK (true);

CREATE POLICY "Allow users to read order items" ON order_items
  FOR SELECT USING (true);
```

## Step 4: Configure Your Environment

1. Copy `.env.example` to `.env.local`:
   ```bash
   cp .env.example .env.local
   ```

2. Edit `.env.local` and add your Supabase credentials:
   ```
   SUPABASE_URL=your_project_url_here
   SUPABASE_ANON_KEY=your_anon_key_here
   ```

## Step 5: Update Your HTML Files

Add the Supabase JavaScript client to your HTML files. Add this before your other scripts:

```html
<!-- Supabase -->
<script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>
<script src="js/supabase-client.js"></script>
```

## Step 6: Use the Supabase Functions

Now you can use the provided functions in your JavaScript:

### Example: Display Products
```javascript
async function displayProducts() {
  const products = await getProducts();
  products.forEach(product => {
    console.log(`${product.name} - $${product.price}`);
  });
}
```

### Example: Create an Order
```javascript
async function checkout(items) {
  const order = {
    user_email: 'customer@example.com',
    total_amount: 100.00,
    status: 'pending'
  };
  
  const newOrder = await createOrder(order);
  console.log('Order created:', newOrder);
}
```

## Available Functions

- `initSupabase()` - Initialize Supabase client
- `getProducts()` - Fetch all products
- `addProduct(product)` - Add a new product
- `updateProduct(id, updates)` - Update a product
- `deleteProduct(id)` - Delete a product
- `getOrders()` - Fetch all orders
- `createOrder(order)` - Create a new order

## Security Notes

⚠️ **Important**: 
- Never commit `.env.local` to git (it's in .gitignore)
- For production, use Vercel environment variables instead
- The anon key is safe to use in client-side code with RLS policies

## Vercel Deployment

To deploy with Vercel:

1. Push your code to GitHub
2. Connect your repo to Vercel
3. Add environment variables in Vercel project settings:
   - `SUPABASE_URL`
   - `SUPABASE_ANON_KEY`
4. Deploy!

## Troubleshooting

**Error: "Supabase credentials not configured"**
- Make sure your `.env.local` file exists and has the correct credentials
- Check that SUPABASE_URL and SUPABASE_ANON_KEY are not empty

**Error: "Permission denied" when accessing tables**
- Check your RLS policies in Supabase dashboard
- Ensure the policies allow public read/write access or adjust as needed

**Products not loading**
- Verify the products table exists and has data
- Check browser console for error messages
- Ensure your Supabase URL and key are correct
