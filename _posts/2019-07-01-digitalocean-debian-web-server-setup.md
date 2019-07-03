---
layout: post
title: "How to set up a Debian 9 server, firewall with DigitalOcean from a Windows machine and connect to your domain"
description: "This is a complete step by step guide on how to set up your server from scratch with DigitalOcean, SFTP, and firewalls, from a Windows machine."
categories: tech
---

This is a complete step by step guide on how to set up your server from scratch with DigitalOcean, SFTP, and firewalls, from a Windows machine. I'm writing this from a relative beginner's perspective so that even if you have no experience you'll be able to follow along.

* TOC
{:toc}

## What you'll need

So, you've built your website, bought a domain, and want to serve it up so people can actually go to it! (Or, you're thinking of doing any of the above.) In my case, I had my site files, and just wanted to migrate from Bluehost to something I had much more control over - being able to fully access the server environment and install packages as I please.

In order, here are the items I needed in order to make that come true, from a Windows machine.

Big thanks to my friend [Tim](https://www.thetimmaeh.com){:target="_blank"} for helping me out throughout the process!

### Services

* **DigitalOcean** account - [my signup link for $100 off your first 2 months](https://m.do.co/c/d76a00a28bb4){:target="_blank"}
* A domain - get it at your domain registrar of choice - I use [**NameSilo**](https://www.namesilo.com/register.php?rid=0255158yn){:target="_blank"} after trying both GoDaddy and Bluehost, NameSilo is my fave for its no-nonsense
* [**CloudFlare**](https://www.cloudflare.com/){:target="_blank"} account

### Software

* [**PuTTY**](https://www.putty.org/){:target="_blank"} - to generate SSH keys and connect to the server using PuTTY terminal
  * Select "Package files" - you’ll want everything like **Pageant** and **PuTTYgen**
* [**WinSCP**](https://winscp.net/eng/download.php){:target="_blank"} - What you'll be using for easy upload/download of files to your server

### Useful links

1. [DigitalOcean docs - how to connect with SSH](https://www.digitalocean.com/docs/droplets/how-to/connect-with-ssh/){:target="_blank"}
2. [DigitalOcean docs - initial server setup with Debian 9](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-debian-9){:target="_blank"}
3. [DigitalOcean docs - set up a firewall with Debian 9](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-debian-9){:target="_blank"}

### Good to know

* Basic Linux commands like `cd, rm, cp, mv, touch, nano`

## Connect to DigitalOcean with SSH

1. Use PuTTYgen to generate keys (add password to the key), save them in a folder and remember its location.
2.  Create Droplet on DO dashboard
   1. Debian 9
   2. SSH (NOT one-time password)
   3. Upload the public key you generated with PuTTYgen to what it requests
1. WinSCP - Tool > Pageant on login dialog box
   1. Add Key - select your private key (usually .ppk) - use the password you set (that it asks for)
   2. Pageant is running on the task bar icon)
   3. Pageant is so don’t have to set private key for ever program that connects to your server
1. Use SSH & SFTP into server using WinSCP, set the IP and user: `root` 
   1. Don’t need to enter anything else, WinSCP will check if Pageant (or equivalent software) is running and uses the private key info they provide
   2. IP: the DO control panel
1. Back to PuTTY
   1. Session: IP address, connection type: SSH
   2. SSH-Auth - add private key location (optional: Pageant will do this automatically if running)
   3. Back to session - put in a name like `myserver`, save
   4. OPTIONAL: using Pageant to leave out privatekey/passphrase info in Atom, for example (Remote-FTP package in Atom)

## Server setup with Debian 9, nginx server

Fun fact: nginx is pronounced engine-x

1. Following [DigitalOcean docs - initial server setup with Debian 9](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-debian-9){:target="_blank"}
   1. Step One - Logging in as Root
   2. Step Two - Creating a New User
   3. Step Three - Granting Administrative Privileges
2. go to `/root/.ssh`

3. .

```console
mkdir /home/<user>/.ssh
cp authorized_keys /home/<user>/.ssh
chown <user>:<user> -R /home/<user>/.ssh
```

4. .
   1. This series of steps creates a folder within <user>’s home directory and log into <user> with the private key created earlier
   2. Note: user `root`’s root folder is /root, <user>’s root folder is `/home/<user>`
   3. Now should be <hostname>@<user>: (on the command line) TODO: check if this is flipped
4. Good to know commands: `ls`, `ls -a` (shows the files beginning with . that is ignored by just `ls`), `<command> --help`, `man <command>` (not installed yet)
5. Now close the window to exit the session
6. Connect again with PuTTY - it will ask for username again
7. Instead of `root`, put in `<user>`
8. Checking if <user> has `sudo` permissions i.e. is a sudoer (admins)
9. `sudo nano /etc/ssh/sshd_config` - enter the password
10. Nano: ctrl+x, y, enter - closes and saves 
11. Line: `PermitRootLogin`, if commented with leading #, uncomment it and make sure it says “no” - `PermitRootLogin no`
    1. this is for security - when done can’t log in with `root` but can do everything else with `su`, `sudo`
    2. if someone gets ahold of your debian password and ssh key password, they might get into the server, but don’t have root privileges and can’t `sudo`
    3. Also they won’t know what your <username> is unless it’s saved in PuTTY session etc. (so discourage from saving it there, type it again every time)

## Set up a firewall with ufw

1. Back to [Step Four - Setting Up a Basic Firewall](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-debian-9#step-four-%E2%80%94-setting-up-a-basic-firewall){:target="_blank"}
    * Now use `sudo` in front of every command
    * Skip Step Five
    * (Optional) Step Six
    * By the way... `ufw` stands for uncomplicated firewall
2. Adding CloudFlare
3. `touch update-ufw-cloudflare.sh` - creates a file called `update-ufw-cloudflare.sh` (can be used for creating files in general)
4. Right click or Shift+Ins can be used to paste

```sh
#!/bin/bash

function rule(){
  echo "allow from $1 to any port $2"
}

curl -s https://www.cloudflare.com/ips-v4 > allowed
echo >> allowed
curl -s https://www.cloudflare.com/ips-v6 >> allowed

yes | /usr/sbin/ufw reset

/usr/sbin/ufw default deny incoming
/usr/sbin/ufw default allow outgoing
# /usr/sbin/ufw allow from 127.0.0.1 to 127.0.0.1 port 2095 proto tcp
/usr/sbin/ufw limit ssh
/usr/sbin/ufw limit sftp

while read ip
do
  echo Allowing incoming port 443 from $ip
  /usr/sbin/ufw $(rule $ip 443)
done < allowed
rm allowed

yes | /usr/sbin/ufw enable
```

## Connect your domain, set up CloudFlare

1. Sign into CloudFlare - add your domain
2. Go to your domain provider - e.g. GoDaddy, NameSilo 
3. Go to control panel for your domain - DNS settings
4. Point them [according to CloudFlare's instructions](https://support.cloudflare.com/hc/en-us/articles/205195708-Changing-your-domain-nameservers-to-Cloudflare){:target="_blank"}
   1. This step might take a while while domain registrar and CloudFlare update your records. Proceed with the next steps in the time being
1. Back to [DigitalOcean docs](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-debian-9){:target="_blank"}
2. Make sure you do `sudo systemctl enable nginx` (at the end leave it on the enabled state)
3. Go to DO control panel and enable Floating IP, copy it
   1. Go to CloudFlare and go to the DNS settings
   2. A record: floating IP address
   3. CNAME: www (this enables www.<your_website>.com to also go to <your_website>.com)
   4. CloudFlare - Crypto tab - SSL setting > Full (strict)
   5. CloudFlare - Crypto tab - create Origin Certificates - default settings - generates private key and CSR (certificate signing request) - save these! (we get Origin Certificate, Private key, instructions at the bottom for Origin CA certificate) - don’t close this page yet!
1. Back to PuTTY terminal

```console
cd /etc/nginx
mkdir certs`
cd certs
mkdir <your_website>.com
```

9. .
    1. Folder will look like `etc/nginx/certs/<your_website>.com`
    2. Save the CloudFlare certificates as `origin-certificate.pem`, `origin-certificate.key` (or whatever name makes it clear to you what they are)
    3. Back to `certs` directory
    4. `wget https://support.cloudflare.com/hc/en-us/article_attachments/201243967/origin-pull-ca.pem`
10. Go to `/etc/nginx/sites-available` and edit the .conf file - remember to substitute your own domain name

```sh
server {
  listen 443;
  listen [::]:443;

  server_name <your_website>.com;
  root /var/www/<your_website>.com;

  index index.php index.html index.htm;

  # SSL
  ssl on;
  ssl_certificate /etc/nginx/certs/<your_website>.com/origin-certificate.pem;
  ssl_certificate_key /etc/nginx/certs/<your_website>.com/origin-certificate.key;
  ssl_client_certificate /etc/nginx/certs/origin-pull-ca.pem;
  ssl_verify_client on;

  # CloudFlare
  include /etc/nginx/cloudflare.conf;
```

   1. (keep the stuff at the end of the file that you had before)
   2. What is changed: listen to port 443 instead of 80 (https instead of http)
   3. `# SSL and # CloudFlare` parts are added (make sure to change the domain to your own)
   4. Save the .conf file, create new file `/etc/nginx/cloudflare.conf`

        ```sh
        # CloudFlare
        # https://www.cloudflare.com/ips-v4
        # https://www.cloudflare.com/ips-v6

        set_real_ip_from 103.21.244.0/22;
        set_real_ip_from 103.22.200.0/22;
        set_real_ip_from 103.31.4.0/22;
        set_real_ip_from 104.16.0.0/12;
        set_real_ip_from 108.162.192.0/18;
        set_real_ip_from 131.0.72.0/22;
        set_real_ip_from 141.101.64.0/18;
        set_real_ip_from 162.158.0.0/15;
        set_real_ip_from 172.64.0.0/13;
        set_real_ip_from 173.245.48.0/20;
        set_real_ip_from 188.114.96.0/20;
        set_real_ip_from 190.93.240.0/20;
        set_real_ip_from 197.234.240.0/22;
        set_real_ip_from 198.41.128.0/17;
        set_real_ip_from 2400:cb00::/32;
        set_real_ip_from 2606:4700::/32;
        set_real_ip_from 2803:f800::/32;
        set_real_ip_from 2405:b500::/32;
        set_real_ip_from 2405:8100::/32;
        set_real_ip_from 2c0f:f248::/32;
        set_real_ip_from 2a06:98c0::/29;

        # use any of the following two
        real_ip_header CF-Connecting-IP;
        # real_ip_header X-Forwarded-For;
        ```

## Deploy your site

1. `sudo nginx -t`  - if output has `syntax is ok`, `test is successful` then continue. If not, common mistakes are: not having the correct url in your files (did you change them all to your domain?) or file name typos in the file paths
2. If previous step ok, `sudo service nginx reload`
3. CloudFlare Crypto panel - enable Authenticated Origin Pulls (below Always Use HTTPS, which you should also turn on)
4. `sudo apt-get install curl` (this might not be default installed, is needed for the .sh)
   1. How to check: `dpkg -s curl | grep Status`
1. `chmod +x update-ufw-cloudflare.sh`
2. `./ update-ufw-cloudflare.sh`
3. Upload your website content to `/var/www/<your_website>.com` using WinSCP
4. Troubleshooting: `sudo ufw status` - check if it’s HTTPS
   1. `sudo service nginx reload`, `sudo service nginx restart`
   2. `sudo ufw enable && sudo ufw status numbered` should show the following (after running `update-ufw-cloudflare.sh`

```sh
    To                         Action      From
     --                         ------      ----
[ 1] 22/tcp                     LIMIT IN    Anywhere
[ 2] 115/tcp                    LIMIT IN    Anywhere
[ 3] 443                        ALLOW IN    103.21.244.0/22
[ 4] 443                        ALLOW IN    103.22.200.0/22
[ 5] 443                        ALLOW IN    103.31.4.0/22
[ 6] 443                        ALLOW IN    104.16.0.0/12
[ 7] 443                        ALLOW IN    108.162.192.0/18
[ 8] 443                        ALLOW IN    131.0.72.0/22
[ 9] 443                        ALLOW IN    141.101.64.0/18
[10] 443                        ALLOW IN    162.158.0.0/15
[11] 443                        ALLOW IN    172.64.0.0/13
[12] 443                        ALLOW IN    173.245.48.0/20
[13] 443                        ALLOW IN    188.114.96.0/20
[14] 443                        ALLOW IN    190.93.240.0/20
[15] 443                        ALLOW IN    197.234.240.0/22
[16] 443                        ALLOW IN    198.41.128.0/17
[17] 22/tcp (v6)                LIMIT IN    Anywhere (v6)
[18] 115/tcp (v6)               LIMIT IN    Anywhere (v6)
[19] 443                        ALLOW IN    2400:cb00::/32
[20] 443                        ALLOW IN    2405:8100::/32
[21] 443                        ALLOW IN    2405:b500::/32
[22] 443                        ALLOW IN    2606:4700::/32
[23] 443                        ALLOW IN    2803:f800::/32
[24] 443                        ALLOW IN    2c0f:f248::/32
[25] 443                        ALLOW IN    2a06:98c0::/29
```

1. Final firewall! Now go to [DigitalOcean firewall panel](https://cloud.digitalocean.com/networking/firewalls){:target="_blank"} and apply the following rules:

    <img src="/assets/digitalocean_firewall_settings.png" alt="DigitalOcean firewall settings"/>

2. These settings block everything except those ports on the DO firewall
3. ICMP is a ping
4. DO firewall - select your droplet and apply it
5. (Optional) DO create snapshot of droplet - pretty cheap, can use snapshot to create droplets with this snapshot. Power down the droplet as instructed, then wait for snapshot to be created. Then power up the droplet again when done.
6. Final notes: if you change computer you’ll have to copy the SSH keys to your new machine (if you have them saved locally) the ones that we generated with PuTTYgen and connect with Pageant (I'm not too sure about this one, could possibly generate another key pair and connect again?)

Disclaimer: I do not claim to be an expert and any suggestions here do not guarantee anything regarding the security, stability, and other aspects of your web server! :)
