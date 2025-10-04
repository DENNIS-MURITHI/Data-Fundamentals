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
* [🛡 Security Notes](#security-notes)  
* [👥 Authors](#authors)  
* [🔭 Future Features](#future-features)  
* [🤝 Contributing](#contributing)  
* [⭐️ Show your support](#support)  
* [🙏 Acknowledgements](#acknowledgements)  
* [❓ FAQ](#faq)  
* [📝 License](#license)  

---

# 📖 About the Project <a name="about-project"></a>

This project models a **music streaming database** while demonstrating **admin and user roles** using **Row Level Security (RLS)** in Supabase.  

It showcases:  
- ✅ UUID-based user authentication (linked with Supabase Auth)  
- ✅ Admin vs User privileges with least privilege enforcement  
- ✅ CRUD operations with RLS enforcement  
- ✅ A tested SQL script to validate roles and policies
- ✅ Output from Supabase based on roles and policies

---

## 🛠 Built With <a name="built-with"></a>

- **Supabase Dashboard** – SQL editor & authentication  
- **PostgreSQL** – database and tables  
- **RLS Policies & Functions** – enforce admin/user restrictions  

---

## 🚀 Live Demo <a name="live-demo"></a>

- [Supabase Dashboard](https://app.supabase.com)  

---

## 💻 Getting Started <a name="getting-started"></a>

### Prerequisites
- Supabase account  
- PostgreSQL basics  
- Git installed  

### Setup

```bash
git clone https://github.com/DENNIS-MURITHI/music-streaming-database.git
cd music-streaming-database
```

```bash
Usage

Open Supabase SQL editor

Run schema.sql to create tables & sample data

Apply UUID + RLS setup:

ALTER TABLE user_favorites ENABLE ROW LEVEL SECURITY;
ALTER TABLE songs ENABLE ROW LEVEL SECURITY;
ALTER TABLE artists ENABLE ROW LEVEL SECURITY;


Apply user vs admin policies.
```

---

## 💾 Sample SQL Queries & Policies <a name="sample-sql-queries"></a>

### 1️⃣ User Policies
```sql
-- Users can view only their own favorites
CREATE POLICY "Users can view their own favorites"
ON user_favorites
FOR SELECT
USING (auth.uid() = user_uuid);
```

```sql
-- Users can insert their own favorites
CREATE POLICY "Users can insert their own favorites"
ON user_favorites
FOR INSERT
WITH CHECK (auth.uid() = user_uuid);
```

```sql
-- Users can read all songs & artists
CREATE POLICY "Users can read all songs"
ON songs
FOR SELECT
USING (true);
```
```sql
CREATE POLICY "Users can read all artists"
ON artists
FOR SELECT
USING (true);
```

---

### 2️⃣ Admin Policies
```sql
-- Admins can manage all favorites
CREATE POLICY "Admins can manage all favorites"
ON user_favorites
FOR ALL
USING (EXISTS (SELECT 1 FROM users WHERE user_uuid = auth.uid() AND role = 'admin'));
```

```sql
-- Admins can manage all songs
CREATE POLICY "Admins can manage all songs"
ON songs
FOR ALL
USING (EXISTS (SELECT 1 FROM users WHERE user_uuid = auth.uid() AND role = 'admin'));
```

```sql
-- Admins can manage all artists
CREATE POLICY "Admins can manage all artists"
ON artists
FOR ALL
USING (EXISTS (SELECT 1 FROM users WHERE user_uuid = auth.uid() AND role = 'admin'));
```

---

### 3️⃣ Example CRUD Queries with their output.
```sql
-- List all songs liked by Alice
SELECT u.username, s.title, a.name AS artist_name
FROM user_favorites uf
JOIN users u ON uf.user_uuid = u.user_uuid
JOIN songs s ON uf.song_id = s.song_id
JOIN artists a ON s.artist_id = a.artist_id
WHERE u.username = 'alice';
```
![Output for Alice’s Favorites](https://github.com/user-attachments/assets/9870a7c2-755d-4c78-af92-12c3b02ba315)

## User Roles and Output

**User add new songs to their favorited**
```sql
-- Insert favorite (User only)
INSERT INTO user_favorites (user_uuid, song_id)
VALUES ('2d953804-5827-4f73-bfd5-41d83d53762f', 8);
```
```sql
SELECT s.title AS song, a.name AS artist
FROM user_favorites uf
JOIN songs s ON uf.song_id = s.song_id
JOIN artists a ON s.artist_id = a.artist_id
WHERE uf.user_uuid = '2d953804-5827-4f73-bfd5-41d83d53762f';
```
  **Ouput upon updating their favorited**

![Output after Insert Favorite](https://github.com/user-attachments/assets/56d8d4d4-639e-4f17-baf3-0d501b8d3b96)

  **User can also delete their favorite songs**

```sql
DELETE FROM user_favorites
WHERE user_uuid = '2d953804-5827-4f73-bfd5-41d83d53762f'  AND song_id = 8;
```
**Output upon deleting song with id=8**

![Output after user deletes a specific songs](https://github.com/user-attachments/assets/794f8a04-ef31-4d8b-800b-8bf91cb6588f)



## Admin Roles and Output

**A view of the songs before updating**

```sql
SELECT song_id, title, artist_id
FROM songs
WHERE song_id = 1;
```
![Output of the song before the title is updated by Admin](https://github.com/user-attachments/assets/3cc4c88f-265f-4468-968c-0c2054b06f17)


**Song Output after Admin update the title**
```sql
-- Update a song (Admin only)
UPDATE songs SET title = 'Programmers choice' WHERE song_id = 1;
```
![Output after Update Song](https://github.com/user-attachments/assets/9f258a69-1202-41a1-b0c9-a7bf2722a3f8)

```sql
-- Delete a song (Admin only)
DELETE FROM artists WHERE song_id = 1;
```
![Output after Delete Artist](your-screenshot-link-here-10)

---

## 🛡 Security Notes <a name="security-notes"></a>

See full explanation of RLS, policies, and admin functions in 👉 [security_notes.md](https://github.com/DENNIS-MURITHI/Data-Fundamentals/blob/data_test_branch/security_notes.md)


---

## 👥 Authors <a name="authors"></a>

- **Dennis Murithi**  
  GitHub: [@dennismurithi](https://github.com/dennismurithi)  
  LinkedIn: [Dennis Murithi](https://www.linkedin.com/in/dennis-murithi)  

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

**Q: How do I test RLS policies?**  
A: Sign in as User vs Admin and try CRUD operations. Policies will restrict or allow access accordingly.  

**Q: Can I extend this to a front-end?**  
A: Yes, connect Supabase Auth with React, Next.js, or any front-end framework.  

---

## 📝 License <a name="license"></a>

This project is licensed under the MIT License - see the LICENSE file for details.

