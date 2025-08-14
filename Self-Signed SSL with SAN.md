# How to create a self-signed SSL Certificate with SubjectAltName(SAN)
After Chrome 58, self-signed certificate without SAN is not valid anymore.
Original from  KeithYeh/Self-Signed SSL with SAN.md

## Step 1: Generate a Private Key
```shell
openssl genrsa -des3 -out example.com.key 2048  #Password protected key, RSA key
openssl genrsa -out example.com.key 2048 #No Password enabled, RSA key
openssl ecparam -out example.com.key -name secp256r1 -genkey #No password ECDSA key
```

## Step 2: Generate a CSR (Certificate Signing Request)
```shell
openssl req -new -key example.com.key -out example.com.csr
```

```
Enter pass phrase for example.com.key if added at generation:
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:XX
State or Province Name (full name) []:State
Locality Name (eg, city) [Default City]:City
Organization Name (eg, company) [Default Company Ltd]:Company
Organizational Unit Name (eg, section) []:BU
Common Name (eg, your name or your server's hostname) []:*.example.com
Email Address []:admin@example.com

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
```

## Step 3: Remove Passphrase from Key if enabled
```shell
cp example.com.key example.com.key.org
openssl rsa -in example.com.key.org -out example.com.key
```

## Step 4: Create config file for SAN
```shell
touch v3.ext
```

File content
```
subjectKeyIdentifier   = hash
authorityKeyIdentifier = keyid:always,issuer:always
basicConstraints       = CA:TRUE
keyUsage               = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment, keyAgreement, keyCertSign
subjectAltName         = DNS:example.com, DNS:*.example.com
issuerAltName          = issuer:copy
```

## Step 5: Generating a Self-Signed Certificate
```shell
openssl x509 -req -in example.com.csr -signkey example.com.key -out example.com.crt -days 3650 -sha256 -extfile v3.ext
```

## One line step: single line generation, will work on this later
```shell
openssl req -x509  -noenc -newkey ec -pkeyopt ec_paramgen_curve:P-256 -days 3650  -keyout example.com.key -out example.com.crt -subj "/CN=example.com"  -addext "subjectAltName=DNS:example.com,DNS:*.example.com,IP:192.168.1.10"
```

## Reference
- http://www.akadia.com/services/ssh_test_certificate.html
- https://alexanderzeitler.com/articles/Fixing-Chrome-missing_subjectAltName-selfsigned-cert-openssl/
- https://www.ibm.com/support/knowledgecenter/SSB23S_1.1.0.12/gtps7/cfgcert.html
