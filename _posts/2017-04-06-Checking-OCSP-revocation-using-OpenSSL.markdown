---
layout: post
title:  "Checking OCSP revocation using OpenSSL"
date:   2017-04-6 12:18:29 +0200
categories: blog
---
# Checking OCSP revocation using OpenSSL

Exist two types of revocation methods, CRL (certificate revocation list) and OCSP (Online Certificate Status Protocol).

CRL was first released to provide the CA with the ability to revoke certificates, however due to limitations with this method it was superseded by OCSP.

## CRL
Certificate revocation list contain a list of the serial number that have been revoked by the CA. The client then check the serial number of the certificate and check if this is in the list provided by the CA.

```bash
Revoked Certificates:
    Serial Number: 2372717EAAF6BEC59800149379A0A7725
        Revocation Date: May  1 12:23:52 2016 GMT
    Serial Number: 998DDD15D25C71361FE7D4A8BCCFB4B4
        Revocation Date: May 24 17:09:23 2016 GMT
```
For indicate to the client where to find the CRL, it is embedded within each certificate.

```bash
apolo@pegasus:~$ openssl s_client -host google.com -port 443|openssl x509 -text
.....
            X509v3 Subject Key Identifier: 
                AB:B8:87:A8:9B:75:B1:F6:D0:C8:B2:FD:AC:C4:FA:63:8B:3E:C2:CC
            X509v3 Basic Constraints: critical
                CA:FALSE
            X509v3 Authority Key Identifier: 
                keyid:4A:DD:06:16:1B:BC:F6:68:B5:76:F5:81:B6:BB:62:1A:BA:5A:81:2F

            X509v3 Certificate Policies: 
                Policy: 1.3.6.1.4.1.11129.2.5.1
                Policy: 2.23.140.1.2.2

            X509v3 CRL Distribution Points: 

                Full Name:
                  URI:http://pki.google.com/GIAG2.crl
.....
```
The disadvantages to CRL are:
  1. Can create a large amount of overhead, as the client has to search through the revocation list. In some cases this can be 1000's of lines long.
  2. CRLs are updated periodically every 5-14 days. Potentially leaving the attack surface open until the next CRL update.
  3. The CRL is not checked for OV or DV based certificates.(OV: Organization Validation, DV: Domain Validation)  
  4. If the client is unable to download the CRL then by default the client will trust the certificate. 
# OCSP

OCSP remove many of disadvantages of CRL, for example permit to the client check the status for a single certificate.

The OCSP precess is very simple:

1. Client receives the certificate
2. Client sends OCSP request to the OCSP server and it query by the serial number of the certificate
3. OCSP response with a certificate status Good, Revoked or Unknown

```bash
Response verify OK
0x36F5B12D5E6FD0BD4EFF2C2C477F3B4aB: good
        This Update: Mar 19 00:24:56 2017 GMT
        Next Update: Mar 26 00:24:56 2017 GMT
```

The main advantage to OCSP is that the client don\`t need download and parse an entire list. They can query the status of a single certificate.

For check the status of one certificate using OCSP you need to perform the following steps:
1. Obtain the certificate that you wish check
2. Obtain the issuer certificate
3. Determine the URL of the OCSP responder
4. Send thee OCSP request to the responder
5. Observe the Response

In first place obtain the certificate chain with openssl:
```bash
apolo@pegasus:~/ocsp-post$ openssl s_client -connect 1and1.com:443 -showcerts
CONNECTED(00000003)
depth=2 C = US, O = GeoTrust Inc., CN = GeoTrust Primary Certification Authority
verify return:1
depth=1 C = US, O = GeoTrust Inc., CN = GeoTrust EV SSL CA - G4
verify return:1
depth=0 jurisdictionC = US, jurisdictionST = Delaware, businessCategory = Private Organization, serialNumber = 3658311, C = US, ST = Pennsylvania, L = Chesterbrook, O = 1&1 Internet Inc., CN = www.1and1.com
verify return:1
---
Certificate chain
 0 s:/jurisdictionC=US/jurisdictionST=Delaware/businessCategory=Private Organization/serialNumber=3658311/C=US/ST=Pennsylvania/L=Chesterbrook/O=1&1 Internet Inc./CN=www.1and1.com
   i:/C=US/O=GeoTrust Inc./CN=GeoTrust EV SSL CA - G4
-----BEGIN CERTIFICATE-----
..........
```
We obtain the certificate chain, if the server is good configured the first certificate will be the server certificate and the second the issuer certificate.

In this case the server is good configured. Now we have two files the server certificate (this is the certificate that we want check) and the issuer certificate.

For me the server certificate is _1and1.crt _ and the issuer certificate is _issuer.crt_

The next pass is obtain the URL of te OCSP responder:

```bash
apolo@pegasus:~/ocsp-post$ openssl x509 -in 1and1.crt -noout -text|grep OCSP
                OCSP - URI:http://gm.symcd.com
```

Send the OCSP request is very easy using openssl:

```bash
apolo@pegasus:~/ocsp-post$ openssl ocsp -issuer issuer.crt -cert 1and1.crt -url http://gm.symcd.com -CAfile issuer.crt 
WARNING: no nonce in response
Response verify OK
1and1.crt: good
	This Update: Apr  1 19:01:19 2017 GMT
	Next Update: Apr  8 19:01:19 2017 GMT
```
***NOTE: If you need check the status of the offline site you can use this web site [crt.sh](https://crt.sh/)***

Regards!


