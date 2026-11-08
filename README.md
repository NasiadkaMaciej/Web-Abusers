# 🛡️ Fail2ban IP Blocklist

This repository contains a frequently updated list of malicious IP addresses caught by my personal `fail2ban` instance. These IPs were detected while attempting various web attacks, vulnerability scans, or brute-force logins.

## 📦 What's inside?

`bots.txt` is a sorted, deduplicated list of addresses currently banned on my server, regenerated daily. It merges the bans of several jails:

  - **Path scanners (`nginx-bots`):** probes for `.env`, `.git` and other hidden files, `phpinfo`, phpMyAdmin, admin panels of software I don't run, `/cgi-bin/`, `.asp`/`.jsp`, web shells, traversals, exploit payloads, `/mcp`, `/sse`, webmail, Exchange/RDP, etc.
  - **WordPress scanners (`nginx-bots-wp`):** `/wp-login`, `/wp-admin`, `/xmlrpc`, etc.
  - **Bad hosts (`nginx-badhosts`):** requests with a `Host` header that is not one of my domains: a bare IP address, an invalid host, proxy/SSRF testers.
  - **Abuse (`nginx-403`, `nginx-ratelimit`, `sshd`):** repeated 401/403 responses, hammering the rate limit, SSH brute-force.

Research scanners that probe unsolicited (Censys and the like) end up on the list too. Legitimate crawlers (Googlebot, Bingbot, ...) are not supposed to.

## ⚠️ Important Considerations

If you plan to use this list in your own setup, please keep a few things in mind:

1. **Dynamic IPs & False Positives**
   IPv4 addresses change hands frequently. An IP that's part of a botnet today might belong to a regular, legitimate user tomorrow. Because of this, **permanent bans are not recommended.** It's much safer to use temporary blocks that expire automatically. In my setup, the base ban duration is **1 month** (shorter for the 403 and rate-limit jails).

2. **Progressive Banning for Repeat Offenders**
   To effectively handle aggressive, returning bots without risking permanent blocks, my server utilizes fail2ban's native progressive banning (`bantime.increment`). If an IP from this list serves its initial ban and attacks again, its penalty is automatically multiplied (e.g., 2 months, 4 months, up to a maximum of 1 year).

3. **Git Repository Size**
   Since this list is updated frequently via an automated script, the commit history will grow quite a bit over time. If you only want the data, it's better to just fetch the raw `.txt` file directly rather than cloning the whole repo:
   ```bash
   curl -sO https://raw.githubusercontent.com/NasiadkaMaciej/Web-Abusers/master/bots.txt
   ```

4. **GeoIP Blocking (Missing RU/CN/SG IPs)**
   You might notice that there are very few or no Russian (RU), Chinese (CN) and Singaporean (SG) IP addresses in this list. This is because these countries are blocked entirely at the firewall level using GeoIP filtering on my server. As a result, traffic from these countries never reaches the web server and is not processed by `fail2ban`.

## 🚀 How to Use

If you'd like to quickly inject the list into your own `fail2ban` setup, you can loop through the lines and use the `fail2ban-client` to ban the IP addresses.

For example, to import the `bots.txt` file into your local `nginx-bots` jail, run:

```bash
for ip in $(cat bots.txt); do sudo fail2ban-client set nginx-bots banip $ip; done
```
*Note: Make sure to replace `nginx-bots` with the actual name of the jail you configured on your server. `banip` also bans addresses from your `ignoreip` list, so remove your own first.*
