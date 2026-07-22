
# SQL Injection Lab

## What I did
I built a small home lab using two virtual machines — a Kali Linux "attacker" machine and an Ubuntu Server "target" machine running DVWA (Damn Vulnerable Web Application). I used these VMs to learn how to do a SQl injection and a a UNION-based SQL Injection, obtain hashed passwords, and then used 'John the Ripper' to figure out the password from the hash.

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
**1' ORDER BY 1-- - 
1' ORDER BY 2-- -
1' ORDER BY 3-- -**

It errored at 3, which means the query only uses 2 columns.

## Step 4: Extracting real credentials
Using that information, I ran **1' UNION SELECT user, password FROM users-- -**

This combined the original query with a new one, the result revealed real usernames and password hashes, including
**admin:5f4dcc3b5aa765d61d8327deb882cf99**

## Step 5: Cracking the password hash
The password wasn't stored as plain text, it was hashed. I used John the Ripper on Kali to crack it:

echo '5f4dcc3b5aa765d61d8327deb882cf99' > hash.txt
john --format=raw-md5 hash.txt
john --show --format=raw-md5 hash.txt

![john the ripper](photo-jacktheripper)

John cracked it and revealed the password as **"password"**.

## Why this happened
The website took what I typed into the search box and entered it directly into a database command, without checking it first. This let me change what the query actually did, instead of just searching for a normal ID.

## How this would be fixed
Use **parameterised queries**, which keep user input strictly separate from the actual database command, so it can't be treated as part of the query itself.

## What I learned
- How a SQL injection works
- What password hashing is
- How to use John the Ripper to crack a hash
- The importance of using paramaterised queries 
