
# Fail2ban — Blocking SSH Brute-Force Attempts

## What I did
For this project I set up Fail2ban on my Ubuntu Server VM to automatically detect and block repeated failed SSH login attempts, then tested it myself from my Kali VM to simulate a brute force attack and prove it actually works.

- **Target VM:** Ubuntu Server, running Fail2ban.
- **Attacker VM:** Kali Linux.
- **Network:** Both VMs on the same private NAT network.

## Step 1: Installing and configuring Fail2ban

sudo apt install fail2ban -y
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo nano /etc/fail2ban/jail.local

I edited the `[sshd]` section to look like this:
```
[sshd]
enabled = true
maxretry = 3
bantime = 600
```

- **enabled = true** — turns on protection for SSH specifically
- **maxretry = 3** — allow 3 failed login attempts before taking action
- **bantime = 600** — once banned, block that IP for 600 seconds (10 minutes)

Then I restarted the service:
```
sudo systemctl restart fail2ban
sudo systemctl enable fail2ban
```

## Step 2: Confirming it was running, before any attempts
```
sudo fail2ban-client status sshd
```
This showed `Currently failed: 0` and `Currently banned: 0` — confirming the jail was active and watching, but hadn't seen anything yet.

## Step 3: Simulating a brute-force attempt
From Kali, I deliberately tried to SSH into the target using a fake username and wrong passwords, three times in a row:
```
ssh -o StrictHostKeyChecking=no fakeuser@192.168.93.130
```
Each of the first 3 attempts failed normally with `Permission denied`, since the ban limit hadn't been reached yet.

## Step 4: Triggering the ban
On the 4th attempt, instead of even being asked for a password, I got:
```
ssh: connect to host 192.168.93.130 port 22: Connection refused
```
This showed my Kali machine had been blocked entirely before it could try again.

## Step 5: Confirming the ban
Back on the Ubuntu VM, I checked Fail2ban's status again:
```
sudo fail2ban-client status sshd
```
This time it showed:
```
Total failed: 3
Currently banned: 1
Banned IP list: 192.168.93.128
```
This confirmed Fail2ban had correctly detected the 3 failed attempts and blocked my Kali VM's IP address, exactly as configured.

## Why this matters
In a real environment, this kind of protection stops (or seriously slows down) an attacker trying to guess passwords over SSH. Without something like Fail2ban, an attacker could attempt thousands of password combinations uninterrupted. With it in place, they're automatically locked out after a small number of failures, giving them far less chance of success and giving defenders time to notice and respond.

## What I learned
- The difference between attacking a system (previous project) and defending one (this project)
- What a brute-force attack is and why SSH is a common target
- How Fail2ban reads logs, detects patterns, and reacts using the firewall
- How to configure a real security tool's settings, rather than just installing it
- How to actually test and prove a security control works, instead of assuming it does
