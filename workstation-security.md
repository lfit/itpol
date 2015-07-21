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
- [CRITICAL] Supports TrustedBoot
- [CRITICAL] Has robust full disk encryption support (LUKS)

## Distro installation guidelines
- [CRITICAL] Use full-disk encryption
- [CRITICAL] Create a separate /home partition
    - Make sure swap is also encrypted

## Untrusted hardware

- Firewire ports are disabled

    - blacklist firewire-core in /etc/modprobe.d/blacklist-firewire.conf

-

Team communication:
- Establish PGP web of trust
  - Or use s/mime with a trusted CA
- Use a password vault


Practices:
- Apply updates daily

SELinux hints:
- Run as SELinux user staff_u
- Never setenforce 0
  - Use "semanage permissive -a somedomain_t"
  - 
