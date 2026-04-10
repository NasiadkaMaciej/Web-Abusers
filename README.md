# 🛡️ Fail2ban IP Blocklist

This repository contains a frequently updated list of malicious IP addresses caught by my personal `fail2ban` instance. These IPs were detected while attempting various web attacks, vulnerability scans, or brute-force logins.

## 📦 What's inside?

The repository provides a blocklist (`bots.txt`) targeting general-purpose vulnerability scanners, automated exploits, and malicious bots. This catches things like:
  - **Webmail scans:** `/roundcube/`, `/horde/`, `/webmail/`
  - **Exposed secrets:** `.env`, `.aws/`, `.git`, `.ds_store`, `/id_rsa`
  - **Web shells & payloads:** `/shell.php`, `/cmd.php`, `/eval(`, `/base64_decode`
  - **IoT & routers:** `/cgi-bin/`, `/luci/`, `/setup.cgi`
  - **Common traps:** Actuator endpoints, Nacos, Kubernetes namespaces, PHPUnit
  - **Directory traversals:** `/etc/passwd`, `@fs`
  - **WordPress enumeration & brute-force:** `/wp-login`, `/wp-admin`, `/wp-includes`, `/xmlrpc`, etc.
  - *...and many more.*

## ⚠️ Important Considerations

If you plan to use this list in your own setup, please keep a few things in mind:

1. **Dynamic IPs & False Positives**
   IPv4 addresses change hands frequently. An IP that's part of a botnet today might belong to a regular, legitimate user tomorrow. Because of this, **permanent bans are not recommended.** It's much safer to use temporary blocks that expire automatically. In my setup, the base ban duration is **1 month**.

2. **Progressive Banning for Repeat Offenders**
   To effectively handle aggressive, returning bots without risking permanent blocks, my server utilizes fail2ban's native progressive banning (`bantime.increment`). If an IP from this list serves its initial 1-month ban and attacks again, its penalty is automatically multiplied (e.g., 2 months, 4 months, up to a maximum of 1 year).

3. **Git Repository Size**
   Since this list is updated frequently via an automated script, the commit history will grow quite a bit over time. If you only want the data, it's better to just fetch the raw `.txt` file directly rather than cloning the whole repo.

4. **GeoIP Blocking (Missing RU/CN IPs)**
   You might notice that there are very few or no Russian (RU) and Chinese (CN) IP addresses in this list. This is because these countries are blocked entirely at the firewall level using GeoIP filtering on my server. As a result, traffic from these countries never reaches the web server and is not processed by `fail2ban`.

## 🚀 How to Use

If you'd like to quickly inject one of these lists into your own `fail2ban` setup, you can loop through the lines and use the `fail2ban-client` to ban the IP addresses.

For example, to import the `bots.txt` file into your local `nginx-bots` jail, run:

```bash
for ip in $(cat bots.txt); do sudo fail2ban-client set nginx-bots banip $ip; done
```
*Note: Make sure to replace `nginx-bots` with the actual name of the jail you configured on your server.*