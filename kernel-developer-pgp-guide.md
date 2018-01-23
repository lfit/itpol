# Kernel developer PGP guide

Updated: 2018-01-22

*Status: CURRENT, BETA*

### Target audience

This document is aimed at Linux kernel developers, and especially subsystem
maintainers. It contains a subset of information discussed in the more
general "Protecting Code Integrity" guide found in the same repository. If you
are not a Linux kernel developer, you should read the more general guide
instead.

This document covers the following topics:

1. How to improve your PGP key security
2. When and how to use PGP with git
3. How to properly use the Web of Trust

### Structure

Each section is split into two areas:

- A checklist of actionable items
- Free-form list of considerations that explain what dictated these decisions,
  together with configuration instructions

#### Checklist priority levels

The items in each checklist include the priority level, which we hope will
help guide your decision:

- _(ESSENTIAL)_ items should definitely be high on the consideration list.
  If not implemented, they will introduce high risks to the code that gets
  committed to the kernel.
- _(NICE)_ to have items will improve the overall security, but will affect how
  you interact with your work environment, and probably require learning new
  habits or unlearning old ones.

Remember, these are only guidelines. If you feel these priority levels do not
reflect your commitment to security, you should adjust them as you see fit.

## The role of PGP in Linux Kernel development

PGP helps ensure the integrity of the code that is produced by the Linux
Kernel development community and, to a lesser degree, establish trusted
communication channels between members of the Linux Kernel development
community.

The Linux Kernel source code is available in two main formats:

- Distributed source repositories (git)
- Periodic release snapshots (tar)

Both git repositories and tarballs carry PGP signatures of the kernel
developers who are tasked with making official kernel releases. These
signatures offer a cryptographic guarantee that downloadable versions made
available via kernel.org or on its multiple worldwide mirrors are identical to
what the developers have on their workstations. To this end:

- git repositories provide PGP signatures on all tags
- tarballs provide detached PGP signatures as separate downloads

### Trusting the developers, not infrastructure

Ever since the 2011 compromise of core kernel.org systems, the main operating
principle of the Kernel Archives project has been to assume that any part of
the infrastructure can be compromised at any time. For this reason, the
administrators have taken deliberate steps to emphasize that trust must always
be placed with the developers and never with code hosting infrastructure,
regardless of how good the security practices for the latter may be.

This guiding principle is the reason why this guide is needed. We want to make
sure that by placing trust into developers we do not simply shift the blame
for future security incidents to someone else. The goal is to provide a set of
guidelines developers can use to create a secure development environment and
safeguard the very PGP keys used to establish the integrity of the Linux
Kernel itself.

## PGP tools

### Checklist

- [ ] Configure GnuPG to always use version 2 _(ESSENTIAL)_
- [ ] Configure gpg-agent options _(ESSENTIAL)_
- [ ] Set up a refresh cronjob _(ESSENTIAL)_

### Considerations

### Installing GnuPG

Your distributions should already have GnuPG installed, unless they are doing
something horribly wrong. We just need to verify that you are using version
2.x and not the legacy 1.4 release. Unfortunately, most distributions still
package both versions, with the default `gpg` command being from GnuPG v.1.
To check, run:

    $ gpg --version

