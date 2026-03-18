README File for this video 
https://www.youtube.com/watch?v=WTw6NZDZ5mQ

# 🛒 Next.js E-commerce Application

This is a **Next.js e-commerce application** bootstrapped with `create-next-app`.

---

## 🚀 Getting Started

### 1⃣ Install Dependencies

```bash
npm install
```

### 2⃣ Run Development Server

```bash
npm run dev
```

### 3⃣ Environment Setup ⚠

* Replace all **environment variables**
* Replace **Bearer keys / API keys** in:

  * `app/api/payment`
  * `app/api/verifyPayment`

> ❗ DO NOT use the original keys — use your own.

---

### 🌐 Open App

Visit:

```
http://localhost:3000
```

---

### ✏ Editing

Start editing here:

```
app/page.tsx
```

---

## 📦 Database Setup (SQL)

---

# ⭐ REVIEWS TABLE

```sql
CREATE TABLE reviews (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    order_id uuid REFERENCES orders(id) ON DELETE CASCADE NOT NULL UNIQUE,
    user_id uuid REFERENCES auth.users(id) ON DELETE CASCADE NOT NULL,
    review_images text[],
    review_title text,
    review_description text,
    product_rating numeric DEFAULT 5,
    delivery_rating numeric DEFAULT 5,
    product_image_url text,
    product_name text,
    amount_paid numeric,
    the_quantity int,
    created_at timestamp DEFAULT now(),
    updated_at timestamp DEFAULT now()
);
```

### 🔐 RLS Policies

```sql
ALTER TABLE reviews ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Readable by User."
ON reviews FOR SELECT
USING ((SELECT auth.uid()) = user_id);

CREATE POLICY "Users can insert their own data."
ON reviews FOR INSERT
WITH CHECK ((SELECT auth.uid()) = user_id);

CREATE POLICY "Users can update own data."
ON reviews FOR UPDATE
USING ((SELECT auth.uid()) = user_id);

CREATE POLICY "Users can delete own data."
ON reviews FOR DELETE
USING ((SELECT auth.uid()) = user_id);
```

---

# 🖼 REVIEW STORAGE

```sql
INSERT INTO storage.buckets (id, name)
VALUES ('review_bucket', 'review_bucket');

CREATE POLICY "review images are publicly accessible."
ON storage.objects FOR SELECT
USING (bucket_id = 'review_bucket');

CREATE POLICY "Anyone can upload a review image."
ON storage.objects FOR INSERT
WITH CHECK (bucket_id = 'review_bucket');
```

---

# 📦 ORDERS TABLE

```sql
CREATE TABLE orders (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id uuid NOT NULL,
    user_email text,
    product_name TEXT NOT NULL,
    product_category TEXT,
    amount_paid NUMERIC(10, 2) NOT NULL,
    reference_paystack text NOT NULL,
    quantity_bought INTEGER NOT NULL,
    image_url TEXT NOT NULL,
    status TEXT NOT NULL CHECK (
        status IN ('processing', 'completed', 'cancelled', 'shipped', 'delivered', 'returned', 'waiting', 'reviewed')
    ),
    size TEXT,
    color TEXT,
    region TEXT,
    state text,
    city text,
    address TEXT NOT NULL,
    phone TEXT NOT NULL,
    country_code text,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    CONSTRAINT fk_user FOREIGN KEY (user_id) REFERENCES auth.users(id)
);
```

### 🔐 RLS

```sql
ALTER TABLE orders ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Readable by User."
ON orders FOR SELECT
USING ((SELECT auth.uid()) = user_id);

CREATE POLICY "Users can insert their own data."
ON orders FOR INSERT
WITH CHECK ((SELECT auth.uid()) = user_id);

CREATE POLICY "Users can update own data."
ON orders FOR UPDATE
USING ((SELECT auth.uid()) = user_id);

CREATE POLICY "Users can delete own data."
ON orders FOR DELETE
USING ((SELECT auth.uid()) = user_id);
```

---

# 📍 ADDRESS TABLE

```sql
CREATE TABLE address (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id uuid NOT NULL,
    title TEXT,
    region TEXT DEFAULT 'Nigeria',
    address TEXT,
    state text,
    city text,
    phone TEXT,
    country_code TEXT,
    flag TEXT,
    is_default boolean DEFAULT false,
    created_at timestamp DEFAULT now(),
    CONSTRAINT fk_user FOREIGN KEY (user_id)
    REFERENCES auth.users(id) ON DELETE CASCADE
);
```

### 🔐 RLS

```sql
ALTER TABLE address ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Readable by User."
ON address FOR SELECT
USING ((SELECT auth.uid()) = user_id);

CREATE POLICY "Users can insert their own data."
ON address FOR INSERT
WITH CHECK ((SELECT auth.uid()) = user_id);

CREATE POLICY "Users can update own data."
ON address FOR UPDATE
USING ((SELECT auth.uid()) = user_id);
```

