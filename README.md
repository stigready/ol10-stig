# ol10_stig — Ansible role (ol10-stig)

**Ansible hardening role** for **Oracle Linux 10**, generated from the DRAFT - STIG for Oracle Linux 10 profile in ComplianceAsCode SSG 0.1.82 (SSG's DRAFT profile; SSG says it is not based on a DISA STIG). Suitable for playbooks, Packer/Ansible provisioners, and golden-image pipelines. Search keywords: `ansible`, `ansible-role`, `compliance`, `devsecops`, `hardening`, `infrastructure`, `oci`, `ol10`, `openscap`, `oracle`, `oracle-linux`, `security`, `stig`, `stigforge`.

StigForge-exported Ansible role **`ol10_stig`** · release **`0.3.51`**.
Matrix cell status: **`green`**.

## Before you run this role

Each default below changes how a running host behaves. Read them before you apply the role to a host you depend on.

### Passwords that root sets must meet the password quality rules

_Requirement: ComplianceAsCode rule `accounts_password_pam_enforce_root`, selected by the STIG profile for this product._

**What the role does.** Adds `enforce_for_root` to /etc/security/pwquality.conf, where the pwquality library is installed. pam_pwquality then rejects a password root sets (for root or for another user, with passwd) that fails the length and complexity rules, instead of only warning.

**Who is affected.** Administrators and provisioning scripts that set passwords as root through PAM (passwd): a password that does not meet the quality rules in /etc/security/pwquality.conf is now refused.

**How to change it.** Set `ol10stig_accounts_password_pam_enforce_root: false` before the first run. If the role has already run, delete the `enforce_for_root` line from /etc/security/pwquality.conf.

**Effect on compliance.** With `ol10stig_accounts_password_pam_enforce_root` set to false, the role no longer remediates that STIG rule, and an OpenSCAP scan reports it as failing unless the host already meets it.

### rsyslog logs remote-access activity (auth, authpriv, daemon)

_Requirement: ComplianceAsCode rule `rsyslog_remote_access_monitoring`, selected by the STIG profile for this product._

**What the role does.** Where rsyslog is installed, adds `auth.* /var/log/secure`, `authpriv.* /var/log/secure` and `daemon.* /var/log/messages` to /etc/rsyslog.conf if the line is not there. It checks /etc/rsyslog.conf and every *.conf file in /etc/rsyslog.d for a line for each facility, and adds the line when any one of them has none, so a facility already logged from another file can be logged twice.

**Who is affected.** Any host running rsyslog: authentication and daemon messages (logins, sudo, sshd) are also written to /var/log/secure and /var/log/messages, which grow faster, and a host that forwards those files sends more.

**How to change it.** Set `ol10stig_rsyslog_remote_access_monitoring: false` before the first run. If the role has already run, delete the three lines from /etc/rsyslog.conf and restart rsyslog.

**Effect on compliance.** With `ol10stig_rsyslog_remote_access_monitoring` set to false, the role no longer remediates that STIG rule, and an OpenSCAP scan reports it as failing unless the host already meets it.

### USBGuard allows only the USB devices present when the role runs

_Requirement: ComplianceAsCode rule `usbguard_generate_policy`, selected by the STIG profile for this product._

**What the role does.** If /etc/usbguard/rules.conf is missing or empty, runs `usbguard generate-policy` and writes its output there (mode 0600): a policy that allows the USB devices connected at that moment and blocks every other one. A non-empty policy is left as it is.

**Who is affected.** A host where USB devices are connected later (keyboards, storage, security keys, smart-card readers, USB passthrough on a virtual machine): while the usbguard service runs, such a device is blocked until it is allowed. A cloud instance with no USB devices at hardening blocks every USB device.

**How to change it.** Set `ol10stig_usbguard_generate_policy: false` before the first run. If the role has already run, allow a device with `usbguard allow-device <id> -p`, or regenerate the policy with the devices attached: `usbguard generate-policy > /etc/usbguard/rules.conf`, then restart usbguard.

**Effect on compliance.** With `ol10stig_usbguard_generate_policy` set to false, the role no longer remediates that STIG rule, and an OpenSCAP scan reports it as failing unless the host already meets it.

## Install

This repository root **is** the Ansible role (Galaxy-style layout). Release metadata and the
scoring helper live under `compliance/` and are not loaded when the role runs.

From **GitHub** (public):

```yaml
# requirements.yml
roles:
  - src: https://github.com/stigready/ol10-stig
    scm: git
    version: v0.3.51   # or an immutable commit SHA
    name: ol10_stig
```

```bash
ansible-galaxy role install -r requirements.yml -p ./roles
ansible-playbook -i inventory site.yml   # role: ol10_stig
```

## Verification status (this release)

This release carries no per-role OpenSCAP score, and no OpenSCAP results ship with it. A container
cannot evaluate most hardening rules, so the role is not scored here. To score a host you hardened
with it, see *Score it yourself*. `compliance/releases/0.3.51/manifest.json` records this.

## Reports & review

- **[reports/index.html](reports/index.html)** — HTML report index
- **[CHANGELOG.md](CHANGELOG.md)** — release notes

## Score it yourself

This repository ships no OpenSCAP results (see *Verification status*). To score a host you
hardened with this role, run `oscap xccdf eval` on it and score the `results.xml` with this role's
exception list: see **[compliance/README.md](compliance/README.md)**.

## Notices

StigForge and StigReady are independent. They are not affiliated with, endorsed by or certified by the Center for Internet Security, Inc. (CIS), the Defense Information Systems Agency (DISA), the U.S. Department of Defense, NIST, Oracle or the ComplianceAsCode project. "CIS" and "CIS Benchmarks" are trademarks of the Center for Internet Security, Inc. DISA STIGs are DISA publications. Oracle and Oracle Linux are registered trademarks of Oracle and/or its affiliates. Linux is the registered trademark of Linus Torvalds in the U.S. and other countries. All trademarks are the property of their respective owners. This role applies ComplianceAsCode (SSG 0.1.82) remediation from SSG's DRAFT - STIG for Oracle Linux 10 profile. This is SSG's DRAFT profile; SSG says it is not based on a DISA STIG. An OpenSCAP score is a measurement, not a certification.

**Login banner.** `login_banner_contents` in `defaults/main.yml` defaults to the DoD Notice and Consent Banner (a U.S. Government information system warning). If the system is not a U.S. Government system, override it with your organization's own text. `dconf_login_banner_contents` defaults to "Authorized users only. All activity may be monitored and reported." Overriding the banner text makes the OpenSCAP banner rules fail unless the scan's `login_banner_text` value is tailored to match it or those rules are recorded as exceptions.

## License

Copyright (c) 2026 StigReady.

- **[LICENSE](LICENSE)** (MIT) — StigForge export packaging
- **[NOTICE](NOTICE)** — ComplianceAsCode / BSD-3-Clause task body attribution

## Catalog

- [https://stigready.com/#stigforge](https://stigready.com/#stigforge)

