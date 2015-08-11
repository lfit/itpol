# Trusted Team Communication

Establishing trusted communication between members of your team is paramount
not only to avoid potential security problems associated with phishing and
impersonation, but also to make it possible to exchange potentially sensitive
information without having to rely on half-baked or insecure channels.

You should establish trusted communication guidelines as early as possible,
before you put down any code or bring up any servers.

There are 3 core technologies you will be using:

- [OpenPGP][1], favored in the open-source world
- [X.509][2] and [S/MIME][3], favored in the corporate world
- [OTR][4], which is largely limited to instant messaging

In this document we'll be looking at each technology purely in terms of
securing communication between team members and will not judge them on any
other merits. We'll look at three areas:

1. Trusting emails from your teammates
2. Trusting IM sessions
3. Trusting git commits
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
  allowing to both read and write secure emails.

#### Main downsides of S/MIME

- Has poor acceptance in the open-source world, which relies a lot heavier on
  OpenPGP standards. If you will need to communicate with other teams of
  developers across the open-source realm, chances are they will not be using
  S/MIME and may ask you to to use OpenPGP instead.
- Many globally trusted CAs only need someone to verify that they can receive
  email sent to the email address being certified. If your team relies on
  external CAs, your trusted communication will be as weak as the password on
  some developer's inbox.
- If you do NOT rely on external CAs, then you will need to bring up and
  maintain your own CA, which means extra work to ensure its security, plus
  you will be placing ultimate trust into the person or persons maintaining
  your CA infrastructure.

#### Main upsides of OpenPGP

- Does not rely on external trust entities. Every member of the project will
  maintain their own web of trust. You do not have to place ultimate trust
  into the person who runs your CA infrastructure.
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
  sending OpenPGP-encrypted mail. Many that do support OpenPGP have
  questionable or awkward GUI.

### Understanding the OpenPGP Web of Trust

### When to sign

### When to encrypt

## Trusting IM sessions

## Trusting git commits

#### Signed-off by's

#### Signed tags and commits

## Releasing code trusted by the community

## Securing infrastructure access

### Using PGP keys with SSH

[1]: https://en.wikipedia.org/wiki/Pretty_Good_Privacy#OpenPGP
[2]: https://en.wikipedia.org/wiki/X.509
[3]: https://en.wikipedia.org/wiki/S/MIME
[4]: https://en.wikipedia.org/wiki/Off-the-Record_Messaging

