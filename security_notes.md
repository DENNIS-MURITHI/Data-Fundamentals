# Security Notes -- Data Fundamentals Project

This document provides detailed notes on **security setup, RLS, roles,
and policies** for the Music Streaming Database project.

------------------------------------------------------------------------

## 🔑 UUID Setup

We switched from integer IDs to UUIDs to align with **Supabase Auth**
and ensure globally unique identifiers.

``` sql
CREATE EXTENSION IF NOT EXISTS "pgcrypto";

ALTER TABLE users
ADD COLUMN user_uuid UUID DEFAULT gen_random_uuid(),
ADD CONSTRAINT uq_users_uuid UNIQUE (user_uuid);
```

UUIDs are then linked across tables (e.g., `user_favorites`) to replace
integer foreign keys.

------------------------------------------------------------------------

## 🔒 Row Level Security (RLS)

RLS ensures **least privilege access**. It restricts what each role
(User/Admin) can read, insert, update, or delete.

``` sql
ALTER TABLE user_favorites ENABLE ROW LEVEL SECURITY;
ALTER TABLE songs ENABLE ROW LEVEL SECURITY;
ALTER TABLE artists ENABLE ROW LEVEL SECURITY;
```

------------------------------------------------------------------------

## 👤 User Policies

Normal users have restricted access.

``` sql
-- View only their own favorites
CREATE POLICY "Users can view their own favorites"
ON user_favorites
FOR SELECT
USING (auth.uid() = user_uuid);

-- Insert only their own favorites
CREATE POLICY "Users can insert their own favorites"
ON user_favorites
FOR INSERT
WITH CHECK (auth.uid() = user_uuid);

-- Read all songs & artists
CREATE POLICY "Users can read all songs"
ON songs FOR SELECT USING (true);

CREATE POLICY "Users can read all artists"
ON artists FOR SELECT USING (true);
```

------------------------------------------------------------------------

## 👨‍💼 Admin Policies

Admins have **full access**.

``` sql
CREATE POLICY "Admins can manage all favorites"
ON user_favorites
FOR ALL
USING (EXISTS (SELECT 1 FROM users WHERE user_uuid = auth.uid() AND role = 'admin'));

CREATE POLICY "Admins can manage all songs"
ON songs
FOR ALL
USING (EXISTS (SELECT 1 FROM users WHERE user_uuid = auth.uid() AND role = 'admin'));

CREATE POLICY "Admins can manage all artists"
ON artists
FOR ALL
USING (EXISTS (SELECT 1 FROM users WHERE user_uuid = auth.uid() AND role = 'admin'));
```

------------------------------------------------------------------------

## ⚡️ Testing Roles & Policies

### ✅ User Tests

1.  **SELECT own favorites** (works)\
2.  **INSERT a favorite** (works)\
3.  **UPDATE a favorite** (blocked)\
4.  **DELETE a favorite** (blocked)

### ✅ Admin Tests

1.  **SELECT all favorites** (works)\
2.  **UPDATE songs** (works)\
3.  **DELETE artists** (works)

------------------------------------------------------------------------

## 🛠 Admin-only Function

Example: Admin deletes a project or song safely.

``` sql
CREATE OR REPLACE FUNCTION delete_song_safe(song_id INT)
RETURNS VOID
LANGUAGE SQL
SECURITY DEFINER
AS $$
  DELETE FROM songs WHERE song_id = $1;
$$;
```

------------------------------------------------------------------------

## 📎 Reference

-   Linked to [README.md](README.md)\
-   Supabase Docs: <https://supabase.com/docs>
