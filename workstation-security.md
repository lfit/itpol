# Workstation security checklist

## Hardware considerations

- [CRITICAL] Has a TPM chip
- [CRITICAL] Supports SecureBoot
- [MODERATE] Has no firewire ports
- [MODERATE] Has no PCMCIA ports

## Pre-boot environment
- [CRITICAL] UEFI boot mode is used (not legacy BIOS)
- [CRITICAL] Password is required to enter UEFI configuration mode
- [CRITICAL] Password is required to initiate boot
- [CRITICAL] SecureBoot is enabled

## Distro choice considerations
- [CRITICAL] Has a robust MAC/RBAC implementation (SELinux/AppArmor/GrSecurity)
- [CRITICAL] Publishes security bulletins
- [CRITICAL] Provides timely security patches
- [CRITICAL] Provides cryptographic verification of packages
- [CRITICAL] Supports TrustedBoot
- [CRITICAL] Has robust full disk encryption support (LUKS)

## Distro installation guidelines
- [CRITICAL] Use full-disk encryption on LVM level
- [CRITICAL] Make sure swap is also encrypted
- [CRITICAL] Set up a unique, robust root password
- [CRITICAL] Use an unprivileged account, part of administrators group (sudo)
- [CRITICAL] Set up a robust user-account password, different from root

## Post-installation hardening
- [CRITICAL] Globally disable firewire modules
             ("blacklist firewire-core" in /etc/modprobe.d/bl-firewire.conf)
- [MODERATE] Check your firewalls to ensure all incoming ports are filtered
- [MODERATE] Check to ensure sshd service is disabled by default
- [MODERATE] Set up an automatic OS update schedule, or update reminders
             (most distros will notify when updates are available)

## Personal workstation backups

## Best practices

### SELinux

- [CRITICAL] Make sure SELinux is enforcing on your workstation
- [CRITICAL] Never `setenforce 0`, use `semanage permissive -a somedomain_t`
- [CRITICAL] Never blindly run `audit2allow`, always check
- [MODERATE] Switch your account to SELinux user `staff_u` (use `usermod -Z`)

### Browsing
- [MODERATE] Use two different browsers, one for work sites only, the other
             for everything else
- [PARANOID] Run the "everything else" browser as a different user
- [PARANOID+] Run the "everthing else" browser inside a local VM accessed
              via RDP.

