# Simple Java SSL/TSL Socket Server

### 1. What is the role of the public key and private key?

* Public key is used to encrypt information.
* Private key is used to decrypt information.

### 2. What is the difference between Digital Signature and Encryption?

* When encrypting, you(client) use their public key to write a message, and they(server) use their private key to decrypt
  to read it.
* When signing, you(client) use your own private key to write the message’s signature, and they(server) use your public key
  to verify if the message is yours.

### 3. What is the difference between Keystore and Truststore?

* A Keystore has certs and keys in it and defines what is going to be presented to the other end of a connection.
* A truststore has just certs in it and defines what certs that the other end will send are to be trusted.

### 4. What is the standard handshake for SSL/TSL Process?

The standard SSL Handshake:

1. **Client Hello** (The server needs information to communicate with the client using SSL.)
   1. SSL version Number
   2. Cipher setting (Compression Method)
   3. Session-specific Data
2. **Server Hello**
   1. Server picks a cipher and compression that both client and server support and tells the client about its choice,
      as well as some other things like a session id.
   2. Server presents its certificate ( This is what the client needs to validate as being signed by a trusted CA.)
   3. Server presents a list of certificate authority DNS that client certs may be signed by.
3. Client response
   1. Client continues the key exchange protocol necessary to set up a TLS session.
   2. Client presents a certificate that was signed by one of the CAs and encrypts with the server’s public key.
   3. Send the pre-master (based on cipher) encrypted by Server’s public key to the server.
4. Server accepts the cert presented by the client.
   * Server uses its private key to decrypt the pre-master secret. Both client and server perform steps to generate the master secret with the agreed cipher.
5. Encryption with Session Key.
   * Both client and server exchange messages to inform them that future messages will be encrypted.

---

# 👀️ DEMO FOR SIMPLE SSL/TSL/MTSL Client, Server. . 👀️

---



Step 1. Create a private key and public certificate for the client & server by OpenSSL tool.

###### **CLIENT:**

```bash
openssl req -newkey rsa:2048 -nodes -keyout clientprivatekey.pem -x509 -days 365 -out clientpubliccert.pem
```

**SERVER:**

```bash
openssl req -newkey rsa:2048 -nodes -keyout serverprivatekey.pem -x509 -days 365 -out serverpubliccert.pem
```

In the process it asks for multiple things for the creation of certification:

```
Country Name (2 letter code) [AU]:
State or Province Name (full name) [Some-State]:
Locality Name (eg, city) []:
Organization Name (eg, company) [Internet Widgits Pty Ltd]:
Organizational Unit Name (eg, section) []:
Common Name (e.g. server FQDN or YOUR name) []:
Email Address []:
```

```
* Step 2. Combine the private key and public certificate into `PCKS12(P12)` format for client and server respectively.

```bash
openssl pkcs12 -inkey clientprivatekey.pem -in clientpubliccert.pem -export -out client-certificate.p12
```

```bash
openssl pkcs12 -inkey server-key.pem -in server-certificate.pem -export -out server-certificate.p12
```

* * It will ask for the password for `p12` format which is bascially for integratity:

```
Enter Export Password:
Verifying - Enter Export Password:
```

* Step 3. Place `client-certificate.p12` and `server-certificate.p12` into `keystore` and `trustStore` location.

  ![client-server](img/client-server.jpg)
* Step 4. Make a keystore and truststore for client:

Keystore for client

```bash
keytool -importkeystore -srckeystore client-certificate.p12 -destkeystore CLIENTKEYSTORE.jks -srcstoretype PKCS12 -deststoretype jks -srcstorepass p12Password -deststorepass keystorePassword -destkeypass confirmKeyStorePassword
```

TrustStore for Client will contain certificate of the server.

```bash
keytool -importkeystore -srckeystore server-certificate.p12 -destkeystore CLIENTTRUSTSTORE.jks -srcstoretype PKCS12 -deststoretype jks -srcstorepass passwordOfP12ServerCert -deststorepass passwordOfTrustStore -srcalias CAserver -destalias CAserver
```

if you have the CA else:

```bash
keytool -importkeystore -srckeystore client-certificate.p12 -destkeystore SERVERTRUSTSTORE.jks -srcstoretype PKCS12 -deststoretype jks -srcstorepass 1234 -deststorepass 123456
```

* Step 4. Make a keystore and truststore for SERVER:

Keystore for server

```bash
keytool -importkeystore -srckeystore server-certificate.p12 -destkeystore SERVERKEYSTORE.jks -srcstoretype PKCS12 -deststoretype jks -srcstorepass p12Password -deststorepass keystorePassword -destkeypass confirmKeyStorePassword
```

TrustStore for server will contain certificate of the client.

```bash
keytool -importkeystore -srckeystore server-certificate.p12 -destkeystore CLIENTTRUSTSTORE.jks -srcstoretype PKCS12 -deststoretype jks -srcstorepass passwordOfP12ServerCert -deststorepass passwordOfTrustStore -srcalias CAserver -destalias CAserver
```

if you have the CA else:

```bash
keytool -importkeystore -srckeystore client-certificate.p12 -destkeystore SERVERTRUSTSTORE.jks -srcstoretype PKCS12 -deststoretype jks -srcstorepass 1234 -deststorepass 123456
```
