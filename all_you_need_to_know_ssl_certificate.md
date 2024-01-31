# Certificate Chain - intermediate and root certificate - installed required?
 
   In short: the intermediate certificates have to be sent within the TLS handshake (**needs proper configuration of the server)** and only the CA local at the client will be considered as trust anchors.

# High-level description of the protocol

After building a TCP connection, the SSL handshake is started by the client. The client (which can be a browser as well as any other program such as Windows Update or Thunderbird) sends a number of specifications:

- which version of SSL/TLS it is running,
- what ciphersuites it wants to use, and
- what compression methods it wants to use.

The server identifies the highest SSL/TLS version supported by both it and the client, picks a ciphersuite from one of the client's options (if it supports one), and optionally picks a compression method.
After this the basic setup is done, the server sends its certificate. This certificate must be trusted by either the client itself or a party that the client trusts. 
For example, if the client trusts GeoTrust, then the client can trust the certificate from Google.com because GeoTrust cryptographically signed Google's certificate.

Having verified the certificate and being certain this server really is who he claims to be (and not a man in the middle), a key is exchanged. This can be a public key, a "PreMasterSecret" or simply nothing, depending on the chosen ciphersuite. Both the server and the client can now compute the key for the symmetric encryption whynot PKE?. The client tells the server that from now on, all communication will be encrypted, and sends an encrypted and authenticated message to the server.
The server verifies that the MAC (used for authentication) is correct and that the message can be correctly decrypted. It then returns a message, which the client verifies as well.
The handshake is now finished, and the two hosts can communicate securely.

# What are the steps of a TLS handshake?

TLS handshakes are a series of datagrams, or messages, exchanged by a client and a server. A TLS handshake involves multiple steps, as the client and server exchange the information necessary for completing the handshake and making further conversation possible.

The exact steps within a TLS handshake will vary depending upon the kind of key exchange algorithm used and the cipher suites supported by both sides. The RSA key exchange algorithm, while now considered not secure, was used in versions of TLS before 1.3. It goes roughly as follows:

## Client Hello

The session begins with the client saying "Hello". The client provides the following:
- protocol version
- client random data (used later in the handshake
- an optional session id to resume
- a list of cipher suites
- a list of compression methods
- a list of extensions

## Server Hello

The server says "Hello" back. The server provides the following:
- the selected protocol version
- server random data (used later in the handshake)
- the session id
- the selected cipher suite
- the selected compression method
- a list of extensions

## Server Certificate

The server provides a certificate containing the following:
- the hostname of the server
- the public key used by this server
- proof from a trusted third party that the owner of this hostname holds the private key for this public key

## Server Key Exchange Generation

The server calculates a private/public keypair for key exchange. Key exchange is a technique where two parties can agree on the same number without an eavesdropper being able to tell what it is.

An explanation of the key exchange can be found on my X25519 site, but doesn't need to be understood in depth for the rest of this page.

The private key is chosen by selecting an integer between 0 and 2256-1. The server does this by generating 32 bytes (256 bits) of random data. The private key selected is:

909192939495969798999a9b9c9d9e9fa0a1a2a3a4a5a6a7a8a9aaabacadaeaf

## Server Key Exchange

The server provides information for key exchange. As part of the key exchange process both the server and the client will have a keypair of public and private keys, and will send the other party their public key. The shared encryption key will then be generated using a combination of each party's private key and the other party's public key.

The parties have agreed on a cipher suite using ECDHE, meaning the keypairs will be based on a selected Elliptic Curve, Diffie-Hellman will be used, and the keypairs are Ephemeral (generated for each connection) rather than using the public/private key from the certificate.

## Server Hello Done

The server indicates it's finished with its half of the handshake.

## Client Key Exchange Generation

The client calculates a private/public keypair for key exchange. Key exchange is a technique where two parties can agree on the same number without an eavesdropper being able to tell what the number is.

An explanation of the key exchange can be found on my X25519 site, but doesn't need to be understood in depth for the rest of this page.

The private key is chosen by selecting an integer between 0 and 2256-1. The client does this by generating 32 bytes (256 bits) of random data. The private key selected is:

202122232425262728292a2b2c2d2e2f303132333435363738393a3b3c3d3e3f

## Client Key Exchange

The client provides information for key exchange. As part of the key exchange process both the server and the client will have a keypair of public and private keys, and will send the other party their public key. The shared encryption key will then be generated using a combination of each party's private key and the other party's public key.

The parties have agreed on a cipher suite using ECDHE, meaning the keypairs will be based on a selected Elliptic Curve, Diffie-Hellman will be used, and the keypairs are Ephemeral (generated for each connection) rather than using the public/private key from the certificate.

## Client Encryption Keys Calculation

The client now has the information to calculate the encryption keys that will be used by each side. It uses the following information in this calculation:
server random (from Server Hello)
client random (from Client Hello)
server public key (from Server Key Exchange)
client private key (from Client Key Generation)
The client multiplies the server's public key by the client's private key using the curve25519() algorithm. The 32-byte result is called the PreMasterSecret, and is found to be:
df4a291baa1eb7cfa6934b29b474baad2697e29f1f920dcc77c8a0a088447624

## Client Change Cipher Spec

The client indicates that it has calculated the shared encryption keys and that all following messages from the client will be encrypted with the client write key.

In the next version of TLS this message type has been removed because it can be inferred.

## Client Handshake Finished

To verify that the handshake was successful and not tampered with, the client calculates verification data and encrypts it with the client write key.

The verification data is built from a hash of all handshake messages and verifies the integrity of the handshake process.

## Server encryption keys calculation

## server change cipher spec

## Server handshake finished

## Client application data ...

## Server application data

## Client Close Notify

The client sends an alert that it is closing the connection.

# The TLS 1.3 handshake
1. Server listens for new connections on port 443.
2. Client connects to port 443 and initiates the handshake process with a ClientHello message to the server. Since the list of cipher suites was vastly reduced in TLS 1.3 (from 37 to 5), the client assumes the server is going to use one of the five. It proactively calculates a key pair for each of the cipher suites and sends it to the server along with the protocol version.
3. Server calculates the key session. With the client’s key share, the server is able to generate the session key.
4. Server says “hello” back with a ServerHello message. The server generates its own key share and sends it over to the client, so it also can generate the session key, along with the server’s encrypted SSL certificate (using the session key created on #3).
5. Client generates master secret and a secure connection is established. The client receives the server’s key share and calculates the session key. It decrypts and verifies the server’s certificates, and if everything is good, the handshake is complete.



