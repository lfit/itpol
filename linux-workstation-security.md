# Linux workstation security checklist

This is a set of recommendations used by the Linux Foundation for their systems
administrators. All of LF employees are remote workers and we use this set of
guidelines to ensure that a sysadmin's system passes core security requirements
in order to reduce the risk of it becoming the attack vector against the rest
of our infrastructure.

Even if your systems administrators are not remote workers, chances are that
they perform a lot of their work either from a portable laptop in a work
environment, or set up their home systems to access work infrastructure for
after-hours/emergency support. In either case, you can adapt this set of
recommendations to suit your environment.

This, by no means, is an exhaustive "workstation hardening" document, but
rather an attempt at a set of baseline recommendations to avoid most glaring
security errors without introducing too much inconvenience. You may read this
document and think it is way too paranoid, while someone else may think this
barely scratches the surface. Security is just like driving on a highway --
anyone going slower than you is an idiot, while anyone driving faster than you
is a crazy person. Following this analogy, this set of recommendations is
merely a basic set of highway safety rules that is neither exhaustive, nor a
replacement for experience, vigilance, and common sense.

## Hardware considerations

We do not mandate that our admins use a specific vendor or a specific model, so
this section addresses core considerations when choosing a work system.

#### [CRITICAL] System supports SecureBoot

Despite its controversial nature, SecureBoot offers prevention against many
attacks targetting workstations (Rootkits, "Evil Maid," etc), without too much
hassle involved. It will not stop a truly dedicated attacker, plus there is a
pretty high degree of certainty that state security agencies have ways to
defeat it (probably by design), but having SecureBoot is better than having
nothing at all.

Alternatively, you may set up [AntiEvilMaid][1] which offers more wholesome
protection against the type of attacks that SecureBoot is supposed to prevent,
but it will require more hassle to set up and maintain.

[1]: https://github.com/QubesOS/qubes-antievilmaid


#### [MODERATE] Has no firewire, thunderbolt, or ExpressCard ports

Firewire is a terrible standard that, by design, allows any connecting device full
direct memory access [see Wikipedia][2]. Thunderbolt and ExpressCard are guilty
of the same sin, though some later implementations of Thunderbold attempt to
mitigate this vulnerability. It is best if the system you are getting has none
of these ports, but it is not critical, as they usually can be turned off via
EFI or disabled in the kernel itself.

[2]: https://en.wikipedia.org/wiki/IEEE_1394#Security_issues

#### [LOW] System has a TPM chip

Trusted Platform Module (TPM) is a crypto chip built into the system itself,
which can be used for additional platform security (such as to store full-disk
encryption keys), but is not normally used for day-to-day workstation
operation. At best, this is a nice-to-have, unless you have a specific need to
use TPM for your workstation security.

## Pre-boot environment

This is a set of recommendations for your workstation before you even start
with OS installation.

#### [CRITICAL] UEFI boot mode is used (not legacy BIOS)

UEFI, with all its warts, offers a lot of goodies that legacy BIOS doesn't,
such as SecureBoot. Most modern systems come with UEFI mode on by default.

#### [CRITICAL] Password is required to enter UEFI configuration mode

Make sure a strong password is required to enter UEFI configuration mode. Pay
attention, as many manufacturers quietly limit the length of the password you
are allowed to use, so you may need to choose high-entropy short passwords vs.
long passphrases.

#### [CRITICAL] SecureBoot is enabled

See note above about SecureBoot. Depending on the Linux distribution you decide
to use, you may or may not have to jump through additional hoops in order to
import your distribution's SecureBoot key that would allow you to boot the
distro. Many distributions have partnered with Microsoft to sign their released
kernels with a key that is already recognized by most system manufacturers,
therefore saving you the trouble of having to deal with this.

#### [MODERATE] Password is required to initiate boot

Before someone is allowed to even get to the boot partition and try some
badness there, let's make them enter a password. This password may be the same
or different from your UEFI management password, depending on whether you'll
always be the only person using this laptop or whether you'll be sharing it
with others. If you shut down and start a lot, you may choose to not bother
with this, as you will already have to enter a LUKS passphrase and this will
save you a few extra keystrokes.

## Distro choice considerations

Chances are you'll stick with a fairly widely-used distribution such as Fedora,
Ubuntu, Arch, Debian, or one of their close spin-offs. In any case, this is
what you should consider when picking a distribution to use.

#### [CRITICAL] Has a robust MAC/RBAC implementation (SELinux/AppArmor/GrSecurity)

