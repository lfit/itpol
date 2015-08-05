# Linux workstation security checklist
This is a set of recommendations used by the Linux Foundation for their systems administrators. All of LF employees are remote workers and we use this set of guidelines to ensure that a sysadmin's system passes core security requirements in order to reduce the risk of it becoming the attack vector against the rest of our infrastructure.

Even if your systems administrators are not remote workers, chances are that they perform a lot of their work either from a portable laptop in a work environment, or set up their home systems to access work infrastructure for after-hours/emergency support. In either case, you can adapt this set of recommendations to suit your environment.

This, by no means, is an exhaustive "workstation hardening" document, but rather an attempt at a set of baseline recommendations to avoid most glaring security errors without introducing too much inconvenience. You may read this document and think it is way too paranoid, while someone else may think this barely scratches the surface. Security is just like driving on a highway -- anyone going slower than you is an idiot, while anyone driving faster than you is a crazy person. Following this analogy, this set of recommendations is merely a basic set of highway safety rules that is neither exhaustive, nor a replacement for experience, vigilance, and common sense.

## Choosing the right hardware
We do not mandate that our admins use a specific vendor or a specific model, so this section addresses core considerations when choosing a work system.

### Checklist
- [ ] System supports SecureBoot _(CRITICAL)_
- [ ] System has no firewire, thunderbolt or ExpressCard ports _(MODERATE)_
- [ ] System has a TPM chip _(LOW)_

### Considerations
#### SecureBoot
Despite its controversial nature, SecureBoot offers prevention against many attacks targeting workstations (Rootkits, "Evil Maid," etc), without too much hassle involved. It will not stop a truly dedicated attacker, plus there is a pretty high degree of certainty that state security agencies have ways to defeat it (probably by design), but having SecureBoot is better than having nothing at all.

Alternatively, you may set up [Anti Evil Maid][1] which offers more wholesome protection against the type of attacks that SecureBoot is supposed to prevent, but it will require more hassle to set up and maintain.

#### Firewire, thunderbolt, and ExpressCard ports
Firewire is a silly standard that, by design, allows any connecting device full direct memory access ([see Wikipedia][2]). Thunderbolt and ExpressCard are guilty of the same sin, though some later implementations of Thunderbolt attempt to mitigate this vulnerability. It is best if the system you are getting has none of these ports, but it is not critical, as they usually can be turned off via EFI or disabled in the kernel itself.

#### TPM Chip
Trusted Platform Module (TPM) is a crypto chip built into the system itself, which can be used for additional platform security (such as to store full-disk encryption keys), but is not normally used for day-to-day workstation operation. At best, this is a nice-to-have, unless you have a specific need to use TPM for your workstation security.

## Pre-boot environment
This is a set of recommendations for your workstation before you even start with OS installation.

### Checklist
- [ ] UEFI boot mode is used (not legacy BIOS) _(CRITICAL)_
- [ ] Password is required to enter UEFI configuration _(CRITICAL)_
- [ ] SecureBoot is enabled _(CRITICAL)_
- [ ] UEFI-level password is required to boot the system _(LOW)_

### Considerations
#### UEFI and SecureBoot
UEFI, with all its warts, offers a lot of goodies that legacy BIOS doesn't, such as SecureBoot. Most modern systems come with UEFI mode on by default.

Make sure a strong password is required to enter UEFI configuration mode. Pay attention, as many manufacturers quietly limit the length of the password you are allowed to use, so you may need to choose high-entropy short passwords vs. long passphrases.

Depending on the Linux distribution you decide to use, you may or may not have to jump through additional hoops in order to import your distribution's SecureBoot key that would allow you to boot the distro. Many distributions have partnered with Microsoft to sign their released kernels with a key that is already recognized by most system manufacturers, therefore saving you the trouble of having to deal with this.

Before someone is allowed to even get to the boot partition and try some badness there, let's make them enter a password. This password may be the same or different from your UEFI management password, depending on whether you'll always be the only person using this laptop or whether you'll be sharing it with others. If you shut down and start a lot, you may choose to not bother with this, as you will already have to enter a LUKS passphrase and this will save you a few extra keystrokes.

## Distro choice considerations
Chances are you'll stick with a fairly widely-used distribution such as Fedora, Ubuntu, Arch, Debian, or one of their close spin-offs. In any case, this is what you should consider when picking a distribution to use.

### Checklist
- [ ] Has a robust MAC/RBAC implementation (SELinux/AppArmor/PaX) _(CRITICAL)_
- [ ] Publishes security bulletins _(CRITICAL)_
- [ ] Provides timely security patches _(CRITICAL)_
- [ ] Provides cryptographic verification of packages _(CRITICAL)_
- [ ] Fully supports UEFI and SecureBoot _(CRITICAL)_
- [ ] Has robust native full disk encryption support _(CRITICAL)_

### Considerations
#### SELinux, AppArmor, and GrSecurity/PaX
Mandatory Access Controls (MAC) or Role-Based Access Controls are an extension of the basic user/group based security mechanism used in legacy POSIX systems. Most distributions these days either already come bundled with a MAC/RBAC implementation (Fedora, Ubuntu), or provide a mechanism to add it via an optional post-installation add-on (Gentoo, Arch, Debian). Obviously, it is highly advised that you pick a distribution that comes pre-configured with a MAC/RBAC system, but if you have strong feelings about a distribution that doesn't come with one enabled by default, do plan to configure it post-installation.

Distributions that do not provide any MAC/RBAC mechanisms should be strongly avoided, as traditional POSIX user- and group-based security mechanisms should be considered insufficient in this day and age. If you would like to start out with a MAC/RBAC workstation, AppArmor and PaX are generally considered easier to learn than SELinux. Furthermore, on a workstation, where there are few or no externally listening daemons, and where user-run applications pose the highest risk, GrSecurity/PaX will probably offer more security benefits than SELinux.