If you see `gpg (GnuPG) 1.4.x`, then you are using GnuPG v.1. Try the `gpg2`
command (if you don't have it, you may need to install the gnupg2 package):

    $ gpg2 --version

If you see `gpg (GnuPG) 2.x.x`, then you are good to go. This guide will
assume you have the version 2.2 of GnuPG (or later). If you are using version
2.0 of GnuPG, some of the commands in this guide will not work, and you should
consider installing the latest 2.2 version of GnuPG. Most recent versions of
gnupg-2.1 should be compatible for the purposes of this guide.

##### Making sure you always use GnuPG v.2

If you have both `gpg` and `gpg2` commands, you should make sure you are
always using GnuPG v2, not the legacy version. You can make sure of this by
setting the alias:

    $ alias gpg=gpg2

You can put that in your `.bashrc` to make sure it's always loaded whenever
you use the gpg commands.

#### Configure gpg-agent options

The GnuPG agent is a helper tool that will start automatically whenever you
use the `gpg` command and run in the background with the purpose of caching
the private key passphrase. It is no longer necessary to start it manually
at the beginning of your shell session.

There are two options you should know in order to tweak when the passphrase
should be expired from cache:

- `default-cache-ttl` (seconds): If you use the same key again before the
  time-to-live expires, the countdown will reset for another period.
  The default is 600 (10 minutes).
- `max-cache-ttl` (seconds): Regardless of how recently you've used the key
  since initial passphrase entry, if the maximum time-to-live countdown
  expires, you'll have to enter the passphrase again. The default is 30
  minutes.

If you find either of these defaults too short (or too long), you can edit
your `~/.gnupg/gpg-agent.conf` file to set your own values:

    # set to 30 minutes for regular ttl, and 2 hours for max ttl
    default-cache-ttl 1800
    max-cache-ttl 7200

#### Set up a refresh cronjob

You will need to regularly refresh your keyring in order to get the latest
changes on other people's public keys. You can set up a cronjob to do that:

    $ crontab -e

Add the following on a new line:

    @daily /usr/bin/gpg2 --refresh >/dev/null 2>&1

**NOTE**: check the full path to your `gpg` or `gpg2` command and use the `gpg2`
command if regular `gpg` for you is the legacy GnuPG v.1.

## Protecting your master PGP key

### Checklist

- [ ] Understand the "master" key vs. subkeys _(ESSENTIAL)_
- [ ] Ensure your private key passphrase is strong _(ESSENTIAL)_
- [ ] Create a separate **[S]** subkey _(ESSENTIAL)_
- [ ] Back up the master key using paperkey _(ESSENTIAL)_
- [ ] Back up your whole `.gnupg` directory to encrypted media _(ESSENTIAL)_

### Considerations

This guide assumes that you already have a PGP key that you use for Linux
Kernel development purposes. If you do not yet have one, please see the
"Protecting Code Integrity" document in this repository for guidance on how to
create one.

You should make a new key if your current one is weaker than 2048 bits.

#### Understanding the "Master" (Certify) key

In this and next section we'll talk about the "master key" and "subkeys". It
is important to understand the following:

1. There are no technical differences between the "master key" and "subkeys."
2. At creation time, we assign functional limitations to each key by
   giving it specific capabilities.
3. A PGP key can have 4 capabilities.
   - **[S]** key can be used for signing
   - **[E]** key can be used for encryption
   - **[A]** key can be used for authentication
   - **[C]** key can be used for certifying other keys
4. A single key may have multiple capabilities.

The key carrying the **[C]** (certify) capability is considered the "master"
key because it is the only key that can be used to indicate relationship with
other keys. Only the **[C]** key can be used to:

- add or revoke other keys (subkeys) with S/E/A capabilities
- add, change or revoke identities (uids) associated with the key
- add or change the expiration date on itself or any subkey
- sign other people's keys for the web of trust purposes

By default, GnuPG creates the following when generating new keys:

- A master key carrying both Certify and Sign capabilities (**[SC]**)
- A single subkey with the Encryption capability (**[E]**)

If you used default parameters when generating your key, that is what you will
have. You can verify by running `gpg --list-secret-keys`:

    sec   rsa2048 2018-01-23 [SC] [expires: 2020-01-23]
          000000000000000000000000AAAABBBBCCCCDDDD
    uid           [ultimate] Ada Dev <adev@kernel.org>
    ssb   rsa2048 2018-01-23 [E] [expires: 2020-01-23]

Any key carrying the **[C]** capability is your master key, regardless of any
other capabilities it may have.

#### Ensure your passphrase is strong

GnuPG uses passphrases to encrypt your private keys before storing them on
disk. This way, even if your `.gnupg` directory is leaked or stolen in its
entirety, the attackers cannot use your private keys without first obtaining
the passphrase to decrypt them.

It is absolutely essential that your private keys are protected by a
strong passphrase. To set it or change it, use:

    $ gpg --change-passphrase [fpr]

#### Create a separate Signing subkey

Our goal is to protect your master key by moving it to offline media, so
if you only have a combined **[SC]** key, then you should create a separate
signing subkey.

##### RSA vs. ECC subkeys

GnuPG v2 has full support for Elliptic Curve Cryptography, with ability to
combine ECC subkeys with traditional RSA master keys. The main upside of ECC
cryptography is that it is much faster computationally and creates much
smaller signatures when comparing byte for byte with 2048+ RSA keys.

Unless you plan on using a smartcard device that does not support ECC crypto,
we recommend that you create an ECC signing subkey for your kernel work:

    $ gpg --quick-add-key [fpr] ed25519 sign

If for some reason you prefer to stay with RSA subkeys, just replace "ed25519"
with "rsa2048" in the above command.

#### Back up your private keys

The more signatures you have on your PGP key from other developers, the more
reasons you have to create a backup version that lives on something other than
digital media, for disaster recovery reasons.

The best way to create a printable hardcopy of your private key is by using
the `paperkey` software written for this very purpose. See `man paperkey` for
more details on the output format and its benefits over other solutions.
Paperkey should already be packaged for most distributions.

Run the following command, replacing `[fpr]` with the full fingerprint of your
key:

    $ gpg --export-secret-key [fpr] | paperkey > /tmp/key-backup.txt

Print out that file, then take a pen and write your passphrase on the margin
of the paper. **This is strongly recommended** because the key printout is
still encrypted with that passphrase, and if you ever change it you will not
remember what it used to be when you had created the backup -- *guaranteed*.

Put the resulting printout and the hand-written passphrase into an envelope
and store in a secure and well-protected place, preferably away from your
home, such as your bank vault.

**NOTE ON PRINTERS**: Your printer is probably no longer a simple dumb device
connected to your parallel port, but since the output is still encrypted with
your passphrase, printing out even to "cloud-integrated" modern printers
should be a relatively safe operation.



Up to here
------------------------------------------------------------------------------

## Generating PGP subkeys

### Checklist

- [ ] Generate a 2048-bit Encryption subkey _(ESSENTIAL)_
- [ ] Generate a 2048-bit Signing subkey _(ESSENTIAL)_
- [ ] Generate a 2048-bit Authentication subkey _(NICE)_
- [ ] Upload your public keys to a PGP keyserver _(ESSENTIAL)_
- [ ] Set up a refresh cronjob _(ESSENTIAL)_

### Considerations

Now that we've created the master key, let's create the keys you'll actually
be using for day-to-day work. We create 2048-bit keys because a lot of
specialized hardware (we'll discuss this further) does not handle larger keys,
but also for pragmatic reasons. If we ever find ourselves in a world where
2048-bit RSA keys are not considered good enough, it will be because of
fundamental breakthroughs in computing or mathematics and therefore longer
4096-bit keys will not make much difference.

#### Create the subkeys

To create the subkeys, run:

    $ gpg --quick-add-key [fpr] rsa2048 encr
    $ gpg --quick-add-key [fpr] rsa2048 sign

You can also create the Authentication key, which will allow you to use your
PGP key for ssh purposes:

    $ gpg --quick-add-key [fpr] rsa2048 auth

You can review your key information using `gpg --list-key [fpr]`:

    pub   rsa4096 2017-12-06 [C] [expires: 2019-12-06]
          111122223333444455556666AAAABBBBCCCCDDDD
    uid           [ultimate] Alice Engineer <alice@example.org>
    uid           [ultimate] Alice Engineer <allie@example.net>
    sub   rsa2048 2017-12-06 [E]
    sub   rsa2048 2017-12-06 [S]

#### Upload your public keys to the keyserver

Your key creation is complete, so now you need to make it easier for others to
find it by uploading it to one of the public keyservers. (Do not do this step
if you're just messing around and aren't planning on actually using the key
you've created, as this just litters keyservers with useless data.)

    $ gpg --send-key [fpr]

If this command does not succeed, you can try specifying the keyserver on a
port that is most likely to work:

    $ gpg --keyserver hkp://pgp.mit.edu:80 --send-key [fpr]

Most keyservers communicate with each-other, so your key information will
eventually synchronize to all the others.

**NOTE ON PRIVACY:** Keyservers are completely public and therefore, by
design, leak potentially sensitive information about you, such as your full
name, nicknames, and personal or work email addresses. If you sign other
people's keys or someone signs yours, keyservers will additionally become
leakers of your social connections. Once such personal information makes it to
the keyservers, it becomes impossible to edit or delete. Even if you revoke a
signature or identity, that does not delete them from your key record, just
marks them as revoked -- making them stand out even more.

That said, if you participate in software development on a public project, all
of the above information is already public record, and therefore making it
additionally available via keyservers does not result in a net loss in
privacy.

##### Upload your public key to GitHub

If you use GitHub in your development (and who doesn't?), you should upload
your key following the instructions they have provided:

- [Adding a PGP key to your GitHub account](https://help.github.com/articles/adding-a-new-gpg-key-to-your-github-account/)

To generate the public key output suitable to paste in, just run:

    $ gpg --export --armor [fpr]

#### Set up a refresh cronjob

You will need to regularly refresh your keyring in order to get the latest
changes on other people's public keys. You can set up a cronjob to do that:

    $ crontab -e

Add the following on a new line:

    @daily /usr/bin/gpg2 --refresh >/dev/null 2>&1

**NOTE**: check the full path to your `gpg` or `gpg2` command and use the `gpg2`
command if regular `gpg` for you is the legacy GnuPG v.1.

## Moving your master key to offline storage

### Checklist

- [ ] Prepare encrypted detachable storage _(ESSENTIAL)_
- [ ] Back up your GnuPG directory _(ESSENTIAL)_
- [ ] Remove the master key from your home directory _(NICE)_
- [ ] Remove the revocation certificate from your home directory _(NICE)_

### Considerations

Why would you want to remove your master **[C]** key from your home directory?
This is generally done to prevent your master key from being stolen or
accidentally leaked. Private keys are tasty targets for malicious actors -- we
know this from several successful malware attacks that scanned users' home
directories and uploaded any private key content found there.

It would be very damaging for any developer to have their PGP keys stolen --
in the Free Software world this is often tantamount to identity theft.
Removing private keys from your home directory helps protect you from such
events.

#### Back up your GnuPG directory

**!!!Do not skip this step!!!**

It is important to have a readily available backup of your PGP keys should you
need to recover them (this is different from the disaster-level preparedness
we did with `paperkey`).

#### Prepare detachable encrypted storage

Start by getting a small USB "thumb" drive (preferably two!) that you will use
for backup purposes. You will first need to encrypt them:

- [Apple instructions](https://support.apple.com/kb/PH25745)
- [Linux instructions](https://help.ubuntu.com/community/EncryptedFilesystemsOnRemovableStorage)

For the encryption passphrase, you can use the same one as on your master key.

#### Back up your GnuPG directory

Once the encryption process is over, re-insert the USB drive and make sure it
gets properly mounted. Find out the full mount point of the device, for
example by running the `mount` command (under Linux, external media usually
gets mounted under `/media/disk`, under Mac it's `/Volumes`).

Once you know the full mount path, copy your entire GnuPG directory there:

    $ cp -rp ~/.gnupg [/media/disk/name]/gnupg-backup

(Note: If you get any `Operation not supported on socket` errors, those are
benign and you can ignore them.)

You should now test to make sure everything still works:

    $ gpg --homedir=[/media/disk/name]/gnupg-backup --list-key [fpr]

If you don't get any errors, then you should be good to go. Unmount the USB
drive, distinctly label it so you don't blow it away next time you need to use
a random USB drive, and put in a safe place -- but not too far away, because
you'll need to use it every now and again for things like editing identities,
adding or revoking subkeys, or signing other people's keys.

#### Remove the master key

Please see the previous section and make sure you have backed up your GnuPG
directory in its entirety. What we are about to do will render your key
useless if you do not have a usable backup!

First, identify the keygrip of your master key:

    $ gpg --with-keygrip --list-key [fpr]

The output will be something like this:

    pub   rsa4096 2017-12-06 [C] [expires: 2019-12-06]
          111122223333444455556666AAAABBBBCCCCDDDD
          Keygrip = AAAA999988887777666655554444333322221111
    uid           [ultimate] Alice Engineer <alice@example.org>
    uid           [ultimate] Alice Engineer <allie@example.net>
    sub   rsa2048 2017-12-06 [E]
          Keygrip = BBBB999988887777666655554444333322221111
    sub   rsa2048 2017-12-06 [S]
          Keygrip = CCCC999988887777666655554444333322221111

Find the keygrip entry that is beneath the `pub` line (right under the master
key fingerprint). This will correspond directly to a file in your home
`.gnupg` directory:

    $ cd ~/.gnupg/private-keys-v1.d
    $ ls
    AAAA999988887777666655554444333322221111.key
    BBBB999988887777666655554444333322221111.key
    CCCC999988887777666655554444333322221111.key

All you have to do is simply remove the `.key` file that corresponds to the
master keygrip:

    $ cd ~/.gnupg/private-keys-v1.d
    $ rm AAAA999988887777666655554444333322221111.key

Now, if you issue the `--list-secret-keys` command, it will show that the
master key is missing (the `#` indicates it is not available):

    $ gpg --list-secret-keys
    sec#  rsa4096 2017-12-06 [C] [expires: 2019-12-06]
          111122223333444455556666AAAABBBBCCCCDDDD
    uid           [ultimate] Alice Engineer <alice@example.org>
    uid           [ultimate] Alice Engineer <allie@example.net>
    ssb   rsa2048 2017-12-06 [E]
    ssb   rsa2048 2017-12-06 [S]

#### Remove the revocation certificate

Another file you should remove (but keep in backups) is the revocation
certificate that was automatically created with your master key. A revocation
certificate allows someone to permanently mark your key as revoked, meaning it
can no longer be used or trusted for any purpose. You would normally use it to
revoke a key that, for some reason, you can no longer control -- for example,
if you had lost the key passphrase.

Just as with the master key, if a revocation certificate leaks into malicious
hands, it can be used to destroy your developer digital identity, so it's
better to remove it from your home directory.

    cd ~/.gnupg/openpgp-revocs.d
    rm [fpr].rev

## Move the subkeys to a hardware device

### Checklist

- [ ] Get a GnuPG-compatible hardware device _(NICE)_
- [ ] Configure the device to work with GnuPG _(NICE)_
- [ ] Set the user and admin PINs _(NICE)_
- [ ] Move your subkeys to the device _(NICE)_

### Considerations

Even though the master key is now safe from being leaked or stolen, the
subkeys are still in the home directory. Anyone who manages to get their hands
on those will be able to decrypt your communication or fake your signatures
(if they know the passphrase, that is).

The best way to completely protect your keys is to move them to a specialized
hardware device that is capable of smartcard operations.

#### The benefits of smartcards

A smartcard contains a cryptographic chip that is capable of storing private
keys and performing crypto operations directly on the card itself. Because the
key contents never leave the smartcard, the operating system of the computer
into which you plug in the hardware device is not able to retrieve the
private keys themselves. This is very different from the encrypted USB storage
device we used earlier for backup purposes -- while that USB device is plugged
in and decrypted, the operating system is still able to access the private key
contents. Using external encrypted USB media is not a substitute to having a
smartcard-capable device.

Some other benefits of smartcards:

- they are relatively cheap and easy to obtain
- they are small and easy to carry with you
- they can be used with multiple devices
- many of them are tamper-resistant (depends on manufacturer)

#### Available smartcard devices

Smartcards started out embedded into actual wallet-sized cards, which earned
them their name. You can still buy and use GnuPG-capable smartcards, and they
remain one of the cheapest available devices you can get. However, actual
smartcards have one important downside: they require a smartcard reader, and
very few laptops come with one.

For this reason, manufacturers have started providing small USB devices, the
size of a USB thumb drive or smaller, that either have the microsim-sized
smartcard pre-inserted, or that simply implement the smartcard protocol
features on the internal chip. Here are a few recommendations:

- [Nitrokey Start](https://shop.nitrokey.com/shop/product/nitrokey-start-6):
  Open hardware and Free Software: one of the cheapest options for GnuPG use,
  but with fewest extra security features
- [Nitrokey Pro](https://shop.nitrokey.com/shop/product/nitrokey-pro-3):
  Similar to the Nitrokey Start, but is tamper-resistant and offers more
  security features (but not U2F, see the Fido U2F section of the guide)
- [Yubikey 4](https://www.yubico.com/product/yubikey-4-series/): Proprietary
  hardware and software, but cheaper than Nitrokey Pro and comes available
  in the USB-C form that is more useful with newer laptops; also offers
  additional security features such as U2F

Our recommendation is to pick a device that is capable of both smartcard
functionality and U2F, which, at the time of writing, means a Yubikey 4.

#### Configuring your smartcard device

Your smartcard device should Just Work (TM) the moment you plug it into any
modern Linux or Mac workstation. You can verify it by running:

    $ gpg --card-status

If you didn't get an error, but a full listing of the card details, then you
are good to go. Unfortunately, troubleshooting all possible reasons why things
may not be working for you is way beyond the scope of this guide. If you are
having trouble getting the card to work with GnuPG, please seek support via
your operating system's usual support channels.

##### PINs don't have to be numbers

Note, that despite having the name "PIN" (and implying that it must be a
"number"), neither the user PIN nor the admin PIN on the card need to be
numbers.

Your device will probably have default user and admin PINs set up when it
arrives. For Yubikeys, these are `123456` and `12345678` respectively. If
those don't work for you, please check any accompanying documentation
that came with your device.

##### Quick setup

To configure your smartcard, you will need to use the GnuPG menu system, as
there are no convenient command-line switches:

    $ gpg --card-edit
    [...omitted...]
    gpg/card> admin
    Admin commands are allowed
    gpg/card> passwd

You should set the user PIN (1), Admin PIN (3), and the Reset Code (4). Please
make sure to record and store these in a safe place -- especially the Admin
PIN and the Reset Code (which allows you to completely wipe the smartcard).
You so rarely need to use the Admin PIN, that you will inevitably forget what
it is if you do not record it.

Getting back to the main card menu, you can also set other values (such as
name, sex, login data, etc), but it's not necessary and will additionally leak
information about your smartcard should you lose it.

#### Moving the subkeys to your smartcard

Exit the card menu (using "q") and save all changes. Next, let's move your
subkeys onto the smartcard. You will need both your PGP key passphrase and the
admin PIN of the card for most operations. Remember, that `[fpr]` stands for
the full 40-character fingerprint of your key.

    $ gpg --edit-key [fpr]

    Secret subkeys are available.

    pub  rsa4096/AAAABBBBCCCCDDDD
         created: 2017-12-07  expires: 2019-12-07  usage: C
         trust: ultimate      validity: ultimate
    ssb  rsa2048/1111222233334444
         created: 2017-12-07  expires: never       usage: E
    ssb  rsa2048/5555666677778888
         created: 2017-12-07  expires: never       usage: S
    [ultimate] (1). Alice Engineer <alice@example.org>
    [ultimate] (2)  Alice Engineer <allie@example.net>

    gpg>

Using `--edit-key` puts us into the menu mode again, and you will notice that
the key listing is a little different. From here on, all commands are done
from inside this menu mode, as indicated by `gpg>`.

First, let's select the key we'll be putting onto the card -- you do this by
typing `key 1` (it's the first one in the listing, our **[E]** subkey):

    gpg> key 1

The output should be subtly different:

    pub  rsa4096/AAAABBBBCCCCDDDD
         created: 2017-12-07  expires: 2019-12-07  usage: C
         trust: ultimate      validity: ultimate
    ssb* rsa2048/1111222233334444
         created: 2017-12-07  expires: never       usage: E
    ssb  rsa2048/5555666677778888
         created: 2017-12-07  expires: never       usage: S
    [ultimate] (1). Alice Engineer <alice@example.org>
    [ultimate] (2)  Alice Engineer <allie@example.net>

Notice the `*` that is next to the `ssb` line corresponding to the key -- it
indicates that the key is currently "selected." It works as a toggle, meaning
that if you type `key 1` again, the `*` will disappear and the key will not be
selected any more.

Now, let's move that key onto the smartcard:

    gpg> keytocard
    Please select where to store the key:
       (2) Encryption key
    Your selection? 2

Since it's our **[E]** key, it makes sense to put it into the Encryption slot.
When you submit your selection, you will be prompted first for your PGP key
passphrase, and then for the admin PIN. If the command returns without an
error, your key has been moved.

**Important**: Now type `key 1` again to unselect the first key, and `key 2`
to select the **[S]** key:

    gpg> key 1
    gpg> key 2
    gpg> keytocard
    Please select where to store the key:
       (1) Signature key
       (3) Authentication key
    Your selection? 1

You can use the **[S]** key both for Signature and Authentication, but we want
to make sure it's in the Signature slot, so choose (1). Once again, if your
command returns without an error, then the operation was successful.

Finally, if you created an **[A]** key, you can move it to the card as well,
making sure first to unselect `key 2`. Once you're done, choose "q":

    gpg> q
    Save changes? (y/N) y

Saving the changes will delete the keys you moved to the card from your home
directory (but it's okay, because we have them in our backups should we need
to do this again for a replacement smartcard).

##### Verifying that the keys were moved

If you perform `--list-secret-keys` now, you will see a subtle difference in
the output:

    $ gpg --list-secret-keys
    sec#  rsa4096 2017-12-06 [C] [expires: 2019-12-06]
          111122223333444455556666AAAABBBBCCCCDDDD
    uid           [ultimate] Alice Engineer <alice@example.org>
    uid           [ultimate] Alice Engineer <allie@example.net>
    ssb>  rsa2048 2017-12-06 [E]
    ssb>  rsa2048 2017-12-06 [S]

The `>` in the `ssb>` output indicates that the subkey is only available on
the smartcard. If you go back into your secret keys directory and look at the
contents there, you will notice that the `.key` files there have been replaced
with stubs:

    $ cd ~/.gnupg/private-keys-v1.d
    $ strings *.key

The output should contain `shadowed-private-key` to indicate that these files
are only stubs and the actual content is on the smartcard.

#### Verifying that the smartcard is functioning

To verify that the smartcard is working as intended, you can create a
signature:

    $ echo "Hello world" | gpg --clearsign > /tmp/test.asc
    $ gpg --verify /tmp/test.asc

This should ask for your smartcard PIN on your first command, and then show
"Good signature" after you run `gpg --verify`.

Congratulations, you have successfully made it extremely difficult to steal
your digital developer identity!

### Other common GnuPG operations

Here is a quick reference for some common operations you'll need to do with
your PGP key.

In all of the below commands, the `[fpr]` is your key fingerprint.

#### Mounting your master key offline storage

You will need your master key for any of the operations below, so you will
first need to mount your backup offline storage and tell GnuPG to use it.
First, find out where the media got mounted, e.g. by looking at the output of
the `mount` command. Then, locate the directory with the backup of your GnuPG
directory and tell GnuPG to use that as its home:

    $ export GNUPGHOME=/media/disk/name/gnupg-backup
    $ gpg --list-secret-keys

You want to make sure that you see `sec` and not `sec#` in the output (the `#`
means the key is not available and you're still using your regular home
directory location).

##### Updating your regular GnuPG working directory

After you make any changes to your key using the offline storage, you will
want to import these changes back into your regular working directory:

    $ gpg --export | gpg --homedir ~/.gnupg --import
    $ unset GNUPGHOME

#### Extending key expiration date

The master key we created has the default expiration date of 2 years from the
date of creation. This is done both for security reasons and to make obsolete
keys eventually disappear from keyservers.

To extend the expiration on your key by a year from current date, just run:

    $ gpg --quick-set-expire [fpr] 1y

You can also use a specific date if that is easier to remember (e.g. your
birthday, January 1st, or Canada Day):

    $ gpg --quick-set-expire [fpr] 2020-07-01

Remember to send the updated key back to keyservers:

    $ gpg --send-key [fpr]

#### Revoking identities

If you need to revoke an identity (e.g. you changed employers and your old
email address is no longer valid), you can use a one-liner:

    $ gpg --quick-revoke-uid [fpr] 'Alice Engineer <aengineer@example.net>'

You can also do the same with the menu mode using `gpg --edit-key [fpr]`.

Once you are done, remember to send the updated key back to keyservers:

    $ gpg --send-key [fpr]

## Using PGP with Git

One of the core features of Git is its decentralized nature -- once a
repository is cloned to your system, you have full history of the project,
including all of its tags, commits and branches. However, with hundreds of
cloned repositories floating around, how does anyone verify that the
repository you downloaded has not been tampered with by a malicious third
party? You may have cloned it from GitHub or some other official-looking
location, but what if someone had managed to trick you?

Or what happens if a backdoor is discovered in one of the projects you've
worked on, and the "Author" line in the commit says it was done by you, while
you're pretty sure you had [nothing to do with
it](https://github.com/jayphelps/git-blame-someone-else)?

To address both of these issues, Git introduced PGP integration. Signed tags
prove the repository integrity by assuring that its contents are exactly the
same as on the workstation of the developer who created the tag, while signed
commits make it nearly impossible for someone to impersonate you without
having access to your PGP keys.

### Checklist

- [ ] Understand signed tags, commits, and pushes _(ESSENTIAL)_
- [ ] Configure git to use your key _(ESSENTIAL)_
- [ ] Learn how tag signing and verification works _(ESSENTIAL)_
- [ ] Configure git to always sign annotated tags _(NICE)_
- [ ] Learn how commit signing and verification works _(ESSENTIAL)_
- [ ] Configure git to always sign commits _(NICE)_
- [ ] Configure gpg-agent options _(ESSENTIAL)_

### Considerations

Git implements multiple levels of integration with PGP, first starting with
signed tags, then introducing signed commits, and finally adding support for
signed pushes.

#### Understanding Git Hashes

Git is a complicated beast, but you need to know what a "hash" is in order to
have a good grasp on how PGP integrates with it. We'll narrow it down to two
kinds of hashes: tree hashes and commit hashes.

##### Tree hashes

Every time you commit a change to a repository, git records checksum hashes
of all objects in it -- contents (blobs), directories (trees), file names and
permissions, etc, for each subdirectory in the repository. It only does this
for trees and blobs that have changed with each commit, so as not to
re-checksum the entire tree unnecessarily if only a small part of it was
touched.

Then it calculates and stores the checksum of the toplevel tree, which will
inevitably be different if any part of the repository has changed.

##### Commit hashes

Once the tree hash has been created, git will calculate the commit hash, which
will include the following information about the repository and the change being
made:

- the checksum hash of the tree
- the checksum hash of the tree before the change (parent)
- information about the author (name, email, time of authorship)
- information about the committer (name, email, time of commit)
- the commit message

##### Hashing function

At the time of writing, git still uses the SHA1 hashing mechanism to calculate
checksums, though work is under way to transition to a stronger algorithm that
is more resistant to collisions. Note, that git already includes collision
avoidance routines, so it is believed that a successful collision attack
against git remains impractical.

#### Annotated tags and tag signatures

Git tags allow developers to mark specific commits in the history of each git
repository. Tags can be "lightweight" -- more or less just a pointer at a
specific commit, or they can be "annotated," which becomes its own object in
the git tree. An annotated tag object contains all of the following
information:

- the checksum hash of the commit being tagged
- the tag name
- information about the tagger (name, email, time of tagging)
- the tag message

A PGP-signed tag is simply an annotated tag with all these entries wrapped
around in a PGP signature. When a developer signs their git tag, they
effectively assure you of the following:

- who they are (and why you should trust them)
- what the state of their repository was at the time of signing:
  - the tag includes the hash of the commit
    - the commit hash includes the hash of the toplevel tree
      - which includes hashes of all files, contents, and subtrees
    - it also includes all information about authorship
    - including exact times when changes were made

When you clone a git repository and verify a signed tag, that gives you
cryptographic assurance that _all contents in the repository, including all of
its history, are exactly the same as the contents of the repository on the
developer's computer at the time of signing_.

#### Signed commits

Signed commits are very similar to signed tags -- the contents of the commit
object are PGP-signed instead of the contents of the tag object. A commit
signature also gives you full verifiable information about the state of the
developer's tree at the time the signature was made. Tag signatures and commit
PGP signatures provide exact same security assurances about the repository and
its entire history.

#### Signed pushes

This is included here for completeness' sake, since this functionality needs
to be enabled on the server receiving the push before it does anything useful.
As we saw above, PGP-signing a git object gives verifiable information about
the developer's git tree, but not about their *intent* for that tree.

For example, you can be working on an experimental branch in your own git fork
trying out a promising cool feature, but after you submit your work for
review, someone finds a nasty bug in your code. Since your commits are
properly signed, someone can take the branch containing your nasty bug and
push it into master, introducing a vulnerability that was never intended to go
into production. Since the commit is properly signed with your key, everything
looks legitimate and your reputation is questioned when the bug is discovered.

Ability to require PGP-signatures during `git push` was added in order to
certify the *intent* of the commit, and not merely verify its contents.

#### Configure git to use your PGP key

If you only have one secret key in your keyring, then you don't really need to
do anything extra, as it becomes your default key.

However, if you happen to have multiple secret keys, you can tell git which
key should be used (`[fpr]` is the fingerprint of your key):

    $ git config --global user.signingKey [fpr]

**NOTE**: If you have a distinct `gpg2` command, then you should tell git to
always use it instead of the legacy `gpg` from version 1:

    $ git config --global gpg.program gpg2

#### How to work with signed tags

To create a signed tag, simply pass the `-s` switch to the tag command:

    $ git tag -s [tagname]

Our recommendation is to always sign git tags, as this allows other developers
to ensure that the git repository they are working with has not been
maliciously altered (e.g. in order to introduce backdoors).

##### How to verify signed tags

To verify a signed tag, simply use the `verify-tag` command:

    $ git verify-tag [tagname]

If you are verifying someone else's git tag, then you will need to import
their PGP key. Please refer to the "Trusted Team communication" document in
the same repository for guidance on this topic.

##### Verifying at pull time

If you are pulling a tag from another fork of the project repository, git
should automatically verify the signature at the tip you're pulling and show
you the results during the merge operation:

    $ git pull [url] tags/sometag

The merge message will contain something like this:

    Merge tag 'sometag' of [url]

    [Tag message]

    # gpg: Signature made [...]
    # gpg: Good signature from [...]

#### Configure git to always sign annotated tags

Chances are, if you're creating an annotated tag, you'll want to sign it. To
force git to always sign annotated tags, you can set a global configuration
option:

    $ git config --global tag.forceSignAnnotated true

Alternatively, you can just train your muscle memory to always pass the `-s`
switch:

    $ git tag -asm "Tag message" tagname

#### How to work with signed commits

It is easy to create signed commits, but it is much more difficult to
incorporate them into your workflow. Many projects use signed commits as a
sort of "Committed-by:" line equivalent that records code provenance -- the
signatures are rarely verified by others except when tracking down project
history. In a sense, signed commits are used for "tamper evidence," and not to
"tamper-proof" the git workflow.

To create a signed commit, you just need to pass the `-S` flag to the `git
commit` command (it's capital `-S` due to collision with another flag):

    $ git commit -S

Our recommendation is to always sign commits and to require them of all
project members, regardless of whether anyone is verifying them (that can
always come at a later time).

##### How to verify signed commits

To verify a single commit you can use `verify-commit`:

    $ git verify-commit [hash]

You can also look at repository logs and request that all commit signatures
are verified and shown:

    $ git log --pretty=short --show-signature

##### Verifying commits during git merge

If all members of your project sign their commits, you can enforce signature
checking at merge time (and then sign the resulting merge commit itself using
the `-S` flag):

    $ git merge --verify-signatures -S merged-branch

Note, that the merge will fail if there is even one commit that is not signed
or does not pass verification. As it is often the case, technology is the easy
part -- the human side of the equation is what makes adopting strict commit
signing for your project difficult.

##### If your project uses mailing lists for patch management

If your project uses a mailing list for submitting and processing patches,
then there is little use in signing commits, because all signature information
will be lost when sent through that medium. It is still useful to sign your
commits, just so others can refer to your publicly hosted git trees for
reference, but the upstream project receiving your patches will not be able to
verify them directly with git.

You can still sign the emails containing the patches, though.

#### Configure git to always sign commits

You can tell git to always sign commits:

    git config --global commit.gpgSign true

Or you can train your muscle memory to always pass the `-S` flag to all `git
commit` operations (this includes `--amend`).

##### Bonus: Using gpg-agent with ssh

If you've created an **[A]** (Authentication) key and moved it to the
smartcard, you can use it with ssh for adding 2-factor authentication for your
ssh sessions. You just need to tell your environment to use the correct socket
file for talking to the agent.

First, add the following to your `~/.gnupg/gpg-agent.conf`:

    enable-ssh-support

Then, add this to your `.bashrc`:

    export SSH_AUTH_SOCK=$(gpgconf --list-dirs agent-ssh-socket)

You will need to kill the existing `gpg-agent` process and start a new login
session for the changes to take effect:

    $ killall gpg-agent
    $ bash
    $ ssh-add -L

The last command should list the SSH representation of your PGP Auth key (the
comment should say `cardno:XXXXXXXX` at the end to indicate it's coming from
the smartcard).

To enable key-based logins with ssh, just add the `ssh-add -L` output to
`~/.ssh/authorized_keys` on remote systems you log in to. Congratulations,
you've just made your ssh credentials extremely difficult to steal.

As a bonus, you can get other people's PGP-based ssh keys from public
keyservers, should you need to grant them ssh access to anything:

    $ gpg --export-ssh-key [keyid]

This can come in super handy if you need to allow developers access to git
repositories over ssh.

## Protecting online accounts

### Checklist

- [ ] Get a U2F-capable device _(ESSENTIAL)_
- [ ] Enable 2-factor authentication for your online accounts _(ESSENTIAL)_
  - [ ] GitHub/GitLab
  - [ ] Google
  - [ ] Social Media
- [ ] Use U2F as primary mechanism, with TOTP as fallback _(ESSENTIAL)_

### Considerations

You may have noticed how a lot of your online developer identity is tied to
your email address. If someone can gain access to your mailbox, they would be
able to do a lot of damage to you personally, and to your reputation as a free
software developer. Protecting your email accounts is just as important as
protecting your PGP keys.

#### Two-factor authentication with Fido U2F

[Two-factor authentication](https://en.wikipedia.org/wiki/Multi-factor_authentication)
is a mechanism to improve account security by requiring a physical token in
addition to a username and password. The goal is to make sure that even if
someone steals your password (via keylogging, shoulder surfing, or other
means), they still wouldn't be able to gain access to your account without
having in their possession a specific physical device ("something you have"
factor).

The most widely known mechanisms for 2-factor authentication are:

- SMS-based verification
- Time-based One-Time Passwords (TOTP) via a smartphone app, such as
  the "Google Authenticator" or similar solutions
- Hardware tokens supporting Fido U2F

SMS-based verification is easiest to configure, but has the following
important downsides: it is useless in areas without signal (e.g. most building
basements), and can be defeated if the attacker is able to intercept or divert
SMS messages, for example by cloning your SIM card.

TOTP-based multi-factor authentication offers more protection than SMS, but
has important scaling downsides (there are only so many tokens you can add to
your smartphone app before finding the correct one becomes unwieldy). Plus,
there's no avoiding the fact that your secret key ends up stored on the
smartphone itself -- which is a complex, globally connected device that may or
may not have been receiving timely security patches from the manufacturer.

Most importantly, neither TOTP nor SMS methods protect you from phishing
attacks -- if the phisher is able to steal both your account password and the
2-factor token, they can replay them on the legitimate site and gain access to
your account.

[Fido U2F](https://en.wikipedia.org/wiki/Universal_2nd_Factor) is a standard
developed specifically to provide a mechanism for 2-factor authentication
*and* to combat credential phishing. The U2F protocol will store each site's
unique key on the USB token and will prevent you from accidentally giving the
attacker both your password and your one-time token if you try to use it on
anything other than the legitimate website.

Both Chrome and Firefox support U2F 2-factor authentication, and hopefully
other browsers will soon follow.

#### Get a token capable of Fido U2F

There are [many options available](http://www.dongleauth.info/dongles/) for
hardware tokens with Fido U2F support, but if you're already ordering a
smartcard-capable physical device, then your best option is a Yubikey 4, which
supports both.

#### Enable 2-factor authentication on your online accounts

You definitely want to enable this option on the email provider you are using
(especially if it is Google, which has excellent support for U2F). Other sites
where this functionality should be enabled are:

- **GitHub**: it probably occurred to you when you uploaded your PGP public key
  that if anyone else is able to gain access to your account, they can replace
  your key with their own. If you publish code on GitHub, you should take care
  of your account security by protecting it with U2F-backed authentication.
- **GitLab**: for the same reasons as above.
- **Google**: if you have a google account, you will be surprised how many
  sites allow logging in with Google authentication instead of site-specific
  credentials.
- **Facebook**: same as above, a lot of online sites offer the option to
  authenticate using a Facebook account. You should 2-factor protect your
  Facebook account even if you do not use it.
- Other sites, as you deem necessary. See
  [dongleauth.info](http://www.dongleauth.info) for inspiration.

#### Configure TOTP failover, if possible

Many sites will allow you to configure multiple 2-factor mechanisms, and the
recommended setup is:

- U2F token as the primary mechanism
- TOTP phone app as the secondary mechanism

This way, even if you lose your U2F token, you should be able to re-gain
access to your account. Alternatively, you can enroll multiple U2F tokens
(e.g. you can get another cheap token that only does U2F and use it for
backup reasons).

## Further reading

By this point you have accomplished the following important tasks:

1. Created your developer identity and protected it using PGP cryptography.
2. Configured your environment so your identity is not easily stolen by moving
   your master key offline and your subkeys to an external hardware device.
3. Configured your git environment to ensure that anyone using your project is
   able to verify the integrity of the repository and its entire history.
4. Secured your online accounts using 2-factor authentication.

You are already in a good place, but you should also read up on the following
topics:

- How to secure your team communication (see the document in this repository).
  Decisions regarding your project development and governance require just as
  much careful protection as any committed code, if not so. Make sure that
  your team communication is trusted and the integrity of all decisions is
  verified.
- How to secure your workstation (see the document in this repository). Your
  goal is to minimize risky behaviour that would cause your project code to be
  contaminated, or your developer identity to be stolen.
- How to write secure code (see various documentation related to the
  programming languages and libraries used by your project). Bad, insecure
  code is still bad, insecure code even if there is a PGP signature on the
  commit that introduced it.
