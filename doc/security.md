Securing The Data
=================

Server Side
-----------

All notes are compressed using ZLIB and encrypted using Blowfish in ECB mode
before being written to disk. Never at any point is the note in plaintext on
disk.

Blowfish was chosen as the cipher for encrypting the data, as the key can be
any length, whereas AES requires static key lengths. This could be enforced
client-side with Javascript, but some clients have Javascript disabled.

ECB mode was chosen over CBC for encrypting the blocks, due to not wanting to
maintain an initialization vector. Even though ECB can leack information about
plaintext blocks, the data is compressed with ZLIB before encrypted. This
increases the entropy of the plaintext, and removes duplicated blocks from the
plaintext, as is the design of compression algorithms.

Although I cannot enforce this, you should serve the application over SSL. The
plaintext must be transmitted between the browser and the server, before it can
be encrypted. The data is also decrypted on the server, and transmitted to the
recipient's browser. As such, you should protect the data from being sniffed
using SSL on this application.

When the note is destroyed, it is first overwritten once with random data,
then removed. There are a couple of benefits from this:

* If the underlying filesystem is copy on write, then new random data is
  written to new sectors on disk, making it difficult to know precisely where
  the encrypted file ended and where the random data started.
* If the underlying filesystem is a journaled filesystem, the journal may have
  logged entries of the data. But again, because it's compressed then
  encrypted, it will be very difficult to know when the encrypted data was
  stored and when it was overwritten with random data.

Client Side
-----------

There are many things I could do to greatly discourage the recipient from
copying the plaintext, but they're mostly just annoying. There's nothing to
stop the recipient from memorizing the note, taking a screenshot (or many), or
copying and pasting the text. If the recipient really wants the note saved that
badly, it will happen. As such, the only thing that can be done is destroying
the note after a given interval, to prevent it from being viewed again.

To prevent form spam, a JavaScript implementation of Hashcash is installed. The
client browser must solve a puzzle accurately before submitting the form. Upon
page load, a token is generated that uniquely identifies your browser with
about 94% accuracy using a JavaScript implementation of browser fingerprints,
as introduced by the EFF. This token is submitted to the server during form
submission. If the SHA1 hash of the token is valid, the form is sucessfully
submitted. Otherwise, an error is thrown.