---

# ⚙ ADDRESS DEFAULT TRIGGER

```sql
CREATE OR REPLACE FUNCTION enforce_single_default_address()
RETURNS TRIGGER AS $$
BEGIN
    UPDATE address
    SET is_default = false
    WHERE user_id = NEW.user_id
      AND id <> NEW.id;

    RETURN NEW;
END;
$$ LANGUAGE plpgsql;
```

```sql
CREATE TRIGGER on_default_address_set
AFTER UPDATE ON address
FOR EACH ROW
WHEN (NEW.is_default = true)
EXECUTE FUNCTION enforce_single_default_address();
```

---

# 🗂 CATEGORIES

```sql
CREATE TABLE categories (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);
```

```sql
ALTER TABLE categories ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Readable by everyone."
ON categories FOR SELECT
USING (true);
```

---

# 🛍 PRODUCTS

```sql
CREATE TABLE products (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    author_id UUID REFERENCES auth.users(id) ON DELETE CASCADE NOT NULL,
    sizes text[],
    colors text[],
    styles text[],
    brand TEXT,
    image_url_array TEXT[] NOT NULL,
    video_url_array TEXT[] DEFAULT ARRAY[]::TEXT[],
    name TEXT NOT NULL,
    category uuid NOT NULL,
    price NUMERIC(10, 2) NOT NULL,
    description TEXT,
    discount NUMERIC(5, 2) DEFAULT 0,
    quantity INTEGER NOT NULL,
    product_shipping_fee integer,
    offer_price numeric(10, 2),
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    location text,
    product_comment text,
    CONSTRAINT fk_category FOREIGN KEY (category) REFERENCES categories(id)
);
```

```sql
ALTER TABLE products ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Readable by everyone."
ON products FOR SELECT
USING (true);
```

---

# 👤 ELDICS USERS

```sql
CREATE TABLE eldics_users (
    id uuid REFERENCES auth.users ON DELETE CASCADE NOT NULL PRIMARY KEY,
    full_name text,
    avatar_url text,
    username text,
    email text UNIQUE,
    created_at timestamp DEFAULT now(),
    updated_at timestamp with time zone,
    CONSTRAINT username_length CHECK (char_length(username) >= 3)
);
```

### 🔐 RLS

```sql
ALTER TABLE eldics_users ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Readable by everyone."
ON eldics_users FOR SELECT
USING (true);

CREATE POLICY "Users can insert their own data."
ON eldics_users FOR INSERT
WITH CHECK ((SELECT auth.uid()) = id);

CREATE POLICY "Users can update own data."
ON eldics_users FOR UPDATE
USING ((SELECT auth.uid()) = id);
```

---

# 🔄 EMAIL SYNC FUNCTION

```sql
CREATE FUNCTION public.updating_auth_user_email()
RETURNS trigger
SET search_path = 'public'
AS $$
BEGIN
    UPDATE auth.users
    SET email = NEW.email
    WHERE id = NEW.id;

    RETURN NEW;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;
```

```sql
CREATE TRIGGER on_eldics_users_email_update
AFTER UPDATE ON public.eldics_users
FOR EACH ROW
WHEN (OLD.email IS DISTINCT FROM NEW.email)
EXECUTE FUNCTION public.updating_auth_user_email();
```

---

# 👶 NEW USER TRIGGER

```sql
CREATE FUNCTION public.handling_new_user()
RETURNS trigger
SET search_path = ''
AS $$
BEGIN
    IF NEW.raw_user_meta_data->>'avatar_url' IS NULL
       OR NEW.raw_user_meta_data->>'avatar_url' = '' THEN
        NEW.raw_user_meta_data = jsonb_set(
            NEW.raw_user_meta_data,
            '{avatar_url}',
            '"https://example.com/user.png"'::jsonb
        );
    END IF;

    INSERT INTO public.eldics_users (id, email, avatar_url)
    VALUES (
        NEW.id,
        NEW.email,
        NEW.raw_user_meta_data->>'avatar_url'
    );

    RETURN NEW;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;
```

```sql
CREATE OR REPLACE TRIGGER on_the_auth_user_verified
AFTER UPDATE ON auth.users
FOR EACH ROW
WHEN (OLD.email_confirmed_at IS NULL AND NEW.email_confirmed_at IS NOT NULL)
EXECUTE PROCEDURE public.handling_new_user();
```

---

# 🗄 AVATAR STORAGE

```sql
INSERT INTO storage.buckets (id, name)
VALUES ('avatars', 'avatars');

CREATE POLICY "Avatar images are publicly accessible."
ON storage.objects FOR SELECT
USING (bucket_id = 'avatars');

CREATE POLICY "Anyone can upload an avatar."
ON storage.objects FOR INSERT
WITH CHECK (bucket_id = 'avatars');
```


