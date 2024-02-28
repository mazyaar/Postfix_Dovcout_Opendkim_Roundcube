
# Postfix + Dovcout + Opendkim + Roundcube 
***
>_Introduction
Postfix is a mail transfer agent (MTA), an application used to send and receive email. It can be configured so that it can be used to send emails by local application only. This is useful in situations when you need to regularly send email notifications from your apps or have a lot of outbound traffic that a third-party email service provider won’t allow. It’s also a lighter alternative to running a full-blown SMTP server, while retaining the required functionality._

***In this tutorial, you’ll install and configure Postfix as a send-only SMTP server. You’ll also request free TLS certificates from Let’s Encrypt for your domain and encrypt the outbound emails using them.***

***


## 1.first add user and group and set permission to users:
```bash
apt update && apt upgrade -y
```

## Create mail group and user
```bash
sudo mkdir -p /<path to vmail>/vmail/
```
```bash
sudo groupadd -g 5000 vmail
```
```bash
sudo useradd -g vmail -u 5000 vmail -d /<path to vmail>/vmail
```

## Change mails directory owner
```bash
sudo chmod 770 /<path to vmail>/vmail
```
```bash
sudo chown -R vmail:vmail /<path to vmail>/vmail
```

## Check permissions
```bash
ls -ld /<path to vmail>/vmail
```

## Output
```bash
drwxrwx--- 2 vmail vmail 4096 Jun 24 03:21 /<path to vmail>/vmail
```

## Check UID
```
id -u vmail
```
# Output
```
5000
```
## Check GID
```bash
id -g vmail
```

## Output
```
5000
```
***
> Other Configuration for add Users:
***
```bash
adduser mail
```
```bash
adduser info
```
```bash
usermod -aG sudo info
```
```bash
usermod -aG sudo mail
```
```bash
groupadd -g 5000 info
```
```bash
groupadd -g 5000 mail
```
```bash
mkdir -p /home/mail
```
```bash
mkdir -p /home/info
```
```bash
chown -R info:info /home/mail/
```
```bash
chown -R mail:mail /home/info/
```
```bash
chmod 775 /home/info/
```
```bash
chmod 755 /home/mail/
```
```bash
usermod info -s /sbin/nologin
usermod mail -s /sbin/nologin
```

## 2. Install Postfix

***2.1 To install Postfix run the following command:***
```bash
sudo apt install zip unzip rar unrar
```
```bash
sudo apt install pyzor razor arj cabextract lzop nomarch p7zip-full rpm2cpio tnef unzip unrar-free zip bzip2 cpio file gzip pax
```
```bash
sudo apt install postfix postfix-mysql
```
```bash
sudo apt-get install postfix-policyd-spf-python
```
```bash
sudo dpkg-reconfigure postfix
```

