# TryHackMe-TakeOver-WriteUp
Room Link : https://tryhackme.com/room/takeover  
Difficulty: Easy

---

## What You'll Need
- ffuf (or Dirbuster)  
- Firefox browser  
- Basic networking knowledge (subdomains, virtual hosts, certificates)

---

## Introduction
Before jumping into the challenge, let's briefly explain two essential concepts:

### 🔹 Subdomains & Virtual Hosts
A domain like example.com can have multiple virtual hosts such as:
- api.example.com  
- dev.example.com  

When you edit /etc/hosts in a lab environment, you manually map an IP to a domain so the browser knows where to send the request. Because TryHackMe doesn't provide a real DNS setup, virtual hosts must be discovered manually.

### 🔹 Certificates in CTF Environments
In HTTPS, certificates validate the identity of a domain.  
However, CTF machines often use self‑signed certificates, causing Chrome/Edge to block the page.  
Firefox allows bypassing certificate warnings more easily, which is why it becomes essential in this room.

---

## Let's Start
As instructed by TryHackMe, we first map the machine's IP address to the main domain:

```bash
sudo nano /etc/hosts
```

```
target-ip    futurevera.thm
```
---

## Exploring the Main Website
<img width="1100" height="549" alt="image" src="https://github.com/user-attachments/assets/a684a068-cf35-4192-8aea-e89c6e01a896" />

Visiting `https://futurevera.thm/` shows… almost nothing.
This is expected — the room description hints that your goal is to discover subdomains.
Since this machine relies on virtual hosts instead of real DNS records, we must enumerate subdomains manually.

---

## Subdomain Enumeration with ffuf

We use the following command to fuzz the Host header:

```bash
ffuf -w /snap/seclists/current/Discovery/DNS/subdomains-top1million-110000.txt \
     -H "Host: FUZZ.futurevera.thm" \
     -u https://target-ip/ \
     -fs 4605
```
<img width="1100" height="612" alt="image" src="https://github.com/user-attachments/assets/e3f70488-5db7-4753-8a1d-b9875fac920e" />

### 🔍 Why `-fs 4605`?

Because every invalid subdomain returns the same page size (4605 bytes), due to virtual host fallback.
Filtering out those responses allows us to see only the real subdomains.

### ✅ Results

We discover two valid subdomains:

* blog.futurevera.thm
* support.futurevera.thm

Add both to `/etc/hosts`:

```
target-ip blog.futurevera.thm
target-ip support.futurevera.thm
```

---

## Investigating the Subdomains

### 🔹 Blog Subdomain
<img width="1100" height="534" alt="image" src="https://github.com/user-attachments/assets/527f1940-9070-40e2-9fb5-a345d7c67896" />

The blog subdomain loads correctly but contains nothing exploitable.
Nothing interesting in the source code, no hidden directories — dead end.

### 🔹 Support Subdomain
<img width="1100" height="592" alt="image" src="https://github.com/user-attachments/assets/19bfb6a3-a075-4661-ae81-6773b29a579f" />


Accessing `https://support.futurevera.thm/` in Chrome or Edge results in:

```
ERR_SSL_KEY_USAGE_INCOMPATIBLE
```

This happens because the server uses an improperly configured certificate.

### 🦊 Solution: Use Firefox
<img width="1100" height="537" alt="image" src="https://github.com/user-attachments/assets/0dff1c2a-bb22-4699-b33d-46b647b536e7" />


Firefox allows bypassing this kind of SSL misconfiguration.
Once you open the support subdomain with Firefox and accept the certificate risk, the page loads correctly.

---

## The Hidden Clue

On the Support page, check the certificate details.
<img width="1904" height="916" alt="image" src="https://github.com/user-attachments/assets/95fbbf82-a6d0-47a0-8776-09b81c583383" />

Inside the certificate, we find an additional domain name listed under the **Subject Alternative Name (SAN)** field.

Add that new hostname to your `/etc/hosts` file:

```
target-ip  secrethelpdesk934752.support.futurevera.thm
```

Then access it:

```
https://secrethelpdesk934752.support.futurevera.thm
```

💥 This is where the final flag is located.
<img width="993" height="370" alt="image" src="https://github.com/user-attachments/assets/f29fd0e9-468b-4b6f-8076-a599334835b2" />


### 🔎 What Is the SAN Field and Why It Matters?

In HTTPS certificates, the Subject Alternative Name (SAN) field lists all domains and subdomains for which the certificate is valid.
This is extremely useful in CTF environments, because SAN entries can reveal hidden subdomains that are not meant to be publicly accessible.

So even if you're browsing `support.futurevera.thm`, the certificate may include something like:

```
DNS Name: secrethelpesk934752.support.futurevera.thm
```

This means the server is configured to handle that hidden subdomain as well.
In this room, the SAN field exposes the exact domain we need to continue the challenge.

---

## Solution Summary

* Add `futurevera.thm` to `/etc/hosts`
* Use ffuf with Host-header fuzzing to discover VHOST subdomains
* Identify blog and support subdomains
* Use Firefox to access the misconfigured support certificate
* Inspect certificate → extract hidden SAN domain
* Add it into `/etc/hosts`
* Visit the hidden domain → retrieve the flag 🎉

---

## Conclusion

This room is a great introduction to:

* Subdomain enumeration via Host-header fuzzing
* Understanding virtual hosts
* How SSL certificates can leak information
* Why Firefox is valuable in pentesting environments
* The importance of checking certificate SAN fields

A simple challenge — but packed with real‑world techniques used in reconnaissance and web exploitation.

Happy hacking!
Hope you enjoyed walking through this room with me!



