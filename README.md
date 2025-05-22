# 🔐 JWT Authentication Bypass via Weak Signing Key for Bug Bounty

## Exploiting weak JWT secrets to forge admin access and delete users in a simulated vulnerable web app.

---

## ✨ Introduction

JSON Web Tokens (JWTs) are a common mechanism for managing sessions and user identity in web applications. However, when implemented with poor security practices — such as using a weak signing key — they open the door to severe privilege escalation attacks. This article demonstrates how I successfully bypassed JWT authentication in a PortSwigger lab by brute-forcing a weak signing key and forging a valid administrator token.

![Cover](https://github.com/user-attachments/assets/cda52296-ad13-45bc-9c1d-1f623398c6a0) <br/>

---

## 🧠 Understanding the Vulnerability

JSON Web Tokens (JWTs) are a widely adopted method for securely transmitting information between parties. The integrity of JWTs is ensured using a cryptographic signature, typically with a secret key. But what happens when that key is weak?

In this lab from PortSwigger, the web application uses an extremely weak symmetric key (`secret1`) to sign JWTs. If an attacker can brute-force this key, they can forge JWTs, escalate privileges, and impersonate users — even administrators.

This article will walk you through the exact steps I took to exploit the weak key and bypass authentication to gain admin access. Perfect for bug bounty hunters and ethical hackers looking to sharpen their JWT exploitation skills.

---

## 💥 Vulnerable Lab Scenario

* **Lab Name**: JWT authentication bypass via weak signing key
* **Environment**: PortSwigger Academy
* **Goal**: Brute-force the JWT secret, forge a token with admin privileges, and delete user `carlos`.

---

## 🔧 Tools Required

* **Burp Suite** (with **JWT Editor** extension)
* **Hashcat** (for brute-forcing the secret)
* **jwt-secrets list** by Wallarm

---

## ✅ Step-by-Step Exploit Walkthrough

> ✨ **Background:** Ensure the JWT Editor extension is installed in Burp before starting.

![JWTEditor](https://github.com/user-attachments/assets/3d63efda-fad3-4e1d-8fc5-589d1fd8cb0d) <br/>

### 1. Access the Lab

Visit: [Lab URL](https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-weak-signing-key) and log in using:
`wiener:peter`

![1](https://github.com/user-attachments/assets/a1a839a3-6abb-4cb0-8a4b-8094daa8bad7) <br/>

### 2. Try accessing `/admin`

You'll get a **403 Forbidden** or redirected — access denied. As expected.

![2](https://github.com/user-attachments/assets/a1e462ea-1a85-4156-83db-f3059aa93a04) <br/>

### 3. Capture `/admin` in Burp and send to Repeater

Inspect the JWT in the `Authorization` header. This is the token we'll forge soon.

![3](https://github.com/user-attachments/assets/c1710873-9767-474b-bccf-30b0e079fe9e) <br/>

### 4. Brute-force the JWT secret

Clone the jwt-secrets list:

```bash
git clone https://github.com/wallarm/jwt-secrets
```

Run Hashcat:

```bash
hashcat -a 0 -m 16500 eyJraWQiOiI5ODAxZmViMC1mOWExLTQzMzYtOTc2Yy1kNjBiYjI5ZTgwMzciLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJwb3J0c3dpZ2dlciIsImV4cCI6MTc0NzkyNjA0OCwic3ViIjoid2llbmVyIn0.s2JicDiLkrWtnSRMxzIJN5PWZaLKrTGZD5PEJbemhsU jwt-secrets/jwt.secrets.list
```

We get:

```
secret1
```

![4](https://github.com/user-attachments/assets/0268dac1-5c3a-4a01-8646-02b91a5ce42d) <br/>

> 💡 Tip: If you run the command multiple times, use `--show` to display cracked secrets.

### 5. Base64 encode the secret using Burp Decoder

Encode `secret1` →
**Result:** `c2VjcmV0MQ==`

![5](https://github.com/user-attachments/assets/b7f798fe-89c4-41d7-bbe3-eae13bc52c08) <br/>

### 6. Generate a symmetric signing key in Burp

* Go to **JWT Editor → Keys → New Symmetric Key**
* Click **Generate**
* Replace the `k` parameter with `c2VjcmV0MQ==`
* Save the key

![6](https://github.com/user-attachments/assets/0dff856e-550d-425e-9267-d9b781d052d3) <br/>

### 7. Modify the JWT and Sign

* In **Repeater**, switch to the **JSON Web Token** tab
* Change the `sub` field from `wiener` to `administrator`
* Click **Sign** → Select the symmetric key created earlier
* Ensure **“Don't modify header”** is checked
* Click **OK**

![7](https://github.com/user-attachments/assets/24f92d6b-a1f0-4376-95e6-121c349e74d3) <br/>

### 8. Send the modified request

Boom. You're now the **administrator**. `/admin` is accessible.

![8](https://github.com/user-attachments/assets/2e133cb3-7f9d-4313-8c76-54c2090afed6) <br/>

### 9. Delete `carlos`

Update the request URL to:

```
/admin/delete?username=carlos
```

Send the request.

![9](https://github.com/user-attachments/assets/4472010f-f73d-400b-a3b4-67642af98e7f) <br/>

### 10. Confirm Lab Completion

Right-click → **Show in Browser** → Copy and paste the URL to confirm.

![10](https://github.com/user-attachments/assets/45a5c26e-715e-410e-bf46-92ebd71223b1) <br/>

🎉 **Lab Solved!**

---

## 💡 Key Takeaways

* Always verify the strength of your JWT signing secrets.
* Avoid hardcoded or weak secrets like `secret1`.
* When working with symmetric signing (e.g., HMAC), brute-forcing becomes feasible if the key is weak and predictable.
* Tools like Hashcat combined with Burp Suite make JWT tampering incredibly effective in vulnerable systems.

---

## 📌 Real-World Relevance

In real-world bug bounty programs, JWT-related vulnerabilities are common but often overlooked. Weak keys, algorithm confusion, or missing validations can lead to full account takeovers. Always inspect JWT headers, verify the `alg` used, and try decoding and tampering under safe conditions.

---

## 🙌 Happy Hacking!

And that’s a wrap on another juicy JWT bypass!

Keep exploring, keep hacking — and as always:

> **Stay curious. Stay ethical. Stay legendary.**
>
> — *Aditya Bhatt 🧠💻 | VAPT Specialist | TryHackMe Top 2%*

---
