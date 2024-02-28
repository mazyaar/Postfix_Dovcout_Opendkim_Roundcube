
# Part 2

# 4. DNS Record Introduce:

![Before visit Bind9 DNS server configurations]([https://tecadmin.net/wp-content/uploads/2021/07/roundcube-step-1.png](https://github.com/mazyaar/Ubuntu_DNS_Server_Bind9))


## 4.1 Explain necessary mails records:

> ***add TXT Record on bind9 server or Domain host and Configurations db on bind9:***
```bash
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     domain.com. root.domain.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;

domain.com	IN	A	83.136.253.111
mydomain.com       IN      MX      10 domain.com.
@		IN	MX	10  mx.domain.com.
@       	IN      TXT	v=spf1 a:cyberred.org ip4:1.2.3.4/24 ip6:2aba:5e80::ed74:651f:858a:1/112 include:google.com include:yahoo.com include:msn.com -all
@       	IN      TXT	v=DKIM1; h=sha256; k=rsa; p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEArw+jkhwYC0SpuVtXtuVKysWXjq6uCu/c1sqTE6DoFE8V4adol90VxiT93HpbKG4Ih2wevDXXhWZsN//u0qhkLb3iBlEtkRzryX1Dz2MeX3W72fm/tbi5Q6SASxxetAojrQQjJtpqDZnLCnqFsWLBj+0hl6SVyo96g7h6PReAd7o27zRE1EC3W4dSOArKtQzbufCKkvURuVtnWH1kntjLRFN3yqfvW5wAzMRCC8Cdk4KERhpzxFtjL7r2sdyrjVTTJTpzX2Hea74H/bVSWefHjubjkZBy634RSAWmpao4rQt2eaUkB6bKpg5VJlFZEebPQr2GZkzViuDi5gyf+0byhQIDAQAB
@		IN	TXT	"v=DMARC1; p=reject; rua=mailto:info@domain.com; ruf=mailto:info@domain.com; sp=reject; aspf=s; adkim=s; fo=0:1:d:s;"
```

***DNS Record (A):*** 
```
domain.com	IN	A	83.136.253.111
```

***PTR Record:***
```
83.136.253.in-addr.arpa. IN  SOA  domain.com.
```

***MX Records:***
```
domain.com       IN      MX      10 domain.com.
@		IN	MX	10  domain.com.
domain.com    IN    A       1.2.3.4
```

***SPF Records:***
```
@       	IN      TXT	v=spf1 a:cyberred.org ip4:1.2.3.4/24 ip6:2aba:5e80::ed74:651f:858a:1/112 include:google.com include:yahoo.com include:msn.com -all
```
![Spf Generators](https://mxtoolbox.com/SPFRecordGenerator.aspx)

***DKIM Record:***
```
@       	IN      TXT	v=DKIM1; h=sha256; k=rsa; p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEArw+jkhwYC0SpuVtXtuVKysWXjq6uCu/c1sqTE6DoFE8V4adol90VxiT93HpbKG4Ih2wevDXXhWZsN//u0qhkLb3iBlEtkRzryX1Dz2MeX3W72fm/tbi5Q6SASxxetAojrQQjJtpqDZnLCnqFsWLBj+0hl6SVyo96g7h6PReAd7o27zRE1EC3W4dSOArKtQzbufCKkvURuVtnWH1kntjLRFN3yqfvW5wAzMRCC8Cdk4KERhpzxFtjL7r2sdyrjVTTJTpzX2Hea74H/bVSWefHjubjkZBy634RSAWmpao4rQt2eaUkB6bKpg5VJlFZEebPQr2GZkzViuDi5gyf+0byhQIDAQAB
```
![DKIM Generators](https://dmarcly.com/tools/dkim-record-generator)

***DMARC Record:***
```
@		IN	TXT	"v=DMARC1; p=reject; rua=mailto:info@domain.com; ruf=mailto:info@domain.com; sp=reject; aspf=s; adkim=s; fo=0:1:d:s;"
```
![DMARC Generators](https://mxtoolbox.com/DMARCRecordGenerator.aspx)

***Troubleshooting:***

dig domain.com

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

*.domain.com

#*.domain.net
#*.domain.org
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

![Bind 9 on Ubuntu](https://help.ubuntu.com/community/BIND9ServerHowto)
![Help prevent spoofing and spam with DMARC](https://support.google.com/a/answer/2466580?hl=en)
![How to Implement DMARC/DKIM/SPF to Stop Email Spoofing/Phishing: The Definitive Guide](https://dmarcly.com/blog/how-to-implement-dmarc-dkim-spf-to-stop-email-spoofing-phishing-the-definitive-guide)
