---
layout: post
title: "Essential Web Server Security: Hiding Versions and Disabling Directory Listing"
date: 2026-02-09 10:00:00 +0900
categories: [Security, Web]
tags: [Nginx, Apache, Security, WebServer]
---

# Stop the 'TMI': Essential Security Hardening for Nginx & Apache

As a developer managing various projects, it is just as important to secure your infrastructure as it is to write clean code. Attackers often look for small clues—like server versions or exposed file lists—to find a way in. Today, we’ll cover two fundamental yet powerful defensive measures: **Disabling Directory Listing** and **Hiding Server Tokens**.

---

## 1. Disabling Directory Listing

**"Don't post your house floor plan on the front door."**

* **The Issue**: If a specific folder is accessed and an `index.html` file is missing, the server "helpfully" displays a full list of every file in that directory to the browser.
* **The Risk**: This can expose backup files (`.zip`, `.bak`), configuration files, or sensitive logs left over from development, providing a roadmap for data theft.

### 🛠️ How to Fix It

| Server | Config File Path | Core Configuration |
| :--- | :--- | :--- |
| **Nginx** | `/etc/nginx/sites-available/default` | `autoindex off;` |
| **Apache** | `/etc/apache2/apache2.conf` | `Options -Indexes` |

> **Nginx Example:**
> ```nginx
> location / {
>     autoindex off; # Explicitly denies directory listing.
> }
> ```

---

## 2. Hiding Server Tokens (Version Information)

**"Don't tell a thief exactly which model of door lock you use."**

* **The Issue**: By default, the HTTP response header (`Server`) often includes not just the server name, but the **exact version**, such as `Nginx/1.18.0 (Ubuntu)`.
* **The Risk**: If a known vulnerability (CVE) exists for that specific version, an attacker can immediately deploy automated tools to exploit it.

### 🛠️ How to Fix It

| Server | Config File Path | Core Configuration |
| :--- | :--- | :--- |
| **Nginx** | `/etc/nginx/nginx.conf` | `server_tokens off;` |
| **Apache** | `/etc/apache2/conf-available/security.conf` | `ServerTokens Prod` |

> **Apache Example:**
> ```apache
> # Hides detailed versioning and removes the server signature from error pages.
> ServerTokens Prod
> ServerSignature Off
> ```

---

## 📊 Final Summary & Checklist

1.  **Directory Listing**: Block file structure exposure → Verify with a `403 Forbidden` response.
2.  **Server Tokens**: Hide detailed versions → Verify that only `Server: nginx` or `Server: Apache` is visible via `curl`.

---

## 💡 Closing Thoughts

From the perspective of security certifications like **ISMS-P**, these settings are the absolute basics of "Information System Access Control". Adding just a single line of configuration can make an attacker think, *"This server is well-maintained; it's going to be a hassle to breach."*

Consistent with the values of the **Boksepansal** brand, even complex security becomes manageable when you tackle it one step at a time.