# Broken Access Control (BAC)

Broken Access Control (BAC) is one of those classic threats—it's been around forever and still sits right at the top of the OWASP Top 10 list. Simply put, BAC lets attackers do things they shouldn’t: think Insecure Direct Object Reference (IDOR), authentication bypass, parameter tampering, privilege escalation, and forced browsing. These are some of the main tricks attackers use to steal user data or take control of other people’s accounts.

---

## Insecure Direct Object Reference (IDOR)

The name says it all... IDOR (Insecure Direct Object Reference) is about getting direct access to something you shouldn't. It’s like being able to change a URL or parameter and suddenly see someone else’s private info—no hacking skills required!

**How it works:**  
When a server just trusts whatever the client sends (without double-checking if that user should really have access), it opens the door for attackers. 

**For example:**  
```
# You’re logged in as user `123` and your profile loads with `GET /user/profile?user_id=123`.
# Now, just change `123` to `456`: `GET /user/profile?user_id=456`.
# If the server doesn’t check permissions, guess what? You can see user 456’s profile—boooom💥, IDOR in action🚨🔫.
```
**Key point:**  
"Servers should never just trust the client! Always make sure to check that the user actually has permission to access or change the data they’re asking for."

---

## Authentication Manipulation

This happens when the server doesn't properly check who you are before letting you access a resource. Here Basically, the attacker finds a way to “skip” the login or trick the system into thinking they’re someone else.

**How it works:**
- The server blindly trusts certain requests or doesn’t verify things properly.
- Attacker can do stuff like:
  - Using default or weak passwords (admin/admin),
  - Changing parameters in requests (role=user → role=admin),
  - Messing with session tokens or cookies to impersonate another user,
  - Exploiting logic flaws in the login process.

**For example:**
```
Imagine a nightclub with a bouncer. Normally, you show your ID to get in.
# Normal way: You show your ID → bouncer checks → you get in if it’s valid.
# Authentication Bypass: The bouncer is lazy or tricked. You flash a fake ID, whisper “ Hey bro! You know what? I’m VIP,” or even just sneak in  a side door. The bouncer doesn’t check properly → B000M💥, you’re inside💀🤖.

- In web terms: the server is the bouncer, your login credentials are the ID, and the attacker finds a way to slip past the check. Once inside, they can do whatever the real VIP (victim) could do.
```
**Key point:**
"Never let users bypass authentication or trust client-side data for identity—always verify who they are before giving access."

---

## Privilege Escalation

Here either.. the names says it all the term "Escalater->Escalation" hinting us, It’s when an attacker gains higher-level permissions than they should have. Basically going from “normal user” -> “admin” or from “guest” -> “root.”
- It has two types:
  - Vertical escalation : jumping up to a higher role (user -> admin).
  - horizontal escalation : staying at the same level but accessing another user’s stuff (user A -> user B).

**How it works:**

- **Attacker starts with some access**  
  Low-right user, stolen session, or a small bug — that’s the foothold.

- **They look for weak spots to move up**  
  Missing server-side checks, editable role/ID fields, modifiable tokens, exposed admin endpoints, leaked creds, bad file perms, SUID/sudo holes, cron jobs reading world-writable files, etc.

- **They tamper / exploit**  
  Modify params, headers, JSON body, JWT claims, cookies, or files — or use a local/system bug — to change their effective permissions.

- **Backend must re-check**  
  If the backend doesn’t re-verify permissions or validate token signatures, it accepts the upgraded request and grants higher rights.

- **Usually a chain**  
  IDOR or auth-bypass → read creds/config → reuse creds or call admin API → get admin/root.

- **After escalation**  
  Attacker often persists (backdoor, new admin user, add SSH key) and pivots to other targets.

**Examples:**

**Mini example 1 (web):**
```http
# attacker modifies API body
POST /api/user/update
{ "userId": 42, "role": "admin" }
# if server never checks, attacker becomes admin
```
**Mini example 2 (system-level):**
```
# Attacker has a normal user account on the server
# They find a root-owned script that runs automatically via cron
# (cron = a scheduler that runs scripts/commands at set times)
# The script is world-writable (any user can edit it)
# Attacker edits the script to add a reverse shell or payload
# When cron runs the script as root, the payload runs with root privileges
# Now the attacker has full control (root shell) over the server

 - See it’s like the “elevator to the top floor” is already there, but you just need to put yourself in it — cron runs the script as root, and you sneak in your code.
```
**Key point:**

"Privilege escalation = start small, find the weak link, tamper/exploit it, and climb the privileges — backend must always verify permissions to stop it."