#### Distro security bulletins
Most widely used distributions have a mechanism to deliver security bulletins to its users, but if you are fond of something esoteric, check whether the developers have a documented mechanism of alerting the users about security vulnerabilities and patches. Absence of such mechanism is a major warning sign that the distribution is not mature enough to be considered for a primary admin workstation.

#### Timely and trusted security updates
Most widely used distributions deliver security updates, but is worth checking to ensure that critical package updates are provided in a timely fashion. Avoid using spin-offs and "community rebuilds" for this reason, as they routinely delay security updates due to having to wait for the upstream distribution to release it first.

You'll be hard-pressed to find a distribution that does not use cryptographic signatures on packages, updates metadata, or both. That being said, fairly widely used distributions have been known to go for years before introducing this basic security measure (Arch, I'm looking at you), so this is a thing worth checking.

#### Distros supporing UEFI and SecureBoot
Check that the distribution supports UEFI and SecureBoot. Find out whether it requires importing an extra key or whether it signs its boot kernels with a key already trusted by systems manufacturers (e.g. via an agreement with Microsoft). Some distributions do not support UEFI/SecureBoot but offer alternatives to ensure tamper-proof or tamper-evident boot environments (Qubes-OS uses Anti Evil Maid, mentioned earlier). If a distribution doesn't support SecureBoot and has no mechanisms to prevent boot-level attacks, look elsewhere.

#### Full disk encryption
Full disk encryption is a requirement for securing data at rest, and is supported by most distributions. As an alternative, systems with self-encrypting hard drives may be used (normally implemented via an on-board TPM chip) and offer comparable levels of security plus faster operation, but at a considerably higher cost.

## Distro installation guidelines
### Checklist
- [ ] Use full disk encryption with a robust passphrase _(CRITICAL)_
- [ ] Make sure swap is also encrypted _(CRITICAL)_
- [ ] Set up a unique, robust root password _(CRITICAL)_
- [ ] Use an unprivileged account, part of administrators group _(CRITICAL)_
- [ ] Set up a robust user-account password, different from root _(CRITICAL)_

### Considerations
#### Full disk encryption
Unless you are using self-encrypting hard drives, it is important to configure your installer to fully encrypt all the disks that will be used for storing your data and your system files. It is not sufficient to simply encrypt the user directory via auto-mounting cryptfs loop devices (I'm looking at you, Ubuntu), as this offers no protection for system binaries or swap, which is likely to contain a slew of sensitive data. The recommended encryption strategy is to encrypt the LVM device, so only one passphrase is necessary during the boot process.

The `/boot` partition will always remain unencrypted, as the bootloader needs to be able to actually boot the kernel before invoking LUKS/dm-crypt. The kernel image itself should be protected against tampering with a cryptographic signature checked by SecureBoot. In other words, `/boot` should always be the only unencrypted partition on your system.

#### Choosing good passphrases
Modern Linux systems have no limitation of password/passphrase length, so the only real limitation is your level of paranoia and your stubbornness. If you boot your system a lot, you will probably have to type at least two different passwords: one to unlock LUKS, and another one to log in, so having long passphrases will probably get old really fast. Pick passphrases that are 2-3 words long, preferably from mixed vocabularies. Unfortunately, attackers have proven that passphrases constructed with common words (`correct horse battery staple`) don't actually have the amount of entropy touted by the infamous comic. Throwing in punctuation and numerals helps improve things.

Examples of good passphrases (yes, you can use spaces):
- My abhorrent pantomimes!
- 12 in-flight Jebediahs
- perdona, tengo flatulence

You can also stick with non-vocabulary passwords that are at least 10-12 characters long, if you prefer that to typing passphrases.

## Post-installation hardening
- [CRITICAL] Globally disable firewire and thunderbolt modules
- ("blacklist firewire-core" in /etc/modprobe.d/bl-firewire.conf)
- [MODERATE] Check your firewalls to ensure all incoming ports are filtered
- [MODERATE] Check to ensure sshd service is disabled by default
- [MODERATE] Set up an automatic OS update schedule, or update reminders
- (most distros will notify when updates are available)
- [MODERATE] Configure screensaver to auto-lock after 10 minutes of inactivity
- [MODERATE] Set up logwatch (usually requires rsyslog on systemd platforms)
- [LOW] Install and use rkhunter (use in conjunction with system updates)
- [PARANOID] Install an Intrusion Detection System (aide, tripwire)

## Personal workstation backups
- [CRITICAL] Set up encrypted workstation backups to external storage
- (duplicity/deja-dup, or encrypt external drive using LUKS)
- [CRITICAL] Use zero-knowledge backup tools for selective cloud backups
- (SpiderOak has a very good Linux GUI tool)

## Best practices
### SELinux
- [CRITICAL] Make sure SELinux is enforcing on your workstation
- [CRITICAL] Never `setenforce 0`, use `semanage permissive -a somedomain_t`
- [CRITICAL] Never blindly run `audit2allow`, always check
- [MODERATE] Switch your account to SELinux user `staff_u` (use `usermod -Z`)
- (use `sudo -r sysadm_r` when performing sudo)

### Browsing
- [CRITICAL] Use two different browsers, firefox for work sites only, chromium
- for everything else.
- [MODERATE] Run chromium inside a local VM accessed via spice/RDP.
  - [PARANOID] Use Qubes-os as your main workstation distro

- [CRITICAL] Disable java in Chromium
- [CRITICAL] Require "click to run plugins" in Chromium

[1]: https://github.com/QubesOS/qubes-antievilmaid
[2]: https://en.wikipedia.org/wiki/IEEE_1394#Security_issues
