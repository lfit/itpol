# Trusted Team Communication

Updated: 2015-08-13

*Status: OUTDATED*

Establishing trusted communication between members of your team is paramount
not only to avoid potential security problems associated with phishing and
impersonation, but also to make it possible to exchange sensitive information
without relying on untrusted or insecure channels.

You should establish trusted communication guidelines as early as possible,
before you put down any code or bring up any servers.

There are 3 core technologies you will be using:

- [OpenPGP][1], favored in the open-source world
- [X.509][2] and [S/MIME][3], favored in the corporate world
- [OTR][4], which is largely limited to instant messaging

In this document we'll be looking at each technology purely in terms of
securing communication between team members and will not judge them on any
other merits. We'll look at several core topics:

1. Trusting emails from your teammates
2. Trusting your IM sessions
3. Trusting your git commits
4. Releasing code trusted by the community

## Trusting email

### OpenPGP vs S/MIME

There are advantages and disadvantages to both, but if you're doing
open-source development at all, chances are that OpenPGP will be a better
solution for you.

S/MIME and OpenPGP do similar things for securing communication, but they
differ in the way they delegate trust. X.509 relies on *Certification
Authorities* to indicate trusted keys, while OpenPGP relies on the *Web of
Trust* to accomplish the same goal.

#### Main upsides of S/MIME

- Your team can choose to rely on global certification authorities to certify
  each teammate's identity, without bothering with setting up your own CA
  infrastructure or dealing with establishing OpenPGP's web of trust.
- S/MIME is very well supported in desktop software, therefore
  barriers to entry and use will be lower than with OpenPGP.
- Portable devices tend to have decent mail client support for S/MIME,
  allowing to both read and send secure emails.

#### Main downsides of S/MIME

- Has poor acceptance in the open-source world, which relies a lot heavier on
  OpenPGP standards. If you will need to communicate with other teams of
  developers across the open-source realm, chances are that they will not be
  using S/MIME and may ask you to use OpenPGP instead.
- Many globally trusted CAs only need someone to verify that they can receive
  messages at the email address being certified. If your team relies on
  external CAs, your trusted communication will be as weak as the password on
  any given teammate's inbox.
- If you do NOT rely on external CAs, then you will need to bring up and
  maintain your own CA infrastructure, which means extra work to ensure its
  security, plus you will be placing ultimate trust into the person or persons
  maintaining your CA systems.

#### Main upsides of OpenPGP

- Does not rely on external trust entities. Every member of the project will
  maintain their own web of trust. You do not have to run a CA, or place
  ultimate trust into the person who runs your CA infrastructure.
- OpenPGP is well accepted across open-source development teams for securing
  email communication, to a much greater degree than S/MIME.
- OpenPGP is used in many free software areas beyond just securing email, for
  example to create signed git tags and commits and to produce trusted
  software releases via detached OpenPGP signatures.
- OpenPGP email has decent support in most desktop email clients, either as a
  core feature, or as an add-on.

#### Main downsides of OpenPGP

- Few developers understand how the web of trust works, and even fewer
  managers do.
- Webs of trust are hard to get going if your team members are geographically
  dispersed.
- OpenPGP tools are difficult to learn and use, even by the technically
  inclined and educated.
- There are almost no mail clients for portable devices that support reading or
  sending OpenPGP-encrypted mail. Those that do support OpenPGP have
  questionable GUI or awkward UX.

### Understanding the OpenPGP Web of Trust

OpenPGP is only a useful tool when everyone involved in team communication
understands how the web of trust works. Please see the following links for a
couple of detailed explanations:

- [PGP Web of Trust: Core Concepts Behind Trusted Communication][5]
- [PGP Web of Trust: How does it work?][6]

Main takeaways should be:

- Never trust keys without any signatures on them
- Only use keys listed as "fully valid" for encrypted correspondence
- Use keyservers and refresh your public keys regularly

### Using the Web of Trust in your team

Once you understand the core concepts behind the OpenPGP Web of Trust, you'll
have to actively enforce and maintain it. Nobody should get access to your
infrastructure or be allowed to push code without being part of your web of
trust.

#### Spinning the web

Web of trust is established via signing your teammate's keys and assigning
them owner-trust. The protocol calls for an in-person meeting where both
parties present documents validating their identities and exchange key
fingerprints. Here's an in-depth document describing the procedure:

- https://www.phildev.net/pgp/gpgsigning.html

##### Yes, but what if they are 12 timezones away?

If the new addition to your team lives far away from everyone else and has no
easy means of travel, then it is acceptable to set up a video session and have
them show their identification papers to the camera. Obviously, this process
is easier to subvert than with a person-to-person meeting, but not by much.
Unless you are an expert at identifying various foreign or out-of-state
identification documents, it would be easy for an attacker to print out and
laminate a convincing driver's license or government ID.

