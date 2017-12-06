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
you should install [GPG-Suite](https://gpgtools.org) or you can use `brew
install gnupg2`. For all other platforms, you'll need to do your own research
to find the correct places to download and install GnuPG.

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

##### Making sure you always use GnuPG2

If you have both `gpg` and `gpg2` commands, you should make sure you are
always using GnuPG v2, not the legacy version. You can make sure of it by
setting the alias:

    alias gpg='/usr/bin/gpg2'

You can put that in your `.bashrc` to make sure it's always loaded whenever
you use the gpg commands.

## Generating and protecting your master PGP key

### Checklist

- [ ] Generate the 4096-bit RSA master key _(ESSENTIAL)_
- [ ] Back up the master key using paperkey _(ESSENTIAL)_
- [ ] Add all relevant identities _(ESSENTIAL)_

### Considerations

#### Understanding the "Master" (Certify) key

In this and next section we'll talk about the "master key" and "subkeys". It
is important to understand the following:

1. There are no technical differences between the "master key" and "subkeys."
2. At creation time, we assign functional limitations to each key by
   giving it specific capabilities.
3. A PGP key can have 4 capabilities.
   - **[S]** Key can be used for signing
   - **[E]** Key can be used for encryption
   - **[A]** Key can be used for authentication
   - **[C]** Key can be used for certifying other keys
4. A single key may have multiple capabilities.

The key carrying the **[C]** (certify) capability is considered the "master"
key because it is the only key that can be used to indicate relationship with
other keys. Only the **[C]** key can be used to:

- add or revoke other keys (subkeys) with S/E/A capabilities
- add, change or revoke identities (uids) associated with the key
- add or change the expiration date on itself or any subkey
- sign other people's keys for the web of trust purposes

In the Free Software world, the **[C]** key is your digital identity. Once you
create the key, you should take extra care to protect it and prevent it from
falling into malicious hands.

#### Before you create the master key

Before you create your master key you need to pick your primary identity and
your master passphrase.

##### Primary identity

An identity is basically in the same format as the From field in emails:

    Alice Engineer <alice.engineer@example.org>

You can create new identities and revoke old ones, and you can also change
which identity is your "primary" one at a later time. Since the primary
identity is shown in all GnuPG operations, you should pick an
address/description that is both professional and the most likely one to be
used for PGP-enforced communication, such as your work address or the address
you use for signing off on project commits.

##### Passphrase

The passphrase is used exclusively for encrypting the private key with a
symmetric algorithm while it is stored on disk. If the contents of your
`.gnupg` directory ever get leaked, a good passphrase is the last line of
defense between the thief and them being able to impersonate you online, which
is why it is important to set up a good passphrase.

A good guideline for a strong passphrase is 3-4 words from a rich or mixed
dictionary that are not quotes from popular sources (songs, books, slogans).
You won't need to type the **[C]** key passphrase very frequently, so it does
not need to be easy to type, just easy to remember.

##### Algorithm and key strength

Even though GnuPG has supported Elliptic Curve crypto for a while now, we'll be
sticking to RSA keys, at least for a little while longer. While it is possible
to start using ED25519 keys right now, it is possible that you will come
across tools and hardware devices that will not be able to handle them
correctly.

For this reason, we will be generating RSA keys. For our master key, we'll use
4096 bits, and for our subkeys we'll stick to 2048 bits -- it is easy enough
to replace subkeys with stronger ones, but the master key must live on for a
long time.

#### Generate the master key

To generate your key, issue the following command, putting in the right values
instead of Alice Engineer:

    gpg --quick-generate-key 'Alice Engineer <alice@example.org>' rsa4096 cert

A dialog will pop up asking to enter the passphrase. Then, you may need to
move your mouse around or type on some keys to generate enough entropy until
the command completes.

Review the output of the command, it will be something like this:

    pub   rsa4096 2017-12-06 [C] [expires: 2019-12-06]
          111122223333444455556666AAAABBBBCCCCDDDD
    uid                      Alice Engineer <alice@example.org>

Note the long string on the 2nd line -- that is the full fingerprint of your
newly generated key. Key ID can be represented in three different forms:

- **fingerprint**, a full 40-character key identifier
- **long**, last 16-characters of the fingerprint (`AAAABBBBCCCCDDDD`)
- **short**, last 8 characters of the fingerprint (`CCCCDDDD`)

You should avoid using 8-character "short key IDs" as they are not
sufficiently unique.

At this point, I suggest you open a text editor, copy the fingerprint of your
new key and paste it there. You'll need to use it for the next few steps.

#### Back up your master key

For disaster recovery purposes -- and especially if you intend to use the Web
of Trust and collect key signatures from other project developers -- you
should create a hardcopy backup of your private key. This is supposed to be a
"last resort" measure in case all other backup mechanisms have failed.

The best way to create a printable hardcopy of your private key is using
`paperkey` software written for this very purpose. Paperkey is available on
all Linux distros, as well installable via `brew install paperkey` on Macs.

Run the following command, replacing `[fpr]` with the full fingerprint of your
key:

    gpg --export-secret-key [fpr] | paperkey > /tmp/key-backup.txt

The output will be in a format that is easy to OCR or input by hand, should
you ever need to recover it. Print out that file, then take a pen and write
the key passphrase on the margin of the paper. This is a required step because
the key printout is still encrypted with the passphrase, and if you ever
change the passphrase on your key, you will not remember what it used to be
when you had first created it -- guaranteed.

Put the resulting printout and the hand-written passphrase into an envelope
and store in a secure and well-protected place that is away from your home,
such as your bank vault.

**NOTE ON PRINTERS**: Long gone are days when printers were dumb devices
connected to the computer's parallel port. These days they have full operating
systems, hard drives, and cloud integration. Since the key content we send to
the printer will be encrypted with the passphrase, this is a fairly safe
operation, but use your best paranoid judgement.

#### Add relevant identities

If you have multiple relevant email addresses (personal, work, open-source
project, etc), you should add them to your master key. You don't need to do
this for any addresses that you don't expect to use with PGP (e.g. probably
not your school alumni address).

The command is (put the full key fingerprint instead of `[fpr]`):

    gpg --quick-add-uid [fpr] 'Alice Engineer <allie@example.net>'

You can review the IDs you've already added using:

    gpg --list-key [fpr] | grep ^uid

##### Pick the primary UID

GnuPG will make the latest UID you add as your primary UID, so if that is
different from what you want, you should fix it back:

    gpg --quick-set-primary-uid [fpr] 'Alice Engineer <alice@example.org>'

## Generating PGP subkeys

### Checklist

- [ ] Generate a 2048-bit Encryption key _(ESSENTIAL)_
- [ ] Generate a 2048-bit Signing key _(ESSENTIAL)_
- [ ] Generate a 2048-bit Authentication key _(NICE)_

### Considerations

Now that we've created the master key, let's create the keys you'll actually
be using for day-to-day work. We create 2048-bit keys because a lot of
specialized hardware (we'll discuss this further) does not handle larger keys,
but also for pragmatic reasons. If we ever find ourselves in a world where
2048-bit RSA keys are not considered good enough, it will be because of
fundamental problems with the RSA protocol and longer 4096-bit keys will not
make much difference.

#### Create the Sign and Encrypt subkeys

To create the subkeys, run:

    gpg --quick-add-key [fpr] rsa2048 encr
    gpg --quick-add-key [fpr] rsa2048 sign

You can also create the Authentication key, which will allow you to use your
PGP key for ssh purposes (covered in other guides):

    gpg --quick-add-key [fpr] rsa2048 auth

You can review your key information using `gpg --list-key [fpr]`:

    pub   rsa4096 2017-12-06 [C] [expires: 2019-12-06]
          111122223333444455556666AAAABBBBCCCCDDDD
    uid           [ultimate] Alice Engineer <alice@example.org>
    uid           [ultimate] Alice Engineer <allie@example.net>
    sub   rsa2048 2017-12-06 [E]
    sub   rsa2048 2017-12-06 [S]

## Moving your master key to offline storage

### Checklist

- [ ] Prepare encrypted detachable storage _(ESSENTIAL)_
- [ ] Back up your GnuPG directory _(ESSENTIAL)_
- [ ] Remove the master key from your home directory _(NICE)_

#### Back up your GnuPG directory

It is important to have a readily available backup of your PGP keys should you
need to recover them (this is different from the disaster-level preparedness
we did with `paperkey`).

**This step is especially important if you are going to remove your master key
or use smartcard hardware. Do not skip this step!**

#### Prepare detachable encrypted storage

Start by getting a detachable USB drive (preferably two) that you will use for
backup purposes. They do not need to be large. You will first need to encrypt
them:

- [Apple instructions](https://support.apple.com/kb/PH25745)
- [Linux instructions](https://help.ubuntu.com/community/EncryptedFilesystemsOnRemovableStorage)

For the encryption passphrase, you can use the same one as on your master key.

#### Back up your GnuPG directory

Once the encryption process is over, re-insert the USB drive and make sure it
gets properly mounted. Find out the full mount point of the device, for
example by running the `mount` command (under Linux, external media usually
gets mounted under `/media/disk`, under Mac it's `/Volumes`).

Once you know the full mount path, copy your entire GnuPG directory there:

    cp -rp $HOME/.gnupg [/media/disk/some/path]/gnupg-backup

You should now test to make sure it still works:

    gpg --homedir=[/media/disk/some/path]/gnupg-backup --list-key [fpr]

If you don't get any errors, then you should be good to go. Unmount the USB
drive, label it accordingly so you don't blow it away next time you need to
use a quick USB drive, and put in a safe place.
