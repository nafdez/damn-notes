## Table of Contents
* [Generating keys](#gen-key)
	+ [Generating private key](#gen-priv)
	+ [Generating certificate](#gen-cert)
		- [Create a CA](#gen-cert-ca)
		- [Create a request](#gen-cert-req)
		- [Sign a certificate](#gen-cert-sign)
	+ [Generating keystore](#gen-ks)
* [Security operations](#sec-ops)
	+ [Retrieving keys](#sec-keys) 
		- [Keystore setup](#sec-keys-ks-setup)
		- [Get certificate](#sec-keys-get-cert)
		- [Get private key](#sec-keys-get-pvk)
		- [Get public key](#sec-keys-get-pubk)
	+ [Sign](#sec-sign)
	+ [Encrypt](#sec-enc)
	+ [Decrypt](#sec-dec)
	+ [Verify](#sec-ver)

---

<a id="gen-key"></a>
## Generating keys

Openssl commands to use in this section:
- **genrsa**: Generation of RSA Private Key
- **req**: PKCS#10 X.509 Certificate Signing Request (CSR) Management
- **x509**: X.509 Certificate Data Management
- **pkcs12**: PKCS#12 Data Management

Full snippet:

```bash
openssl genrsa -out rsa_priv.pem -aes256 4096

openssl req -newkey rsa:4096 -x509 -keyout cakey -out ca.crt -days 3650 -nodes -subj "/C=ES/ST=ASTURIAS/L=OVIEDO/O=DAM/CN=DAMCA"

openssl req -new -key rsa_priv.pem -out mycert.csr -subj "/C=ES/ST=ASTURIAS/L=OVIEDO/O=DAM/CN=JULIO"

openssl x509 -req -in mycert.csr -days 3650 -CA ca.crt -CAkey cakey -set_serial 01 -out mycertificate.crt

openssl pkcs12 -export -in mycertificate.crt -inkey rsa_priv.pem -out keystore.p12 -name mykey
```

> [!NOTE] See man openssl for more info and alternative commands

---

<a id="gen-priv"></a>
### Generating private key

Generated files:
- *rsa_priv.pem*

```bash
openssl genrsa -out rsa_priv.pem -aes256 4096
```

Breakdown:
- *-out*: output the key to the specified file
- *-aes256*: encrypt the private key with aes256
- *4096*: size of the private key to generate in bits

> [!NOTE] See man openssl-genrsa for more information

---

<a id="gen-cert"></a>
### Generating certificate

All commands:

```bash
openssl req -newkey rsa:4096 -x509 -keyout cakey -out ca.crt -days 3650 -nodes -subj "/C=ES/ST=ASTURIAS/L=OVIEDO/O=DAM/CN=DAMCA"

openssl req -new -key rsa_priv.pem -out mycert.csr -subj "/C=ES/ST=ASTURIAS/L=OVIEDO/O=DAM/CN=JULIO"

openssl x509 -req -in mycert.csr -days 3650 -CA ca.crt -CAkey cakey -set_serial 01 -out mycertificate.crt
```

Breakdown to each command:

<a id="gen-cert-ca"></a>
#### Create a CA certificate (Authority)

A CA certificate is a digital certificate issued by a certificate authority (CA). The CA verifies trusted certificates for trusted roots. Trusted roots are the foundation upon which chains of trust are built in certificates.

Trusting a CA root means that you trust all certificates issued by that CA.

CA certificates contain a public key corresponding to a private key. The CA owns the private key and uses it to sign the certificates it issues. To validate a trusted certificate, you must first check in a CA certificate. [Reference](https://www.ibm.com/docs/en/b2b-integrator/5.2?topic=certificates-ca)

(Keep in mind what we're doing are self-signed certificates)

These keys will be used to sign other certificates

Generated files:
- *ca.crt*
- *cakey*

```bash
openssl req -newkey rsa:4096 -x509 -keyout cakey -out ca.crt -days 3650 -nodes -subj "/C=ES/ST=ASTURIAS/L=OVIEDO/O=DAM/CN=DAMCA"
```

- *newkey*: Used to generate a new private key
	- *rsa:4096*: generates a rsa key with nbits in size 
- *-x509*: outputs a certificate instead of a certificate request
- *-keyout*: don't ask me
- *-out*: specifies filename to write
- *-days*: specifies the number of days this certificate is valid
- *-nodes*: specifies if the generated private key should be encrypted
- *-subj*: sets subject name for new request or supersedes when processing a request

---

<a id="gen-cert-req"></a>
#### Create a certificate request

Creates a certificate request that the CA will later be signing and returning a valid certificate from this

Generated files:
- *mycert.csr*

```bash
openssl req -new -key rsa_priv.pem -out mycert.csr -subj "/C=ES/ST=ASTURIAS/L=OVIEDO/O=DAM/CN=JULIO"
```

- *-new*: generates a new certificate request
- *-key*: provides the private key for signing a new certificate or cert request

---

<a id="gen-cert-sign"></a>
#### Sign a certificate

Signs the certificate request previously created

Generated files:
- *mycertificate.crt*

```bash
openssl x509 -req -in mycert.csr -days 3650 -CA ca.crt -CAkey cakey -set_serial 01 -out mycertificate.crt
```

- *-req*: specifies that a certificate is expected on input
- *-in*: certificate path
- *-CA*: specifies CA certificate to be used for signing
- *-CAkey*: sets the CA private key to sign a certificate with
- *-set-serial*: specifies serial number to use

> [!NOTE] See man openssl-req and openssl-x509 for more information

---

<a id="gen-ks"></a>
### Generating keystore

A keystore contains personal certificates, plus the corresponding private keys that are used to identify the owner of the certificate. [Referece](https://www.ibm.com/docs/ro/zos-connect/zosconnect/3.0?topic=connect-keystores-truststores)

```bash
openssl pkcs12 -export -in mycertificate.crt -inkey rsa_priv.pem -out keystore.p12 -name mykey
```

Breakdown:
- *-export*: specifies that a PKCS#12 file will be created
- *-in*: specifies the input certificate
- *-inkey*: the private key input
- *-out*: the file to write certificates and private keys to
- *-name*: the friendly name or alias for the certificates and private key

---

<a id="sec-ops"></a>
## Security operations

<a id="sec-keys"></a>
### Retrieving keys

<a id="sec-keys-ks-setup"></a>
#### Keystore setup

To initialize a keystore you first need three parameters:
- Keystore Type, in this case PKCS12
- Keystore Path, path to the keystore
- Password, password of the keystore, needs to be a char array

Example of initializing a keystore:

```java
KeyStore keyStore = KeyStore.getInstance("PKCS12");
keyStore.load(new FileInputStream("/path/to/ks"), "password".toCharArray());
```

---

<a id="sec-keys-get-cert"></a>
#### Certificate

To get a certificate you just need the alias of the certificate. In this case is "mykey"

```java
Certificate certificate = keyStore.getCertificate("mykey");
```

---

<a id="sec-keys-get-pvk"></a>
#### Private key

For a certificate, you need the alias and the password (as char array) to get the private key

```java
PrivateKey key = 
	(PrivateKey) keyStore.getKey("mykey", "password".toCharArray());
```

---

<a id="sec-keys-get-pubk"></a>
#### Public key

In the case of the public key you need the certificate first.

```java
PublicKey key = certificate.getPublicKey();
```

---

<a id="sec-sign"></a>
### Sign

In order to sign a file you need the following:
- PrivateKey
- Algorithm for signature, in this case "SHA512withRSA"
- input path of the file to sign
- output path of the signed file

```java
try (BufferedInputStream bis = new BufferedInputStream(new FileInputStream(inputPath)); BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream(outputPath))) {

    Signature sign = Signature.getInstance("SHA512withRSA");  
    sign.initSign(privateKey);  
  
    byte[] buffer = new byte[1024];  
    int n;  
    while ((n = bis.read(buffer)) > 0) {  
        sign.update(buffer, 0, n);  
    }  
    byte[] signature = sign.sign();  
  
    bos.write(signature);  
    return true;  
} catch (Exception ignored) {  
}
```

---

<a id="sec-enc"></a>
### Encrypt

The needed params to encrypt a file are the following:
- Private key
- Transformation to use, in this case "RSA/ECB/PKCS1Padding"
- Input path
- Output path

```java
try (BufferedInputStream bis = new BufferedInputStream(new FileInputStream(inputPath)); BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream(outputPath))) {

    Cipher cipher = Cipher.getInstance("RSA/ECB/PKCS1Padding");  
	cipher.init(Cipher.ENCRYPT_MODE, privateKey); 
	 
	byte[] block = new byte[501];  
	int n;
	while ((n = bis.read(block)) != -1) 
	    bos.write(cipher.doFinal(block, 0, n));
}
```

---

<a id="sec-dec"></a>
### Decrypt

The needed params to decrypt a file are the following:
- Public key
- Transformation to use, in this case "RSA/ECB/PKCS1Padding"
- Input path
- Output path

```java
try (BufferedInputStream bis = new BufferedInputStream(new FileInputStream(inputPath)); BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream(outputPath))) {

    Cipher cipher = Cipher.getInstance("RSA/ECB/PKCS1Padding");  
	cipher.init(Cipher.DECRYPT_MODE, publicKey); 
	 
	byte[] block = new byte[512];  
	int n;
	while ((n = bis.read(block)) != -1) 
	    bos.write(cipher.doFinal(block, 0, n));
}
```

---

<a id="sec-ver"></a>
### Verify

To verify a file against it's signature you need:
- Public key
- Algorithm to use, in this case "SHA512withRSA"
- Input path of the unsigned file
- Input path of the signed file

```java
try (BufferedInputStream bisOriginal = new BufferedInputStream(new FileInputStream(inputPath)); BufferedInputStream bisSigned = new BufferedInputStream(new FileInputStream(toVerifyPath));) {

    Signature sig = Signature.getInstance("SHA512withRSA");  
    sig.initVerify(publicKey);
    
    byte[] buffer = new byte[1024];  
    int n;  
    while ((n = bisOriginal.read(buffer)) > 0) {  
        sig.update(buffer, 0, n);  
    }  
  
    return sig.verify(bisSigned.readAllBytes());  
}
```