At any rate, this protocol is less about identifying a person's state-issued
identity, and more about creating a communication channel that is equally as
trusted as a video session, a phone call, or an in-person meeting. If you are
comfortable enough that the person at the other end of your video chat is who
they say they are, and that they belong on your team, then you should feel
comfortable enough to sign their key.

##### Keysigning parties

Chances are, large portions of your team will be attending the same events or
even specially-organized hackfests. You should use this opportunity to
strengthen your web of trust by holding "[keysigning parties][8]."

### Sending trusted emails

#### When to sign

There are two distinct operations with each outgoing email, both for S/MIME
and OpenPGP:

- **Sign**, which signs your message in a way that proves to the recipient
  that you were the one who sent it and that the message was not tampered with
  in transit.
- **Encrypt**, which encrypts your message so the contents cannot be read in
  transit by mail relays.

Best practice is to always sign your messages, unless you have a good reason
not to (usually for plausible deniability reasons). For OpenPGP, the
recommended mechanism for signatures is MIME-signing, as inline-signing tends
to clutter the message with OpenPGP headers and footers and may annoy your
correspondents.

#### When to encrypt

You should only need to encrypt messages actually containing sensitive data
that you do not wish others to know about (passwords and other account
information, confidential details that should not leak to the public, etc).
Chances are, your recipient has only configured their mail client to read
encrypted emails on their workstation and not on their mobile device, so
adopting a policy of "always encrypt by default" may annoy your recipients
when they can't read your message from the comfort of their couch, especially
if the contents are not confidential or sensitive (chitchat, lunch
invitations, bikeshedding discussions, etc).

**When encrypting, you should also sign the message** (unless you *do* need
that plausible deniability). **Simply encrypting the message does not make it
trusted**, as an enemy is also perfectly capable of encrypting a message to
someone's public key and faking the "From:" header.

If you receive an encrypted message that isn't also signed, be extra wary,
as it may be a [spear phishing][7] attempt.

## Trusting IM sessions

Almost all teams use some kind of instant messaging mechanism in order to
coordinate their activities in real time -- be it IRC, Hangouts, Slack, or any
number of other means. If critical decisions are taken during such meetings,
then you should ensure that they happen over trusted communication channels.

### One-on-one messaging

There is no lack of clients for instant messaging, and while most of them will
encrypt your conversations from the client to the server, the contents will
still be seen in cleartext by the service providers, and most likely logged in
some fashion. In some cases, these conversations can be later retrieved by
attackers, so if you need to ensure that the provider does not know about the
contents of your messages, you must take steps to communicate via a protocol
that offers point-to-point encryption and verification.

The only widely used cross-client protocol for securing end-to-end
communication is Off-The-Record messaging (OTR). It is easy to set up in most
desktop clients, and there are several mobile apps available for communicating
on the go, just search for "OTR" and you should be able to find them (e.g.
SecureChat, IM+).

As with email, merely encrypting your connections does nothing to assure that
the person you are talking to is who they claim they are. You will need to
verify your contacts via OTR's excellent verification protocols before you
trust the chat session to be secure.

If you choose not to bother with point-to-point encryption for chat sessions
with your team members, then you should firmly establish, as a matter of
policy, what kind of conversations are suitable for IM, and what should be
only sent via secured email.

**NOTE:** Google has, confusingly, called something else "Off-The-Record"
conversations, which merely exclude your chat sessions from being logged in
your Inbox, but they are not point-to-point encrypted, and are still known to
Google.

### Group messaging

There is currently no widely used mechanisms to set up perfectly secure
multi-user group chat sessions with point-to-point encryption. You may
sidestep this limitation by running your own multi-user chat server (IRC,
Jabber, HipChat) and requiring that everyone both authenticates and connects
via a trusted protocol (i.e. TLS), but you will still have to trust the
administrators of that server not to log or misuse your data.

Alternatively, simply establish a firm policy that only public conversations
are allowed in group chat and everything else should happen over secure email
exchange.

## Trusting git commits

You will *most likely* be using git in some fashion -- be it to track your
code, your system's config management tree, etc. You can use OpenPGP to add a
layer of trust to your tags and commits.

#### Signed-off-by's

First of all, *Signed commits* shouldn't be confused with *Signed-off-by:*
lines in git commit messages. Despite sounding similar, the *Signed-off-by:*
entries actually offer no guarantees that they were inserted there by those
people. These are merely a way to track code provenance and code review
chains, but offer no assurances.

#### Signed tags and commits

You can, however, create cryptographic signatures when committing code to a
git repository, or when creating a tag, as a way to assure people who will be
pulling these commits that the repository they are pulling from has not been
tampered with by an attacker.

The easiest is to just sign the tags -- which will help, however may not be
sufficient depending on the nature of your project. More recent versions of
git have introduced a way to sign each individual commit, which makes it
significantly more difficult for an attacker to sneak in malicious code
into someone's tree. However, without proper checking done by project
maintainers, this will only make it tamper-evident, and not tamper-proof (in
other words, someone may sneak in a malicious commit, and the most you'll be
able to do is exonerate a trusted developer, but not prevent the compromise
from going out to your users).

