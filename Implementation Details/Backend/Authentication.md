## Two Main Types of Authentication
### 1. Session-based authentication (stateful)
	How it works 
	1. User logs in 
	2. Backend verifies credentials
	3. Backend creates a session record in database or cache (Redis)
	4. Backend gives the user a session cookie
	5. On every request
		- Browser automatically sends the cookie 
		- Backend looks up session_id 
		- If found -> user authenticated
	Why is it called stateful - server remembers sessions
		- Session store containes active sessions
		- you can revoke the session anytime
		- you can track session expiration, lockouts, multi-device login

### 2. Token-based authentication (JWT) (stateless)

	How it works 
	1. User logs in 
	2. Backend verifies credentials 
	3. Backend creates JWT (Json Web Token)
	4. Backend signs it with a secret key and returns it to the user
	5. User stores JWT in:
		- Local Storage
		- Session Storage
		- Memory
	6. On every request:
		- Frontend includes: Authentication: Bearer <jwt>
		- Backend verifies signature
		- Backend does NOT look up anything in the backend
	Why it is called stateless - server doesn't store any auth state

## History 
### 1990–1993 — HTTP was stateless
HTTP was originally designed so that:
> Every request is independent with no memory.
Meaning:
- No login
- No user state
- No concept of a session
- Each request forgets who you are
The early web = static HTML pages only.
### 1994 — Netscape invents HTTP cookies
This is _the_ breakthrough that made authentication possible.
Cookies allowed browsers to store **small pieces of data** automatically sent to servers on every request.
For the first time, websites could:
- Keep users logged in
- Save user preferences
- Track who is adding items to a cart
This era gave birth to **login systems**.
### Early Authentication (1995–1999) — "Store user_id in cookie"
Primitive systems did: `Set-Cookie: user_id=123`
But this was **insecure**:
- User can modify cookies in browser
- Users could impersonate each other
- No encryption, no signature    
- No expiration control
So developers needed something safer.

### Evolution #1 (2000): Server-side sessions
Instead of storing user info in cookies, developers realized:
> “Let’s store only a random unpredictable session_id in the cookie,  
> and keep all sensitive session data on the server.”

Why a database was originally used for sessions
Early systems stored sessions in:
- file system
- in-memory dict in the application
- MySQL / PostgreSQL tables
##### ❌ Problem 1 — When server restarts, in-memory sessions disappear
Users get logged out.
##### ❌ Problem 2 — When scaling to multiple servers, sessions must be shared
Otherwise:
- user logs in on server A → works
- next request hits server B → session is missing
##### ❌ Problem 3 — Databases are slow for session reads/writes
Every request needs authentication checking, which means:
- lookup session_id
- update expiration
- maybe store new data
Database overhead becomes large.

### Evolution #2 (2010–2015): Redis becomes the standard
Redis solves all session problems:
##### ✔ In-memory → extremely fast
##### ✔ Persistent enough (optional)
##### ✔ Built-in expiration (TTL)
##### ✔ Safe concurrency
##### ✔ Perfect for sharing across servers

## **Evolution #3 (2010s): Rise of Mobile apps → Token-based auth**

Mobile apps cannot easily store cookies and session state.
So the industry created:
- OAuth2 access tokens
- JWT tokens
JWT = portable, stateless authentication.

Pros:
- Works across API servers without session store
- Good for mobile & cross-domain APIs
- Easy for microservices
Cons:
- Cannot be revoked easily
- Security is trickier
- Token leaks are disastrous
So JWT became popular but **not a replacement for session cookies in web apps**.

## Because I want it to also be a mobile app 
### **Hybrid Auth Model (Best of both worlds)**
##### 🔹 **For browsers (React)** → SESSION-BASED AUTH
Stored in HttpOnly cookies:
- Safe
- Simple
- Auto-handled by browser
- Revocable
##### 🔹 **For mobile apps** → JWT (access + refresh tokens)
- Works without cookies
- Easy to store in secure storage
- Designed for API use
- Can be used offline briefly
##### 🔹 **Backend manages BOTH with a unified session store**
Typically Redis:
- Sessions for web → `session_id → user_id`
- Refresh tokens for mobile → `token_id → user_id`

## Password Hashing 
### What algorithms are used in industry?
Today, best practice is:
##### 🔥 **Argon2** (most recommended)
Modern, memory-hard, slow algorithm resistant to GPU attacks.
##### ⭐ **PBKDF2** (still widely used)
Used by Django, AWS Cognito, and many enterprise apps.
##### ⭐ **bcrypt** (older but still acceptable)
Better than plaintext/SHA but aging.
**DO NOT USE:**
❌ MD5  
❌ SHA1  
❌ SHA256  
❌ Any plain cryptographic hash  
❌ "hashlib.sha256(password)"

These are not password hashing algorithms and are extremely unsafe.
I was thinking of having 2 password hash functions and then maybe use the other onw when the users complain, but it is not that easy because you have to deal with the rehash where every user's hash has to be re ran.