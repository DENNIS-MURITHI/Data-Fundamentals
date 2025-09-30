# 📖 Data Fundamentals: Admin & User Roles in Music Streaming DB

<div align="center">
  <img width="314" height="285" alt="Supabase Logo" src="https://github.com/user-attachments/assets/20661293-a214-4004-9042-657102fb0710" />
  <br/>
  <h3><b>Data Fundamentals Project</b></h3>
</div>

---

## 📗 Table of Contents

* [📖 About the Project](#about-project)  
* [🛠 Built With](#built-with)  
* [🚀 Live Demo](#live-demo)  
* [💻 Getting Started](#getting-started)  
* [💾 Sample SQL Queries & Policies](#sample-sql-queries)   
* [👥 Authors](#authors)  
* [🔭 Future Features](#future-features)  
* [🤝 Contributing](#contributing)  
* [⭐️ Show your support](#support)  
* [🙏 Acknowledgements](#acknowledgements)  
* [❓ FAQ](#faq)  
* [📝 License](#license)  

---

# 📖 About the Project <a name="about-project"></a>

> This project models a **music streaming database** while demonstrating **admin and user roles** using **Row Level Security (RLS)** in Supabase.  
> Students can explore CRUD operations, user access restrictions, and admin-only functions.

**Key Goals:**

- Implement least privilege access  
- Apply policies for users and admins  
- Test queries safely in Supabase  

---

## 🛠 Built With <a name="built-with"></a>

- **Supabase Dashboard** – SQL editor & authentication  
- **PostgreSQL** – database and tables  
- **RLS Policies & Functions** – enforce admin/user restrictions  

---

## 🚀 Live Demo <a name="live-demo"></a>

- [Supabase Dashboard Link](https://app.supabase.com)  

---

## 💻 Getting Started <a name="getting-started"></a>

### Prerequisites

- Supabase account  
- PostgreSQL knowledge  
- Git installed  

### Setup

```bash
git clone https://github.com/DENNIS-MURITHI/music-streaming-database.git
cd music-streaming-database
```

### Usage

1. Open Supabase Dashboard  
2. Execute `schema.sql` to create tables & sample data  
3. Enable **RLS** on tables:

```sql
ALTER TABLE user_favorites ENABLE ROW LEVEL SECURITY;
ALTER TABLE songs ENABLE ROW LEVEL SECURITY;
ALTER TABLE artists ENABLE ROW LEVEL SECURITY;
```

4. Add **policies** for admin & user roles.

---

## 💾 Sample SQL Queries & Policies <a name="sample-sql-queries"></a>

### 1️⃣ User Policies

```sql
-- Users can view only their own favorites
CREATE POLICY "Users can view their own favorites"
ON user_favorites
FOR SELECT
USING (auth.uid() = user_id);

-- Users can insert only their own favorites
CREATE POLICY "Users can insert their own favorites"
ON user_favorites
FOR INSERT
WITH CHECK (auth.uid() = user_id);

-- Users can read all songs & artists
CREATE POLICY "Users can read all songs"
ON songs
FOR SELECT
USING (true);

CREATE POLICY "Users can read all artists"
ON artists
FOR SELECT
USING (true);
```

### 2️⃣ Admin Policies

```sql
-- Admins can manage all favorites
CREATE POLICY "Admins can manage all favorites"
ON user_favorites
FOR ALL
USING (EXISTS (SELECT 1 FROM users WHERE id = auth.uid() AND role = 'admin'));

-- Admins can manage all songs & artists
CREATE POLICY "Admins can manage all songs"
ON songs
FOR ALL
USING (EXISTS (SELECT 1 FROM users WHERE id = auth.uid() AND role = 'admin'));

CREATE POLICY "Admins can manage all artists"
ON artists
FOR ALL
USING (EXISTS (SELECT 1 FROM users WHERE id = auth.uid() AND role = 'admin'));
```

### 3️⃣ Example CRUD Queries

```sql
-- List all songs liked by Alice
SELECT u.username, s.title, a.name AS artist_name
FROM user_favorites uf
JOIN users u ON uf.user_id = u.user_id
JOIN songs s ON uf.song_id = s.song_id
JOIN artists a ON s.artist_id = a.artist_id
WHERE u.username = 'alice';

-- Find all songs by Sauti Sol
SELECT s.title, s.release_year
FROM songs s
JOIN artists a ON s.artist_id = a.artist_id
WHERE a.name = 'Sauti Sol';

-- Most popular artist (aggregate)
SELECT a.name, COUNT(uf.user_id) AS total_favorites
FROM artists a
JOIN songs s ON a.artist_id = s.artist_id
LEFT JOIN user_favorites uf ON s.song_id = uf.song_id
GROUP BY a.artist_id
ORDER BY total_favorites DESC;

-- Insert favorite (User only)
INSERT INTO user_favorites (user_id, song_id)
VALUES ('<alice-uid>', '<song-id>');

-- Update or delete songs (Admin only)
UPDATE songs SET title = 'New Song Title' WHERE song_id = '<song-id>';
DELETE FROM songs WHERE song_id = '<song-id>';
```

### 4️⃣ Admin-only Function Example

```sql
CREATE OR REPLACE FUNCTION delete_song_safe(song_id INT)
RETURNS VOID
LANGUAGE SQL
SECURITY DEFINER
AS $$
    DELETE FROM songs WHERE song_id = $1;
$$;
```

---

## 👥 Authors <a name="authors"></a>

**Dennis Murithi**  
- GitHub: [@dennismurithi](https://github.com/DENNIS-MURITHI)  
- LinkedIn: [Dennis Murithi](https://www.linkedin.com/in/dennis-muthuri/)  

---

## 🔭 Future Features <a name="future-features"></a>

- Integrate with front-end music streaming app  
- Add analytics for popular songs/artists  
- Implement audit logging for admin actions  

---

## 🤝 Contributing <a name="contributing"></a>

Open issues or pull requests are welcome.

---

## ⭐️ Show your support <a name="support"></a>

Give a ⭐️ if you like this project!

---

## 🙏 Acknowledgements <a name="acknowledgements"></a>

- Supabase docs for SQL & RLS policies  
- PostgreSQL official docs   
---

## ❓ FAQ <a name="faq"></a>

**Q:** How do I test RLS policies?  
**A:** Sign in as User vs Admin and try CRUD operations. Policies will restrict or allow access accordingly.

**Q:** Can I extend this to a front-end?  
**A:** Yes, connect Supabase Auth with React, Next.js, or any front-end framework.

---

## 📝 License <a name="license"></a>
This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