Mandatory Access Controls (MAC) or Role-Based Access Controls are an extention
of the basic user/group based security mechanism used in legacy POSIX systems.
Most distributions these days either already come bundled with a MAC/RBAC
implementation (Fedora, Ubuntu), or provide a mechanism to add it via an
optional post-installation add-on (Gentoo, Arch, Debian). Obviously, it is
highly advised that you pick a distribution that comes pre-configured with a
MAC system, but if you have strong feelings about a distribution that doesn't
come with one enabled by default, do plan to configure it post-installation.

#### [CRITICAL] Publishes security bulletins

This is the case for most widely used distributions, but if you are fond of
something esoteric, check whether the developers have a documented mechanism of
alerting the users about security vulnerabilities and patches. Absence of such
mechanism is a major warning sign that the distribution is not mature enough to
be considered for a primary admin workstation.

#### [CRITICAL] Provides timely security patches

Also true for most widely used distributions, but is worth checking to ensure
that critical package updates are provided in a timely fashion. Avoid using
spin-offs and "community rebuilds" for this reason, as they routinely delay
security updates due to having to wait for the upstream distribution to release
it first.

#### [CRITICAL] Provides cryptographic verification of packages

You'll be hard-pressed to find a distribution that does not use cryptographic
signatures on packages, updates metadata, or both. That being said, fairly
widely used distributions have been known to go for years before introducing
this basic security measure (Arch, I'm looking at you), so this is a thing
worth checking.

#### [CRITICAL] Supports SecureBoot

Check that the distribution supports UEFI and SecureBoot. Find out whether it
requires importing an extra key or whether it signs its boot kernels with a key
already trusted by systems manufacturers (e.g. via an agreement with
Microsoft). Some distributions do not support UEFI/SecureBoot but offer
alternatives to ensure tamper-proof or tamper-evident boot environments
(Qubes-OS uses Anti Evil Maid, mentioned earlier). If a distribution doesn't
support SecureBoot and has no mechanisms to prevent boot-level attacks, look
elsewhere.

#### [CRITICAL] Has robust full disk encryption support (LUKS)

Full disk encryption is a requirement for securing data at rest, and is
supported by most distributions. As an alternative, systems with
self-encrypting hard drives may be used (normally implemented via an on-board
TPM chip) and offer comparable levels of security plus faster operation, but
cost extra.

## Distro installation guidelines
- [CRITICAL] Use full-disk encryption on LVM level
- [CRITICAL] Make sure swap is also encrypted
- [CRITICAL] Set up a unique, robust root password
- [CRITICAL] Use an unprivileged account, part of administrators group
- [CRITICAL] Set up a robust user-account password, different from root

## Post-installation hardening
- [CRITICAL] Globally disable firewire and thunderbolt modules
             ("blacklist firewire-core" in /etc/modprobe.d/bl-firewire.conf)
- [MODERATE] Check your firewalls to ensure all incoming ports are filtered
- [MODERATE] Check to ensure sshd service is disabled by default
- [MODERATE] Set up an automatic OS update schedule, or update reminders
             (most distros will notify when updates are available)
- [MODERATE] Configure screensaver to auto-lock after 10 minutes of inactivity
- [MODERATE] Set up logwatch (usually requires rsyslog on systemd platforms)
- [LOW] Install and use rkhunter (use in conjunction with system updates)
- [PARANOID] Install an Intrusion Detection System (aide, tripwire)

## Personal workstation backups
- [CRITICAL] Set up encrypted workstation backups to external storage
             (duplicity/deja-dup, or encrypt external drive using LUKS)
- [CRITICAL] Use zero-knowledge backup tools for selective cloud backups
             (SpiderOak has a very good Linux GUI tool)

## Best practices

### SELinux

- [CRITICAL] Make sure SELinux is enforcing on your workstation
- [CRITICAL] Never `setenforce 0`, use `semanage permissive -a somedomain_t`
- [CRITICAL] Never blindly run `audit2allow`, always check
- [MODERATE] Switch your account to SELinux user `staff_u` (use `usermod -Z`)
             (use `sudo -r sysadm_r` when performing sudo)

### Browsing
- [CRITICAL] Use two different browsers, firefox for work sites only, chromium
             for everything else.
- [MODERATE] Run chromium inside a local VM accessed via spice/RDP.
  - [PARANOID] Use Qubes-os as your main workstation distro
- [CRITICAL] Disable java in Chromium
- [CRITICAL] Require "click to run plugins" in Chromium

