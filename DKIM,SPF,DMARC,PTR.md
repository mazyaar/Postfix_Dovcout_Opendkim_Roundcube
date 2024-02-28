
** Part 2 **

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