Signed commits also make merges and other branch operations more complicated,
but not insurmountable. Please see the following in-depth document to learn
more about how you can organize your workflow around signed git commits:

- http://mikegerwitz.com/papers/git-horror-story

## Releasing code trusted by the community

If you produce open-source code, you have to establish a way for the community
to validate that the releases you put forward are trusted and have not been
tampered with by attackers. If you are relying on the OpenPGP web of trust,
you should designate one or several "release engineers" who will be
responsible for creating detached OpenPGP signatures and making them available
at the same download location as the rest of the code.

Alternatively, you may choose to create a separate key that will be used for
signing the releases, but in this case you should get several core members of
your team to sign that key to indicate that it should be trusted by others.
Special care should be taken to ensure that the release-signing key is kept
secure, and preferably protected by a passphrase. If you choose to make it
part of an automated process, make sure you don't expose the key to external
entities.

To create a detached signature for a tarball, use the following command:

    gpg -ba tarball.tar.xz

This will create a file called `tarball.tar.xz.asc` which should be uploaded
to the release server.

Alternatively, if you only release via git, you may simply use signed git
tags and let packagers create their own tarballs from git itself.

## Securing infrastructure access

You should rely on these trusted communication channels to grant access to
your infrastructure -- be it shell access to the servers, git-over-ssh access
with public keys, credentials to access VPN or shared services, etc.

Most commonly, this will entail having someone send in their ssh public key,
which should always be sent via signed email (no sense encrypting it, though).

### Using PGP keys with SSH

Alternatively, you can use GnuPG with ssh to turn an OpenPGP Auth key into an
ssh private key. We publish a [detailed guide][9] on how to do that using
either a smartcard reader or a Yubikey NEO.

This will offer an extra benefit of adding [2-factor authentication][10] to
your infrastructure access (yubikeys and smartcards are "something you have").

As a server administrator, you can easily convert someone's GnuPG Auth key
into an SSH public key. First, find out what their Auth keyid is by running
`gpg --edit-key [email]` and looking for the "`Usage: A`" entry, e.g.:

    sub  rsa2048/80A407E7
     created: 2014-06-03  expires: never       usage: A

Then, run `gpgkey2ssh` command with that key ID:

    gpgkey2ssh 80A407E7

This will produce the output that you can use for the `authorized_keys` file.
This saves you the trouble of asking them to send you their ssh public key,
and assures that the key actually belongs to your team member since it's part
of their trusted OpenPGP master key.

## Checklist

Here is a convenient checklist for your team to ensure that you have all these
aspects covered.

- [ ] Every member of the team has a trusted PKI certificate
  - [ ] If using X.509, the certificate is issued by a CA trusted by all
        members of the team
  - [ ] If using OpenPGP, each team member's key carries at least one,
        preferably three signatures from other members of the team

- [ ] Members of the team are able to send and receive encrypted, signed email
  - [ ] Members of the team understand the necessity behind email security,
        and agree to it as a requirement for discussing sensitive subjects
  - [ ] Members of the team know how to verify cryptographic signatures to
        establish that the email came from a trusted source and has not been
        altered in transit

- [ ] The team uses an agreed-upon IM client and protocol for secure, trusted
      real-time communication with point-to-point encryption
- [ ] The team uses an agreed-upon group chat mechanism and understands its
      limitations and security risks
- [ ] Members of the team know which subjects should be discussed only via
      trusted and encrypted channels (security, accounts, confidential data,
      etc), and never via public channels such as group chat or untrusted
      IM and email

- [ ] The team has an established workflow that makes use of cryptographic
      features in git
- [ ] The project has a defined mechanism for providing trusted releases
      either via detached OpenPGP signatures, or via signed git tags
- [ ] Access to infrastructure is granted via trusted communication channels

## License
This work is licensed under a
[Creative Commons Attribution-ShareAlike 4.0 International License][0].

[0]: http://creativecommons.org/licenses/by-sa/4.0/
[1]: https://en.wikipedia.org/wiki/Pretty_Good_Privacy#OpenPGP
[2]: https://en.wikipedia.org/wiki/X.509
[3]: https://en.wikipedia.org/wiki/S/MIME
[4]: https://en.wikipedia.org/wiki/Off-the-Record_Messaging
[5]: https://www.linux.com/learn/tutorials/760909-pgp-web-of-trust-core-concepts
[6]: http://slides.com/mricon/pgp-web-of-trust
[7]: https://en.wikipedia.org/wiki/Phishing#List_of_phishing_types
[8]: https://en.wikipedia.org/wiki/Key_signing_party
[9]: https://github.com/lfit/ssh-gpg-smartcard-config
[10]: https://en.wikipedia.org/wiki/Two-factor_authentication
