# Security

Seafile provides an advanced feature called encrypted library to protect your privacy. The file encryption/decryption is performed in the client-side. The password of an encrypted library is not stored in the server. **Even the system admin of the server can't view the file contents**.

## How does an encrypted library work?

When you create an encrypted library, you'll provide a password for it. All the data in that library will be encrypted with the password before uploading to the server.

The encryption procedure is:

1. Generate a 32-byte long cryptographically strong random number. This will be used as the file encryption key ("file key").
2. Encrypt the file key with the user provided password. We first use PBKDF2 algorithm to derive a key/iv pair from the password, then use AES 256/CBC to encrypt the file key. The result is called the "encrypted file key". This encrypted file key will be sent to and stored on the server. When you need to access the data, you can decrypt the file key from the encrypted file key.
3. All file data is encrypted by the file key with AES 256/CBC. We use PBKDF2 algorithm to derive key/iv pair from the file key. After encryption, the data is uploaded to the server.

The above encryption procedure can be executed on both the desktop client and on web browser. The browser-side encryption feature can be enabled on the server. Whenever you upload to or download from an encrypted library, the following happens:

* The server sends back the encrypted data, then the browser decrypts them with JavaScript on the client side;
* The browser encrypts the data with JavaScript on the client side then sends encrpted data to the server. The server just store the encrypted result directly.

In both cases, your password won't be transferred to the server side.

When you sync an encrypted library to the desktop or browse a library in the web browser, the client/browser needs to verify your password. When you create the library, a "magic token" is derived from the password and library id. This token is stored with the library on the server side. The client use this token to check whether your password is correct before you sync or browse the library. The magic token is generated by PBKDF2 algorithm with 1000 iterations. So it's safe from brute force attack.

For maximum security, the plain-text password won't be saved on the client side too. The client only saves the key/iv pair derived from the "file key", which is used to decrypt the data. So if you forget the password, you won't be able to recover it or access your data on the server.

## How is the connection between client and server encrypted?

Every seafile desktop client has a unique private key. When a client and a server connect, they will exchange public key and negotiate a session key. The session key is derived from cryptographically secure random number with PDKDF2 algorithm. And it's exchanged between the client and the server with RSA encryption. This session key will be used to encrypt the data transfer with AES-256/CBC algorithm.


## Why httpserver delivers every content to everybody knowing the content URL of an unshared private file?

When a file download link is clicked, a random URL is generated for user to access the file from httpserver. This url will be valid for 1 hour to support web browser cache. 

In the future, we will add mechanism to further prevent brute-force attack by trying URLs.
