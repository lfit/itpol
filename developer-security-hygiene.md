# Free software developer security hygiene

Updated: 2017-12-01

### Target audience

This document is aimed at developers working on free software projects. It
covers the following topics:

1. Basic introduction to PGP and Git
2. PGP key best practices
3. Basic workstation security

We use the term "Free" as in "Freedom," but this guide can also be used for
developing non-free or source-available ("Open Source") software. If you write
code that goes into public source repositories, you can benefit from getting
acquainted with and following this guide.

#### Topics NOT covered

This is not a "how to write secure software" guide. Please check the resources
on secure coding best practices that are available for the programming
languages, libraries, and development environments used by your free software
project.

### Structure

Each section is split into two areas:

- The checklist that can be adapted to your project's needs
- Free-form list of considerations that explain what dictated these decisions

#### Checklist priority levels

The items in each checklist include the priority level, which we hope will
help guide your decision:

- _(ESSENTIAL)_ items should definitely be high on the consideration list.
  If not implemented, they will introduce high risks to the code that gets
  committed to the open-source project.
- _(NICE)_ to have items will improve the overall security, but will affect how
  you interact with your work environment, and probably require learning new
  habits or unlearning old ones.
- _(PARANOID)_ is reserved for items we feel will significantly improve your
  security, but will require making equally significant adjustments to the way
  you interact with your operating system.

Remember, these are only guidelines. If you feel these priority levels do not
reflect your project's commitment to security, you should adjust them as you
see fit.

## PGP and  Free Software development

The Free Software community has long relied on PGP for assuring the
authenticity and integrity of software products it produced. You may not be
aware of it, but whether you are a Linux, Mac or Windows user, you have
previously relied on PGP to ensure the integrity of your computing
environment:

- Linux distributions rely on PGP to ensure that binary or source packages have
  not been altered between when they have been produced and when they are
  installed by the end-user.
- Free Software projects usually provide detached PGP signatures to accompany
  released software archives, so that downstream projects can verify the
  integrity of downloaded releases before integrating them into their own
  distributed downloads.

This is very similar to developer certificates/code signing mechanisms used by
programmers working on proprietary platforms. In fact, the core concepts
behind these two technologies are very much the same -- they differ mostly in
the technical aspects of the implementation and the way they delegate trust.
PGP does not rely on centralized Certification Authorities, but instead lets
each user assign their own trust to each certificate.

### Extremely Basic Overview of PGP

You do not need to know the exact details of how PGP works -- understanding
the core concepts is enough to be able to use it successfully. PGP relies on
Public Key Cryptography to convert plain text into encrypted text. This
process requires two distinct keys:

- A public key that is known to everyone
- A private key that is only known to the owner

#### Encryption

For encryption, PGP uses the public key of the owner to create a message that
is only decryptable using the owner's private key:

1. the sender generates a random encryption key ("session key")
2. the sender encrypts the contents using the session key
3. the sender encrypts the session key using the recipient's _public_ PGP key
4. the sender sends both the encrypted contents and the encrypted session key
   to the recipient

To decrypt:

1. the recipient decrypts the session key using their _private_ PGP key
2. the recipient uses the session key to decrypt the contents of the message

#### Signatures

For creating signatures, the private/public PGP keys are used the opposite way:

1. the signer generates the checksum hash of the contents
2. the signer uses their own _private_ PGP key to encrypt that checksum
3. the signer provides the encrypted checksum alongside the contents

To verify the signature:

1. the verifier generates their own checksum hash of the contents
2. the verifier uses the signer's _public_ PGP key to decrypt the provided
   checksum
3. if the checksums match, the integrity of the contents is verified

#### Combined usage

Frequently, encrypted messages are also signed with the sender's own PGP key.
This should be the default whenever using encrypted messaging, as encryption
without authentication is not very meaningful (unless you are a whistleblower
or a secret agent).
