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

## Basic PGP concepts and tools

### Checklist

- [ ] Understand the role of PGP in Free Software Development _(ESSENTIAL)_
- [ ] Understand the basics of Public Key Cryptography _(ESSENTIAL)_
- [ ] Understand PGP Encryption vs. Signatures _(ESSENTIAL)_
- [ ] Understand PGP key identities _(ESSENTIAL)_
- [ ] Understand PGP key validity _(ESSENTIAL)_
- [ ] Install GnuPG utilities (version 2.x) _(ESSENTIAL)_

### Considerations

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
- Free Software projects routinely rely on PGP signatures within the code
  itself in order to track provenance and verify integrity of code commits
  by project developers.

This is very similar to developer certificates/code signing mechanisms used by
programmers working on proprietary platforms. In fact, the core concepts
behind these two technologies are very much the same -- they differ mostly in
the technical aspects of the implementation and the way they delegate trust.
PGP does not rely on centralized Certification Authorities, but instead lets
each user assign their own trust to each certificate.

Our goal is to get your project on board using PGP for code provenance and
integrity tracking, following best practices and observing basic security
precautions.

### Extremely Basic Overview of PGP operations

You do not need to know the exact details of how PGP works -- understanding
the core concepts is enough to be able to use it successfully for our
purposes. PGP relies on Public Key Cryptography to convert plain text into
encrypted text. This process requires two distinct keys:

- A public key that is _known to everyone_
- A private key that is _only known to the owner_

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

### Understanding Key Identities

Each PGP key must have one or multiple Identities associated with it. Usually,
an "Identity" is the person's full name and email address in the following
format:

    Alice Engineer <alice.engineer@example.com>

Sometimes it will also contain a comment in brackets, to tell the end-user
more about that particular key:

    Bob Designer (obsolete 1024-bit key) <bob.designer@example.com>

Since people can be associated with multiple professional and personal
entities, they can have multiple identities on the same key:

    Alice Engineer <alice.engineer@example.com>
    Alice Engineer <aengineer@personalmail.example.org>
    Alice Engineer <webmaster@girlswhocode.example.net>

When multiple identities are used, one of them would be marked as the "primary
identity" to make searching easier.

### Understanding Key Validity

To be able to use someone else's public key for encryption or verification,
you need to be sure that it actually belongs to the right person (Alice) and
not to an impostor (Eve). In PGP, this certainty is called "key validity:"

- **Validity: full** -- means we are pretty sure this key belongs to Alice
- **Validity: marginal** -- means we are *somewhat* sure this key belongs to
  Alice
- **Validity: uknown** -- means there is no assurance at all that this key
  belongs to Alice

#### Web of Trust (WoT) vs. Trust on First Use (TOFU)

PGP uses a trust delegation mechanism known as the "Web of Trust." At its
core, this is an attempt to replace the need for centralized Certification
Authorities of the HTTPS/TLS world. Instead of various software makers
dictating who should be your trusted certification authorities, PGP leaves
this responsibility to each user.

Unfortunately, very few people understand how the Web of Trust works, and even
fewer bother to keep it going. It remains an important aspect of the OpenPGP
specification, but recent versions of GnuPG (2.2 and above) have implemented
an alternative mechanism called "Trust on First Use" (TOFU).

You can think of TOFU as "the SSH-like approach to trust." With SSH, the first
time you connect to a remote system, its key fingerprint is recorded and
remembered. If the key changes in the future, the SSH client will alert you
and refuse to connect, forcing you to make a decision on whether you choose to
trust the changed key or not.

Similarly, the first time you import someone's PGP key, it is assumed to be
trusted. If at any point in the future GnuPG comes across another key with the
same identity, both the previously imported key and the new key will be marked
as invalid and you will need to manually figure out which one to trust.

In this guide, we will be using the TOFU trust model.

### Installing OpenPGP software

First, it is important to understand the distinction between PGP, OpenPGP,
GnuPG and gpg:

- **PGP** ("Pretty Good Privacy") is the name of the original commercial software
- **OpenPGP** is the IETF standard compatible with the original PGP tool
- **GnuPG** ("Gnu Privacy Guard") is free software that implements the OpenPGP
  standard
- The command-line tool for GnuPG is called "**gpg**"

Today, the term "PGP" is almost always used to mean "the OpenPGP standard,"
not the original commercial software, and therefore "PGP" and "OpenPGP" are
interchangeable. The terms "GnuPG" and "gpg" should only be used when
referring to the tools, not to the output they produce or OpenPGP features
they implement. For example:

- PGP (not GnuPG or GPG) key
- PGP (not GnuPG or GPG) signature
- PGP (not GnuPG or GPG) keyserver

Understanding this should protect you from an inevitable pedantic "actually"
from other PGP users you come across.

#### Installing GnuPG

If you are using Linux, you should already have GnuPG installed.  On a Mac,
you should install [GPG-Suite](https://gpgtools.org). For all other platforms,
you'll need to do your own research to find the correct places to download and
install GnuPG.

##### GnuPG 1 vs. 2

Both GnuPG v.1 and GnuPG v.2 implement the same standard, but they provide
incompatible libraries and command-line tools, so many distributions ship both
the legacy version 1 and the latest version 2. You need to make sure you are
always using GnuPG v.2.

First, run:

    gpg --version | head -1

If you see `gpg (GnuPG) 1.4.x`, then you are using GnuPG v.1. Try the `gpg2`
command:

    gpg2 --version | head -1

If you see `gpg (GnuPG) 2.x.x`, then you are good to go. This guide will
assume you have the version 2.2 of GnuPG (or later).
