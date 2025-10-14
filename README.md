# remembering: class: 8 Learning

# Software:
```
# PostgreSQL
https://www.postgresql.org/

# Valentina Studio
https://www.valentina-db.com/en/download-valentina-studio

```
# SQL Query

CREATE DATABASE titanMedical;

CREATE TABLE user_table(
 user_id SERIAL,
first_name VARCHAR(50),
last_name VARCHAR(50),
 age INTEGER,
 isActive BOOLEAN,
 dob DATE
 )

ALTER TABLE "user_table" Rename to "user";

DROP TABLE "user";

## primary key, not null, foreign key, int check and Unique 
CREATE TABLE students (
  student_id INT PRIMARY KEY,
  full_name VARCHAR(100) NOT NULL,
  email VARCHAR(100) UNIQUE NOT NULL,
  department_id INT,
  FOREIGN KEY (department_id) REFERENCES departments(department_id),
   status VARCHAR(20) DEFAULT 'Active' ,
   admission_date DATE DEFAULT CURRENT_DATE,
    age INT CHECK (age >= 18),         
  marks INT CHECK (marks BETWEEN 0 AND 100)
);

## inserting data
INSERT INTO table_name (column1, column2, column3, ...)
VALUES (value1, value2, value3, ...);


## Adding new col to the table
ALTER TABLE students
ADD COLUMN email VARCHAR(100); 









# PostgreSQL Essential Shortcuts & Cheatsheet (Mac-friendly)

> দ্রুত হাতে কলমে কাজের জন্য `psql` কমান্ড, মেটা-কমান্ড, সাধারণ SQL, ও ব্যাকআপ/রিস্টোর—সবচেয়ে দরকারি জিনিসগুলোর সংক্ষিপ্ত তালিকা।

---

## ##) Most-used Commands that I use

```text

# Login --- pass added: 123
psql -h localhost -U postgres

# pager to psql prompt
q    

# All database listings            
\l   

# New database create
CREATE DATABASE mydb OWNER postgres;                  

# Clear input buffer
\r

# See all users
\du


# create User
create user user1 with login encrypted password '123';

# remove user 
DROP ROLE user1;


# Connection between user and database
GRANT CONNECT ON DATABASE test1 TO user1;  

# create table
create table test_table (name varchar(50))

# check table
\d  

# Insert into table
insert into test_table(name) values('Kamal');

# check data into table
select * from test_table;


# giving access all privilege to the table
grant all privileges on test_table to user1;


#direct switch from postgres database to test1
\c test1


#direct switch from postgres database to test1 with user also
\c test1 user1

```

---


---

## 0) Quick Setup (Mac)

```bash
# EnterpriseDB installer হলে (ধরা: v18)
echo 'export PATH="/Library/PostgreSQL/18/bin:$PATH"' >> ~/.zshrc && source ~/.zshrc

# Homebrew হলে (Apple Silicon)
brew install postgresql@16
echo 'export PATH="/opt/homebrew/opt/postgresql@16/bin:$PATH"' >> ~/.zshrc && source ~/.zshrc

# Postgres.app হলে
echo 'export PATH="$PATH:/Applications/Postgres.app/Contents/Versions/latest/bin"' >> ~/.zshrc && source ~/.zshrc
```
> নোট: Intel Mac হলে Homebrew path সাধারণত `/usr/local/opt/postgresql@16/bin`।

---

## 1) Connect & Basics

```bash
psql --version
psql -h localhost -U postgres           # postgres ইউজার দিয়ে
psql -d mydb                             # ডাটাবেস নির্দিষ্ট করে
psql "postgresql://user:pass@localhost:5432/mydb"  # URI
```
- `psql` থেকে বের হওয়া: `\q`  
- হেল্প: `\?` (psql help), `\h` (SQL command help)

---

## 2) Most-used psql Meta-commands

```text
\l                  # ডাটাবেস লিস্ট
\c dbname           # ডাটাবেসে কানেক্ট
\dt                 # টেবিল লিস্ট (public schema)
\dt+                # টেবিল + সাইজ/ডিটেইল
\d table_name       # টেবিলের স্ট্রাকচার
\d+ table_name      # টেবিল + স্টোরেজ/ডিটেইল
\dn                 # স্কিমা লিস্ট
\du                 # রোল/ইউজার লিস্ট
\df                 # ফাংশন লিস্ট
\di                 # ইনডেক্স লিস্ট
\dv                 # ভিউ লিস্ট
\encoding           # এনকোডিং দেখায়/সেট করে
\conninfo           # কানেকশন ইনফো
\timing             # কুয়েরি টাইমিং টগল
\pset pager off     # লম্বা আউটপুটে পেজার বন্ধ
\watch 2            # শেষ কুয়েরি প্রতি 2 সেকেন্ডে রান
\! <cmd>            # শেল কমান্ড রান (যেমন \! clear)
\copy table TO 'file.csv' CSV HEADER      # ক্লায়েন্ট-সাইড কপি
\copy table FROM 'file.csv' CSV HEADER    # CSV ইমপোর্ট
```

---

## 3) User/Role & Database Management