# 2.2 configuration:
![Select Internet Site from the menu, then press TAB to select <Ok>, then ENTER](https://assets.digitalocean.com/articles/postfix-16.04/zJuFrgI.png?1)

***The user interface will be displayed. On each screen, select the following values:***
-   Internet Site
-   **`mail.example.com`**
-   **`steve`**
-   **`mail.example.com`**,  `localhost.localdomain`,  `localhost`
-   No
-   `127.0.0.0/8 \[::ffff:127.0.0.0\]/104 \[::1\]/128`  **`192.168.0.0/24`**
-   0
-     +
-   all

## 2.3 To configure the mailbox format for Maildir: ( Note same path to dovecot mail mail configuration)
```bash
sudo postconf -e 'home_mailbox = /Maildir/'
```
***
## 2.4 Configure TLS: ( Make One of this options to crate cert)

***1.Generating a Certificate Signing Request (CSR)***
```bash
openssl genrsa -des3 -out server.key 2048
```
```bash
openssl rsa -in server.key -out server.key.insecure
```
```bash
mv server.key server.key.secure
```
```bash
mv server.key.insecure server.key
```
```bash
openssl req -new -key server.key -out server.csr
```

***You can now submit this CSR file to a CA for processing. The CA will use this CSR file and issue the certificate. On the other hand, you can create self-signed certificate using this CSR.***

***

***2.Creating a Self-Signed Certificate***
> ## _Many way to create Certifications for Encrypt Mails:_
```bash
mkdir -p /etc/certs/private
```
```bash
mkdir -p /etc/certs/certs
```
```bash
openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
```
```bash
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/certs/private/postfix-selfsigned.key -out /etc/certs/certs/postfix-selfsigned.crt
```
```bash
sudo cp server.crt /etc/ssl/certs
```
```bash
sudo cp server.key /etc/ssl/private
```
***
***3. Use certbot:***
```bash
apt install certbot
```
```bash
certbot certonly --standalone -d myminio.com --staple-ocsp -m test@yourdomain.io --agree-tos
```
```bash
cp /etc/letsencrypt/live/myminio.com/fullchain.pem /home/user/.minio/certs/public.crt
```
```bash
cp /etc/letsencrypt/live/myminio.com/privkey.pem /home/user/.minio/certs/private.key
```
```bash
sudo chown user:user /home/user/.minio/certs/private.key
```
```bash
sudo chown user:user /home/user/.minio/certs/public.crt
```
```bash
sudo cp server.crt /etc/ssl/certs
```
```bash
sudo cp server.key /etc/ssl/private
```
```bash
sudo chown user:user /home/user/.minio/certs/private.key
```
```bash
sudo chown user:user /home/user/.minio/certs/public.crt
```
***
***4. Acme Commands:***
```bash
sudo acme.sh --issue --standalone -d foo.internal \
        --server https://ca.internal/acme/acme/directory \
        --ca-bundle $(step path)/certs/root_ca.crt \
        --fullchain-file foo.crt \
        --key-file foo.key
```
        
***

***5. Create Wildcards***  
_For wildcard:_

```bash
certbot certonly --manual \
  --preferred-challenges=dns \
  --email marcin@hotmail.com \
  --server https://acme-v02.api.letsencrypt.org/directory \
  --agree-tos \
  --manual-public-ip-logging-ok \
  -d “*.domain.com”
  ```
***
  
***6. Acme another:***
```bash
apt update && apt upgrade -y
```
```bash
apt install curl socat -y
```
```bash
curl https://get.acme.sh | sh
```
```bash
~/.acme.sh/acme.sh --set-default-ca --server letsencrypt
```
```bash
~/.acme.sh/acme.sh --register-account -m <email Address>
```
```bash
~/.acme.sh/acme.sh --issue -d <domain> --standalone
```
```bash
~/.acme.sh/acme.sh --installcert -d <domain> --key-file /root/private.key --fullchain-file /root/cert.crt
```
```bash
sudo chown user:user /home/user/.minio/certs/private.key
```
```bash
sudo chown user:user /home/user/.minio/certs/public.crt
```
***

***7. use cerbot auto generate Certificate:***
```bash
certbot --apache
```
***

***8. create a self-signed PEM file:***
```bash
openssl req -newkey rsa:2048 -new -nodes -x509 -days 3650 -keyout key.pem -out cert.pem
```
***How to create a PEM file from existing certificate files that form a chain:***
> (optional) Remove the password from the Private Key by following the steps listed below:

```bash
openssl rsa -in server.key -out nopassword.key
```
> Note: Enter the pass phrase of the Private Key.
> Combine the private key, public certificate and any 3rd party intermediate certificate files:

```bash
cat nopassword.key > server.pem
cat server.crt >> server.pem
```
> Note: Repeat this step as needed for third-party certificate chain files, bundles, etc:
```bash
cat intermediate.crt >> server.pem
```
## 2.5 Set aliasses for mail 

***4- change reciver mail:***
```bash
vi /etc/aliases
```
Edit like this:
```bash
postmaster: root
root: yourmail@domain.local
```

```bash
sudo newaliases
```

## 2.6 Backup postfix Important files
```bash
cp /etc/postfix/main.cf cp /etc/postfix/main.cf.bk
cp /etc/postfix/master.cf cp /etc/postfix/master.cf.bk
```

## 2.7 main.cf configurations
```bash
nano /etc/postfix/main.cf:
```
***edit main.cf file like this and delete extra configurations:***
- Update this configurations and delete extra configurations:
```bash
smtpd_banner = $myhostname ESMTP $mail_name (Ubuntu)
biff = no
readme_directory = no
append_dot_mydomain = no
compatibility_level = 3.6

# TLS parameters
# SMTPD TLS configuration for inbound connections
smtpd_tls_cert_file=/etc/ssl/certs/cert.crt
smtpd_tls_key_file=/etc/ssl/private/private.key #(it's better that use .crt and .key files)
smtpd_tls_security_level=may
smtp_tls_CApath=/etc/cyberredcert/cert.crt #(it's better that use .crt and .key files)
smtp_tls_security_level=may
smtpd_tls_auth_only = yes
smtpd_tls_loglevel = 1
tls_random_source = dev:/dev/urandom
smtpd_tls_received_header = yes
smtp_tls_session_cache_database = btree:${data_directory}/smtp_scache
smtpd_relay_restrictions = permit_mynetworks permit_sasl_authenticated defer_unauth_destination

#General Configurations:
myhostname = domain.com
alias_maps = hash:/etc/aliases
alias_database = hash:/etc/aliases
myorigin = /etc/mailname
mydestination = domain.com, domain.com, localhost.org, , localhost, cyberblue.pro
relayhost =
mynetworks = 127.0.0.0/8 [::ffff:127.0.0.0]/104 [::1]/128
mailbox_size_limit = 0
recipient_delimiter = +
inet_interfaces = all
inet_protocols = ipv4
home_mailbox= /Maildir/
#home_mailbox= /var/mail/

#TLS parameters
#SMTP TLS configuration for outbound connections
smtp_tls_cert_file=/etc/ssl/certs/cert.crt #(it's better that use .crt and .key files)
smtp_tls_key_file=/etc/ssl/private/private.key #(it's better that use .crt and .key files)
smtp_tls_protocols = !SSLv2, !SSLv3
smtp_tls_auth_only = yes
#Enable Opportunistic TLS
smtp_tls_security_level = may
#displays TLS information in the E-Mail header
smtp_tls_received_header = yes
smtp_tls_session_cache_database = btree:${data_directory}/smtp_scache
smtpd_use_tls = yes
mailbox_transport = lmtp:unix:private/dovecot-lmtp
smtputf8_enable = no
#Enforce TLSv1.3 or TLSv1.2
smtpd_tls_mandatory_protocols = !SSLv2, !SSLv3, !TLSv1, !TLSv1.1
smtpd_tls_protocols = !SSLv2, !SSLv3, !TLSv1, !TLSv1.1
smtp_tls_mandatory_protocols = !SSLv2, !SSLv3, !TLSv1, !TLSv1.1
smtp_tls_protocols = !SSLv2, !SSLv3, !TLSv1, !TLSv1.1
```

## 2.8 master.cf configurations: 
```bash
nano /etc/postfix/master.cf:
```
- Update this configurations:
```bash
#
# Postfix master process configuration file.  For details on the format
# of the file, see the master(5) manual page (command: "man 5 master" or
# on-line: http://www.postfix.org/master.5.html).
#
# Do not forget to execute "postfix reload" after editing this file.
#
# ==========================================================================
# service type  private unpriv  chroot  wakeup  maxproc command + args
#               (yes)   (yes)   (no)    (never) (100)
# ==========================================================================
smtp      inet  n       -       y       -       -       smtpd
#smtp      inet  n       -       y       -       1       postscreen
#smtpd     pass  -       -       y       -       -       smtpd
#dnsblog   unix  -       -       y       -       0       dnsblog
#tlsproxy  unix  -       -       y       -       0       tlsproxy
# Choose one: enable submission for loopback clients only, or for any client.
#127.0.0.1:submission inet n -   y       -       -       smtpd
submission inet n       -       y       -       -       smtpd
  -o syslog_name=postfix/submission
  -o smtpd_tls_security_level=encrypt
  -o smtpd_tls_wrappermode=no
  -o smtpd_sasl_auth_enable=yes
#  -o smtpd_tls_auth_only=yes
#  -o smtpd_reject_unlisted_recipient=no
#  -o smtpd_client_restrictions=$mua_client_restrictions
#  -o smtpd_helo_restrictions=$mua_helo_restrictions
#  -o smtpd_sender_restrictions=$mua_sender_restrictions
  -o smtpd_recipient_restrictions=permit_mynetworks,permit_sasl_authenticated,reject
  -o smtpd_relay_restrictions=permit_sasl_authenticated,reject
  -o smtpd_sasl_type=dovecot
  -o smtpd_sasl_path=private/auth
#  -o milter_macro_daemon_name=ORIGINATING
# Choose one: enable smtps for loopback clients only, or for any client.
#127.0.0.1:smtps inet n  -       y       -       -       smtpd
#smtps     inet  n       -       y       -       -       smtpd
  -o syslog_name=postfix/smtps
  -o smtpd_tls_wrappermode=yes
  -o smtpd_sasl_auth_enable=yes
#  -o smtpd_reject_unlisted_recipient=no
#  -o smtpd_client_restrictions=$mua_client_restrictions
#  -o smtpd_helo_restrictions=$mua_helo_restrictions
#  -o smtpd_sender_restrictions=$mua_sender_restrictions
  -o smtpd_recipient_restrictions=permit_mynetworks,permit_sasl_authenticated,reject
  -o smtpd_relay_restrictions=permit_sasl_authenticated,reject
  -o smtpd_sasl_type=dovecot
  -o smtpd_sasl_path=private/auth
#  -o milter_macro_daemon_name=ORIGINATING
#628       inet  n       -       y       -       -       qmqpd
pickup    unix  n       -       y       60      1       pickup
cleanup   unix  n       -       y       -       0       cleanup
qmgr      unix  n       -       n       300     1       qmgr
#qmgr     unix  n       -       n       300     1       oqmgr
tlsmgr    unix  -       -       y       1000?   1       tlsmgr
rewrite   unix  -       -       y       -       -       trivial-rewrite
bounce    unix  -       -       y       -       0       bounce
defer     unix  -       -       y       -       0       bounce
trace     unix  -       -       y       -       0       bounce
verify    unix  -       -       y       -       1       verify
flush     unix  n       -       y       1000?   0       flush
proxymap  unix  -       -       n       -       -       proxymap
proxywrite unix -       -       n       -       1       proxymap
smtp      unix  -       -       y       -       -       smtp
relay     unix  -       -       y       -       -       smtp
        -o syslog_name=postfix/$service_name
#       -o smtp_helo_timeout=5 -o smtp_connect_timeout=5
showq     unix  n       -       y       -       -       showq
error     unix  -       -       y       -       -       error
retry     unix  -       -       y       -       -       error
discard   unix  -       -       y       -       -       discard
local     unix  -       n       n       -       -       local
virtual   unix  -       n       n       -       -       virtual
lmtp      unix  -       -       y       -       -       lmtp
anvil     unix  -       -       y       -       1       anvil
scache    unix  -       -       y       -       1       scache
postlog   unix-dgram n  -       n       -       1       postlogd
#
# ====================================================================
# Interfaces to non-Postfix software. Be sure to examine the manual
# pages of the non-Postfix software to find out what options it wants.
#
# Many of the following services use the Postfix pipe(8) delivery
# agent.  See the pipe(8) man page for information about ${recipient}
# and other message envelope options.
# ====================================================================
#
# maildrop. See the Postfix MAILDROP_README file for details.
# Also specify in main.cf: maildrop_destination_recipient_limit=1
#
maildrop  unix  -       n       n       -       -       pipe
  flags=DRXhu user=vmail argv=/usr/bin/maildrop -d ${recipient}
#
# ====================================================================
#
# Recent Cyrus versions can use the existing "lmtp" master.cf entry.
#
# Specify in cyrus.conf:
#   lmtp    cmd="lmtpd -a" listen="localhost:lmtp" proto=tcp4
#
# Specify in main.cf one or more of the following:
#  mailbox_transport = lmtp:inet:localhost
#  virtual_transport = lmtp:inet:localhost
#
# ====================================================================
#
# Cyrus 2.1.5 (Amos Gouaux)
# Also specify in main.cf: cyrus_destination_recipient_limit=1
#
#cyrus     unix  -       n       n       -       -       pipe
#  flags=DRX user=cyrus argv=/cyrus/bin/deliver -e -r ${sender} -m ${extension} ${user}
#
# ====================================================================
# Old example of delivery via Cyrus.
#
#old-cyrus unix  -       n       n       -       -       pipe
#  flags=R user=cyrus argv=/cyrus/bin/deliver -e -m ${extension} ${user}
#
# ====================================================================
#
# See the Postfix UUCP_README file for configuration details.
#
uucp      unix  -       n       n       -       -       pipe
  flags=Fqhu user=uucp argv=uux -r -n -z -a$sender - $nexthop!rmail ($recipient)
#
# Other external delivery methods.
#
ifmail    unix  -       n       n       -       -       pipe
  flags=F user=ftn argv=/usr/lib/ifmail/ifmail -r $nexthop ($recipient)
bsmtp     unix  -       n       n       -       -       pipe
  flags=Fq. user=bsmtp argv=/usr/lib/bsmtp/bsmtp -t$nexthop -f$sender $recipient
scalemail-backend unix -       n       n       -       2       pipe
  flags=R user=scalemail argv=/usr/lib/scalemail/bin/scalemail-store ${nexthop} ${user} ${extension}
mailman   unix  -       n       n       -       -       pipe
  flags=FRX user=list argv=/usr/lib/mailman/bin/postfix-to-mailman.py ${nexthop} ${user}
```

## 2.9 restart and check configurations:
```bash
sudo systemctl restart postfix.service
```
```bash
postfix check
```
```bash
postfix -n
````
```bash
sudo systemctl restart postfix
````
```bash
sudo ss -lnpt | grep master
````
***To disable backwards compatibility use:***
```bash
postconf compatibility_level=3.6
```
```bash
postfix reload
```

# 3. Dovecot Installation:

## 3.1 SASL and Configure SASL:
```bash
sudo apt-get install dovecot-imapd dovecot-pop3d
```
```bash
sudo apt install dovecot dovecot-core dovecot-imapd dovecot-pop3d dovecot-lmtpd dovecot-mysql dovecot-sieve dovecot-managesieved
```
```bash
apt-get install libsasl2-modules
```
* Add ``lmtp`` and ``sieve`` and ``imap`` and ``pop3``  to the supported protocols.
```bash
protocols = imap pop3 lmtp sieve
```

## 3.2 Dovecot SSL configuration
- Update this configurations:
    
* Next, edit /etc/dovecot/conf.d/10-ssl.conf and amend following lines to specify that Dovecot should use these custom certificates :
```bash
nano /etc/dovecot/conf.d/10-ssl.conf
```
```bash
ssl = yes #or set to: required
ssl_cert = </etc/cyberredcert/cert.crt #(it's better that use .crt and .key files)
ssl_key = </etc/cyberredcert/private.key #(it's better that use .crt and .key files)
ssl_client_ca_dir = /etc/ssl/certs
ssl_dh = </usr/share/dovecot/dh.pem
ssl_prefer_server_ciphers = yes
ssl_min_protocol = TLSv1.2

```

## 3.3 permit use of SMTP-AUTH by Outlook clients,
- Update this configurations:
```bash
nano /etc/dovecot/conf.d/10-auth.conf
```
```bash
disable_plaintext_auth = yes
auth_username_format = %n
auth_mechanisms = plain login
!include auth-system.conf.ext
```

## 3.4 To access to Mail Directory: 
```bash
nano /etc/dovecot/conf.d/10-mail.conf 
```
- Update this configurations:
```bash
mail_location = maildir:~/Maildir 
#(same as home on Postfix main.cf file)
```

## 3.5 Configure master, lda, lmtp files to enable protocols:
- Update this configurations same as this:
- Be careful about the syntax. Each opening bracket needs to be paired with a closing bracket.
```bash
nano /etc/dovecot/conf.d/10-master.conf 
```
```bash
service imap-login {
  inet_listener imap {
    port = 143
  }
  inet_listener imaps {
    port = 993
    ssl = yes
  }

}

service pop3-login {
  inet_listener pop3 {
    port = 110
  }
  inet_listener pop3s {
    port = 995
    ssl = yes
  }
}

service submission-login {
  inet_listener submission {
    port = 587
  }
}

service lmtp {
  unix_listener /var/spool/postfix/private/dovecot-lmtp {
    group = postfix
    mode = 0666
    user = postfix
  }
service auth {
    unix_listener /var/spool/postfix/private/auth {
      mode = 0660
      user = postfix
      group = postfix
    }
}
```
* Open the /etc/dovecot/conf.d/15-lda.conf file.
```bash
nano /etc/dovecot/conf.d/15-lda.conf 
```
* add this config:
```bash
protocol lda {
    # Space separated list of plugins to load (default is global mail_plugins).
    mail_plugins = $mail_plugins sieve
}
```
* Open the /etc/dovecot/conf.d/20-lmtp.conf file.
```bash
nano /etc/dovecot/conf.d/20-lmtp.conf 
```
* add this config:
```bash
protocol lmtp {
      mail_plugins = quota sieve
}
```
## 3.5 Service restart:
```bash
sudo systemctl restart postfix dovecot
```
```bash
sudo ss -lnpt | grep dovecot
```
## 3.6  Test your Conetion Ports
>Test your setup
SMTP-AUTH configuration is complete – now it is time to test the setup. To see if SMTP-AUTH and TLS work properly, run the following command:
```bash
telnet mail.example.com 25
telnet gmail-smtp-in.l.google.com 25
```
>If you see the following in the output, then everything is working perfectly. Type quit to exit.
```
220 cyberred.org ESMTP Postfix (Linux) [308 ms]  
EHLO keeper-us-east-1d.mxtoolbox.com  
250-host.com 
250-PIPELINING  
250-SIZE 10240000  
250-VRFY  
250-ETRN  
250-STARTTLS  
250-ENHANCEDSTATUSCODES  
250-8BITMIME  
250-DSN  
250-SMTPUTF8  
250 CHUNKING [322 ms]  
MAIL FROM:<supertool@mxtoolboxsmtpdiag.com>  
250 2.1.0 Ok [317 ms]  
RCPT TO:<test@mxtoolboxsmtpdiag.com>  
454 4.7.1 <test@mxtoolboxsmtpdiag.com>: Relay access denied [307 ms]  
  
LookupServer 4685ms
```


# 4. DNS Record Introduce:

## 4.1 Explain necessary mails records:

> ***add TXT Record on bind9 server or Domain host and Configurations db on bind9:***
```bash
domain.com	IN	A	83.136.253.111
domain.com	IN	MX	1   mail.example.com. 
@		IN	MX	1   mail.example.com. 
@		IN	MX	2   mail.example.com.
@       	IN      TXT	v=spf1 ip4:83.136.253.111 include:_spf.google.com include: ~all
@       	IN      TXT	v=DKIM1; h=sha256; k=rsa; p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEArw+jkhwYC0SpuVtXtuVKysWXjq6uCu/c1sqTE6DoFE8V4adol90VxiT93HpbKG4Ih2wevDXXhWZsN//u0qhkLb3iBlEtkRzryX1Dz2MeX3W72fm/tbi5Q6SASxxetAojrQQjJtpqDZnLCnqFsWLBj+0hl6SVyo96g7h6PReAd7o27zRE1EC3W4dSOArKtQzbufCKkvURuVtnWH1kntjLRFN3yqfvW5wAzMRCC8Cdk4KERhpzxFtjL7r2sdyrjVTTJTpzX2Hea74H/bVSWefHjubjkZBy634RSAWmpao4rQt2eaUkB6bKpg5VJlFZEebPQr2GZkzViuDi5gyf+0byhQIDAQAB
@		IN	TXT	"v=DMARC1; p=none; rua=mailto:734eda41@mxtoolbox.dmarc-report.com; ruf=mailto:734eda41@forensics.dmarc-report.com; fo=1"
```

***DNS Record (A):*** 
```
domain.com	IN	A	83.136.253.111
```
PTR Record:
```
193.252.76.in-addr.arpa. IN  SOA  ns1.swbell.net. rm-hostmaster.ems.att.com.
```
***MX Records:***
```
domain.com	IN	MX	1   mail.example.com. 
@		IN	MX	1   mail.example.com. 
@		IN	MX	2   mail.example.com.
```

***SPF Records:***
```
@       	IN      TXT	v=spf1 ip4:83.136.253.111 include:_spf.google.com include: ~all
```

***DKIM Record:***
```
@       	IN      TXT	v=DKIM1; h=sha256; k=rsa; p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEArw+jkhwYC0SpuVtXtuVKysWXjq6uCu/c1sqTE6DoFE8V4adol90VxiT93HpbKG4Ih2wevDXXhWZsN//u0qhkLb3iBlEtkRzryX1Dz2MeX3W72fm/tbi5Q6SASxxetAojrQQjJtpqDZnLCnqFsWLBj+0hl6SVyo96g7h6PReAd7o27zRE1EC3W4dSOArKtQzbufCKkvURuVtnWH1kntjLRFN3yqfvW5wAzMRCC8Cdk4KERhpzxFtjL7r2sdyrjVTTJTpzX2Hea74H/bVSWefHjubjkZBy634RSAWmpao4rQt2eaUkB6bKpg5VJlFZEebPQr2GZkzViuDi5gyf+0byhQIDAQAB
```

***DMARC Record:***
```
@		IN	TXT	"v=DMARC1; p=quarantine; rua=mailto:734eda41@mxtoolbox.dmarc-report.com; ruf=mailto:734eda41@forensics.dmarc-report.com; fo=1"
```
# 4.2 OpenDkim Configurations:
```bash
sudo chown -R opendkim:opendkim /etc/opendkim
sudo nano /etc/opendkim.conf
```
- Update this configurations:
```bash
AutoRestart Yes
AutoRestartRate 10/1h
Umask 002
Syslog yes
SyslogSuccess Yes
LogWhy Yes
Mode sv
Canonicalization relaxed/simple
UserID opendkim:opendkim
Socket inet:12301@localhost
PidFile /var/run/opendkim/opendkim.pid
ExternalIgnoreList refile:/etc/opendkim/TrustedHosts
InternalHosts refile:/etc/opendkim/TrustedHosts
KeyTable refile:/etc/opendkim/KeyTable
SigningTable refile:/etc/opendkim/SigningTable
SignatureAlgorithm rsa-sha256 
```

# 4.3 Connect the milter to Postfix:

```bash
sudo nano /etc/default/opendkim
```
_Add the following line, edit the port number only if a custom one is used_
```
SOCKET="inet:12301@localhost"
```
# 4.4 Add Postfix DKIM Configuration:
```bash
sudo nano /etc/postfix/main.cf
```
```bash
milter_protocol = 2
milter_default_action = accept
smtpd_milters = inet:localhost:12301
non_smtpd_milters = inet:localhost:12301
```
_optional_
```
smtpd_milters = unix:/spamass/spamass.sock, inet:localhost:12301
non_smtpd_milters = unix:/spamass/spamass.sock, inet:localhost:12301
```
# 4.5 Create a directory structure

```bash
sudo mkdir /etc/opendkim
sudo mkdir /etc/opendkim/keys
```
# 4.6 Add Trusted Hosts
```bash
sudo nano /etc/opendkim/TrustedHosts
```
```bash
127.0.0.1
localhost
192.168.0.1/24

*.example.com

#*.example.net
#*.example.org
```
# 4.7 Create Table

```bash
sudo nano /etc/opendkim/KeyTable
```
_Customize and add the following lines to the newly created file. Multiple domains can be specified, do not edit the first three lines:_
```
mail._domainkey.example.com example.com:mail:/etc/opendkim/keys/example.com/mail.private

#mail._domainkey.example.net example.net:mail:/etc/opendkim/keys/example.net/mail.private
#mail._domainkey.example.org example.org:mail:/etc/opendkim/keys/example.org/mail.private
```

# 4.8 Create a signing table:

```bash
sudo nano /etc/opendkim/SigningTable
```
_This file is used for declaring the domains/email addresses and their selectors._
```
*@example.com mail._domainkey.example.com

#*@example.net mail._domainkey.example.net
#*@example.org mail._domainkey.example.org
```

# 4.9 Generate the public and private keys
_Change to the keys directory:_
```bash
cd /etc/opendkim/keys
```
_Create a separate folder for the domain to hold the keys:_
```bash
sudo mkdir example.com
cd example.com
```
_Generate the keys_
```bash
sudo opendkim-genkey -s mail -d example.com
```

> -s specifies the selector and -d the domain, this command will create two files, mail.private is our private key and mail.txt contains the public key.

_Change the owner of the private key to opendkim:_

```bash
sudo chown opendkim:opendkim mail.private
```
# 4.10 Add the public key to the domain’s DNS records
_Open mail.txt:_
```
mail._domainkey IN TXT "v=DKIM1; k=rsa; p=MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQC5N3lnvvrYgPCRSoqn+awTpE+iGYcKBPpo8HHbcFfCIIV10Hwo4PhCoGZSaKVHOjDm4yefKXhQjM7iKzEPuBatE7O47hAx1CJpNuIdLxhILSbEmbMxJrJAG0HZVn8z6EAoOHZNaPHmK2h4UUrjOG8zA5BHfzJf7tGwI+K619fFUwIDAQAB" ; ----- DKIM key mail for example.com
```
* Copy that key and add a TXT record to your domain’s DNS entries:

```
Name: mail._domainkey.example.com.
Text: "v=DKIM1; k=rsa; p=MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQC5N3lnvvrYgPCRSoqn+awTpE+iGYcKBPpo8HHbcFfCIIV10Hwo4PhCoGZSaKVHOjDm4yefKXhQjM7iKzEPuBatE7O47hAx1CJpNuIdLxhILSbEmbMxJrJAG0HZVn8z6EAoOHZNaPHmK2h4UUrjOG8zA5BHfzJf7tGwI+K619fFUwIDAQAB"
```
# 4.11 Restart Service:
```bash
sudo service postfix restart
sudo service opendkim restart
```
> _The configuration can be tested by sending an empty email to ``check-auth@verifier.port25.com`` and a reply will be received. If everything works correctly you should see DKIM ``check: pass`` under Summary of Results._

> _Alternatively, you can send a message to a Gmail address that you control, view the received email’s headers in your Gmail inbox, dkim=pass should be present in the Authentication-Results header field._
```
Authentication-Results: mx.google.com;
       spf=pass (google.com: domain of contact@example.com designates --- as permitted sender) smtp.mail=contact@example.com;
       dkim=pass header.i=@example.com;
```

# 5. Rouncube 

## 5.1 update and upgrade:
```bash
sudo apt-get update
```
```bash
sudo apt-get upgrade
```

## 5.2 Install apache2 and php7:
```bash
sudo apt-get install apache2 mariadb-server php7.2 php7.2-gd php-mysql php7.2-curl php7.2-zip php7.2-ldap php7.2-mbstring php-imagick php7.2-intl php7.2-xml unzip wget curl -y
```
```bash
sudo nano /etc/php/7.2/apache2/php.ini
```
Set TimeZone:
```
date.timezone = Asia/Tehran
```

## 5.3 install Mysql:
```bash
sudo apt install mysql-client mysql-server
```
```bash
sudo mysql -p
```
```bash
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'MasterCyber123!@#';
exit
```
```bash
sudo mysql_secure_installation
```
```bash
sudo mysql -u root -p
```
```bash
CREATE DATABASE rc_db;
CREATE USER 'cyberrc'@'localhost' IDENTIFIED BY 'P@ssw0rdRc';
GRANT ALL PRIVILEGES ON rc_db.* TO 'cyberrc'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

## 5.4 restart services 
```bash
sudo systemctl start apache2
```
```bash
sudo systemctl enable apache2
```
```bash
sudo systemctl start mysql
```
```bash
sudo systemctl enable mysql
```

## 5.5 Roundcube Install:
```bash
cd /tmp
```

>_wget https://github.com/roundcube/roundcubemail/releases/download/1.3.8/roundcubemail-1.6.3-complete.tar.gz_
>
```bash
tar -xvzf roundcubemail-1.6.2-complete.tar.gz
mkdir -p /var/www/html/roundcub
chown www-data -R www-data:www-data /var/www/html/roundcub
sudo chmod -R 775 /var/www/html/roundcube
rsysnc -av /tmp/* /var/www/html/roundcub
```
**Now initiate the database by executing the following command:**
```bash
mysql -u cyberrc -p rc_db < /var/www/roundcube/SQL/mysql.initial.sql 
```
## 5.6 apache Configuration:
```bash
sudo nano /etc/apache2/sites-available/roundcube.conf
```
```bash
<VirtualHost *:80>
 ServerName example.com
 ServerAdmin admin@example.com
 DocumentRoot /var/www/html/roundcube

ErrorLog ${APACHE_LOG_DIR}/roundcube_error.log
 CustomLog ${APACHE_LOG_DIR}/roundcube_access.log combined

<Directory /var/www/html/roundcube>
 Options -Indexes
 AllowOverride All
 Order allow,deny
 allow from all
 </Directory>
 </VirtualHost>
```
## 5.7 Enable sites
 ```bash
sudo a2ensite roundcube
sudo a2enmod rewrite
sudo systemctl restart apache2
```

## 5.8 Options to crate free cert and redirect to https:
```bash
certbot --apache
```
## 5.9 Configure Roundcube
>_Now, Open your browser and navigate to  **http://host.com/roundcubemail/installer.**

![Roundcube installation Wizzard ](https://tecadmin.net/wp-content/uploads/2021/07/roundcube-step-1.png)

![Roundcube installation Step 2](https://tecadmin.net/wp-content/uploads/2021/07/roundcube-step-2.png)


![](https://www.nginxweb.ir/blog/images/2019/05/15568927536446.png)

![](https://www.nginxweb.ir/blog/images/2019/05/15568927545714.png)

**Create Configuration file:**
![](https://www.nginxweb.ir/blog/images/2019/05/15568928263420.png)

**Continue**
![](https://www.nginxweb.ir/blog/images/2019/05/15568928619111.png)

## 5.10 Removing Sensitive Information from Email Headers
> _By default, Roundcube will add a User-Agent email header, indicating that you are using Roundcube webmail and the version number. You can tell Postfix to ignore it so recipient can not see it. Run the following command to create a header check file_
```bash
sudo nano /etc/postfix/smtp_header_checks
```
**Put the following lines into the file.**
```
/^User-Agent.*Roundcube Webmail/            IGNORE
```
```bash
sudo nano /etc/postfix/main.cf
```
**Add the following line at the end of the file.**
```bash
smtp_header_checks = regexp:/etc/postfix/smtp_header_checks
```
_Save and close the file. Then run the following command to rebuild hash table._
```bash
sudo postmap /etc/postfix/smtp_header_checks
```
***Reload Postfix for the change to take effect.***
```bash
sudo systemctl reload postfix
```
## 5.11 Increase Upload File Size Limit
> _If you use PHP-FPM to run PHP scripts, then files such as images, PDF files uploaded to Roundcube can not be larger than 2MB. To increase the upload size limit, edit the PHP configuration file._
```bash
sudo nano /etc/php/7.2/apache2/php.ini
```
```
upload_max_filesize = 50M
post_max_size = 50M
```
***Alternatively, you can run the following two commands to change the value without manually opening the file.***
```bash
sudo sed -i 's/upload_max_filesize = 2M/upload_max_filesize = 50M/g' /etc/php/7.2/fpm/php.ini
sudo sed -i 's/post_max_size = 8M/post_max_size = 50M/g' /etc/php/7.2/fpm/php.ini
```
```bash
systemctl restart apache2
```
* There are 3 plugins in Roundcube for attachments/file upload:

* database_attachments
* filesystem_attachments
* redundant_attachments
_Roundcube can use only one plugin for attachments/file uploads. I found that the database_attachment plugin can be error_prone and cause you trouble. To disable it, edit the Roundcube config file._
```bash
sudo nano /var/www/roundcube/config/config.inc.php
```
_Scroll down to the end of this file. You will see a list of active plugins. Remove 'database_attachments' from the list. Note that you need to activate at least one other attachment plugin, for example, filesystem_attachments._
```bash
// ----------------------------------
// PLUGINS
// ----------------------------------
// List of active plugins (in plugins/ directory)
$config['plugins'] = ['acl', 'additional_message_headers', 'archive', 'attachment_reminder', 'autologon', 'debug_logger', 'emoticons', 'enigma', 'filesystem_attachments', 'help', 'hide_blockquote', 'http_authentication', 'identicon', 'identity_select', 'jqueryui', 'krb_authentication', 'managesieve', 'markasjunk', 'new_user_dialog', 'new_user_identity', 'newmail_notifier', 'password', 'reconnect', 'redundant_attachments', 'show_additional_headers', 'squirrelmail_usercopy', 'subscriptions_option', 'userinfo', 'vcard_attachments', 'virtuser_file', 'virtuser_query', 'zipdownload'];
```
***Save and close the file.***
