# Fail2ban IP Blocklist

This repository contains a frequently updated list of malicious IP addresses caught by my `fail2ban` instance. These IPs were detected while attempting various web attacks or scanning for vulnerabilities.

## What's inside?

- `bots.txt`: General-purpose vulnerability scanners, automated exploits, and bots. This catches things like:
  - **Webmail scans:** `/roundcube/`, `/horde/`, `/webmail/`
  - **Exposed secrets:** `.env`, `.aws/`, `.git`, `.ds_store`, `/id_rsa`
  - **Web shells & payloads:** `/shell.php`, `/cmd.php`, `/eval(`, `/base64_decode`
  - **IoT & routers:** `/cgi-bin/`, `/luci/`, `/setup.cgi`
  - **Common traps:** Actuator endpoints, Nacos, Kubernetes namespaces, PHPUnit
  - **Directory traversals:** `/etc/passwd`, `@fs`
  - and many more
- `bots-wp.txt`: IPs specifically attempting to brute-force or enumerate WordPress installations. It matches requests to paths like `/wp-login`, `/wp-admin`, `/wp-includes`, `/xmlrpc`, etc.

## Keep in mind

If you plan to use these lists in your own setup, just a heads-up on a couple of things:

1. **Dynamic IPs & False Positives**
   IPv4 addresses change hands frequently. An IP that's part of a botnet today might belong to a regular user tomorrow. Because of this, it's usually not a good idea to ban these IPs permanently. It's much safer to use them for temporary blocks that expire automatically so you avoid dropping legitimate traffic out of the blue. In my case (and in this repository) IPs are banned for 1 month.
   
2. **Git Repository Size**
   Since these lists are updated frequently via a script, the commit history will grow quite a bit over time. If you're utilizing the data, it's easier to just fetch the raw `.txt` files directly rather than cloning the whole repo.

## How to Import Manually

If you'd like to quickly inject one of these lists into your own `fail2ban` setup, you can loop through the lines and use the `fail2ban-client` to ban the IP addresses.

For example, to import the `bots.txt` file into your local `nginx-bots` jail, run:

```bash
for ip in $(cat bots.txt); do sudo fail2ban-client set nginx-bots banip $ip; done
```
*Note: Make sure to replace `nginx-bots` with the actual name of the jail you configured on your server.*