```sql
-- সুপারইউজার তৈরি (টুল/অ্যাডমিনিস্ট্রেটিভ কাজের জন্য)
CREATE ROLE myadmin WITH LOGIN PASSWORD 'secret' SUPERUSER;

-- নরমাল ইউজার
CREATE ROLE app_user WITH LOGIN PASSWORD 'secret' NOSUPERUSER;

-- ডাটাবেস তৈরি ও মালিক সেট
CREATE DATABASE mydb OWNER app_user;

-- প্রিভিলেজ
GRANT CONNECT ON DATABASE mydb TO app_user;
GRANT USAGE ON SCHEMA public TO app_user;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO app_user;

-- ভবিষ্যতের টেবিলগুলোর ডিফল্ট প্রিভিলেজ
ALTER DEFAULT PRIVILEGES IN SCHEMA public
GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO app_user;

-- পাসওয়ার্ড বদল
ALTER ROLE app_user WITH PASSWORD 'newpass';
```

---

## 4) Table Essentials

```sql
-- টেবিল তৈরি
CREATE TABLE users (
  id BIGSERIAL PRIMARY KEY,
  email TEXT UNIQUE NOT NULL,
  name TEXT,
  created_at TIMESTAMPTZ DEFAULT now()
);

-- ইনডেক্স
CREATE INDEX idx_users_email ON users(email);

-- আল্টার
ALTER TABLE users ADD COLUMN is_active BOOLEAN DEFAULT true;

-- ড্রপ
DROP TABLE IF EXISTS users CASCADE;
```

---

## 5) CRUD Quickies

```sql
-- Insert
INSERT INTO users (email, name) VALUES ('a@b.com', 'Alice') RETURNING id;

-- Select
SELECT id, email, name FROM users WHERE is_active = true ORDER BY id DESC LIMIT 10;

-- Update
UPDATE users SET name = 'A. Doe' WHERE email = 'a@b.com';

-- Delete
DELETE FROM users WHERE id = 10;
```

---

## 6) Transactions

```sql
BEGIN;
  UPDATE accounts SET balance = balance - 100 WHERE id = 1;
  UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;

-- সমস্যা হলে
ROLLBACK;
```

---

## 7) Performance & Explain

```sql
EXPLAIN SELECT * FROM users WHERE email = 'a@b.com';
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'a@b.com';

VACUUM (VERBOSE, ANALYZE);
ANALYZE;   -- স্ট্যাটস আপডেট
```

---

## 8) Backup & Restore (pg_dump / pg_restore)

```bash
# সম্পূর্ণ ডাটাবেস (SQL টেক্সট ডাম্প)
pg_dump -h localhost -U postgres -d mydb -F p > mydb.sql

# কাস্টম ফরম্যাট (কমপ্রেসড), পরে pg_restore
pg_dump -h localhost -U postgres -d mydb -F c -f mydb.dump

# রিস্টোর (SQL)
psql -h localhost -U postgres -d mydb -f mydb.sql

# রিস্টোর (কাস্টম ডাম্প)
createdb mydb_restored
pg_restore -h localhost -U postgres -d mydb_restored --verbose mydb.dump
```

---

## 9) Service Control (Mac)

```bash
# EnterpriseDB ctl
sudo /Library/PostgreSQL/18/bin/pg_ctl -D /Library/PostgreSQL/18/data status
sudo /Library/PostgreSQL/18/bin/pg_ctl -D /Library/PostgreSQL/18/data start
sudo /Library/PostgreSQL/18/bin/pg_ctl -D /Library/PostgreSQL/18/data stop

# Homebrew
brew services list
brew services start postgresql@16
brew services stop postgresql@16
```

---

## 10) Troubleshooting Quick Checks

```bash
which -a psql                 # psql কোথায় আছে?
psql -l                       # ডাটাবেস তালিকা (কানেক্ট হলে)
\conninfo                     # psql-এর ভেতরে কানেকশন তথ্য
SHOW data_directory;          # ডাটা ডিরেক্টরি
SHOW port;                    # চলমান পোর্ট (ডিফল্ট 5432)
```
**Common issues**
- `psql: command not found` → PATH ঠিক করো (উপরে 0 নম্বর সেকশন)  
- `FATAL: password authentication failed` → ইউজার/পাস ঠিক আছে কিনা চেক, `pg_hba.conf` কনফিগ  
- কানেক্ট না হলে: সার্ভিস চলছে কিনা দেখো (`pg_ctl status` / `brew services list`)

---

## 11) Handy SQL Snippets

```sql
-- ইউনিক কনফ্লিক্টে আপসার্ট
INSERT INTO users (email, name)
VALUES ('a@b.com', 'Alice')
ON CONFLICT (email) DO UPDATE SET name = EXCLUDED.name;

-- পেজিনেশন
SELECT * FROM users ORDER BY id DESC LIMIT 10 OFFSET 20;

-- JSON
SELECT id, data->>'title' AS title FROM articles WHERE (data->>'published')::boolean = true;
```

---

## 12) psql Quality-of-life

```text
\set ECHO_HIDDEN on     # রান হওয়ার আগে এক্সপ্যান্ডেড কুয়েরি দেখায়
\pset format aligned    # সুন্দর টেবিল ফরম্যাট
\pset border 2
\gx                     # শেষ কুয়েরি আবার রান
\o out.txt              # আউটপুট ফাইলে
\o                      # আউটপুট ফাইল বন্ধ
```

---

### লাইসেন্স
এই চিটশিট যেখানে দরকার কপি–পেস্ট করে ব্যবহার করতে পারো। 👍
