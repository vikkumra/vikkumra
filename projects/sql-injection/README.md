
# SQL Injection Lab

## What I did
I built a small home lab using two virtual machines — a Kali Linux "attacker" machine and an Ubuntu Server "target" machine running DVWA (Damn Vulnerable Web Application). I used these VMs to learn how to do a SQl injection and a a UNION-based SQL Injection, obtain hashed passwords, and then used 'Jack the Ripper' to figure out the password from the hash.

- **Attacker VM:** Kali Linux
- **Target VM:** Ubuntu Server running Apache, PHP, MySQL, and DVWA
- **Network:** Both VMs on the same private (NAT) network, seperate from my real home network
- **Difficulty setting:** DVWA set to "Low"

## Step 1: Testing for the vulnerability
DVWA's SQL Injection page has a "User ID" search box that looks up a user's name.

Searching **1** returned one normal result, as expected.

![1 Response](photo-1response)

Searching **1'** (adding a quote) caused a **500 Internal Server Error**. This shows the input isn't being handled properly, the extra quote broke the database query behind the scenes in sql.

![500 internal error](photo-servererror)

## Step 2: Bypassing the query's logic
I then tried **1' OR '1'='1**

This returned every user in the database, not just user 1. This works because **'1'='1'** is always true, so the database ended up matching every row instead of just one.

![injection Response](photo-injectionresponse)

## Step 3: Finding the number of columns
Before pulling data from another table, I needed to know how many columns the original query used. I tested:
```
1' ORDER BY 1-- -
1' ORDER BY 2-- -
1' ORDER BY 3-- -
```
It errored at 3, confirming the query only uses **2 columns**.

## Step 4: Extracting real credentials
Using that information, I ran:
```
1' UNION SELECT user, password FROM users-- -
```
This combined the original query with a new one, pulling data straight out of the `users` table — a table the page was never designed to show. The result revealed real usernames and password hashes, including:
```
admin : 5f4dcc3b5aa765d61d8327deb882cf99
```

## Step 5: Cracking the password hash
The password wasn't stored as plain text — it was hashed using MD5, an older hashing algorithm. I used **John the Ripper** on Kali to crack it:
```
echo '5f4dcc3b5aa765d61d8327deb882cf99' > hash.txt
john --format=raw-md5 hash.txt
john --show --format=raw-md5 hash.txt
```
John cracked it almost instantly, revealing the original password: **"password"**.

## Why this happened
The website took whatever I typed into the search box and glued it directly into a database command, without checking it first. This let me change what the query actually did, instead of just searching for a normal ID.

## Why it matters
If this were a real system, this exact technique could let someone:
- Bypass a login without knowing a real password
- Steal usernames and password hashes from a database
- In more serious cases, access or modify other data on the server

The fact that the hash cracked instantly is also a useful finding on its own — even if passwords are hashed, a weak password like "password" makes that protection meaningless.

## How this would be fixed in a real system
- Use **parameterized queries** (also called prepared statements), which keep user input strictly separate from the actual database command, so it can never be treated as part of the query itself
- Validate and filter user input before using it
- Store passwords using a modern, slow hashing algorithm like **bcrypt** or **Argon2**, not MD5
- Enforce stronger password policies so common/weak passwords aren't usable

## What I learned
- How SQL injection actually works, not just how to run the commands
- How to test for a vulnerability step by step, rather than jumping straight to an exploit
- What password hashing is, and why weak hashing algorithms and weak passwords are both separate risks
- How to use John the Ripper to crack a hash
- How to explain a technical vulnerability in plain terms, including its real-world impact and fix
