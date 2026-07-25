
# Fail2ban — Blocking SSH Brute-Force Attempts

## What I did
For this project I set up Fail2ban on my Ubuntu Server VM to automatically detect and block repeated failed SSH login attempts, then tested it myself from my Kali VM to simulate a brute force attack and prove it actually works.

- **Target VM:** Ubuntu Server, running Fail2ban.
- **Attacker VM:** Kali Linux.
- **Network:** Both VMs on the same private NAT network.

## Step 1
i began by installing fail2ban on the target Ubuntu VM.

I edited the "[sshd]" section to look like this:

![SSHD edit](photo-sshconfirmation)

- enabled = true — turns on protection for SSH specifically
- maxretry = 3 — allow 3 failed login attempts before taking action
- bantime = 600 — once banned, blocks that IP for 600 seconds 

## Step 2: Confirming it was running, before any attempts

![show fail2ban was working](photo-banempty)

This confirmed the jail was up and working and rightly hadn't detected any attempts to login yet.

## Step 3: Simulating a brute-force attempt
From Kali, I deliberately tried to SSH into the target using a fake username and wrong passwords, three times in a row.

![failed attempts](photo-2failedattempts)

Each of the first 3 attempts failed normally with `Permission denied`, since the ban limit hadn't been reached yet.

## Step 4: Triggering the ban
On the 4th attempt, instead of even being asked for a password, I got "connection refused".

![connection refused](photo-connectionrefused)

This showed my Kali machine had been blocked before it could try again.

## Step 5: Confirming the ban
Back on the Ubuntu VM, I checked Fail2ban's status again:

![ban proof](photo-banproof)

This confirmed Fail2ban had detected the 3 failed attempts and blocked my Kali VM's IP address, exactly as configured.

## Why this matters
In a real situation, Fail2ban would help prevent brute-force attacks and blocks to user out from the server. Although brute-force attacks are less common today, this is still an essential part of sercurity.

## What I learned
- What a brute-force attack is and why SSH is a common target
- How to configure a real security tool's settings
- How to actually test and prove a security control works
