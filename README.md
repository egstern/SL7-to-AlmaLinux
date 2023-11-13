# SL7-to-AlmaLinux
Convert an SL7 to AlmaLinux

## NEW! See the bottom for a what happened on a real live working system!

# WARNING!
# This is a dangerous procedure! Things can go wrong. You have been warned. This procedure was minimally tested.

This procedure first converts an SL7 system to CentOS-7 and then upgrades to AlmaLinux-8 using the ELevate tools https://wiki.almalinux.org/elevate/.

## FAQ
1. Who is this for?
   - This document provides an migration path for someone with a active SL-7 machine who wishes to preserve its operations past the EOL of SL-7 in June 2024 but doesn't want to do a full install of some other operating system. A full install often involves disk partition reformatting causing complications with data preservation. This procedure lets them update the operating system in place to AlmaLinux-8 leaving their user files and data undisturbed and preserving many system settings unchanged.
2. Why can't they just use the ELevate tools directly?
   - They don't work for SL-7, as of 2024-10-26. See https://bugs.almalinux.org/view.php?id=433

## First convert to CentOS-7

To convert an SL7 system to AlmaLinux-8 using the ELevate tools, I first had to
convert to CentOS-7.
The procedure was a little tricky because doing something in the wrong order resulted
in an unusable system or a system with leftover SL bits that prevented a clean upgrade.

I think the it is safest to do this conversion in text mode.
To bring a running system into text mode use
```
sudo systemctl isolate multi-user
```
or set the system to boot into text mode with
```
sudo systemctl set-default multi-user
```

Download and save CentOS rpms. Use either a browser or `wget` to get these files:
```
  https://linux-mirrors.fnal.gov/linux/centos/7.9.2009/updates/x86_64/Packages/centos-release-7-9.2009.1.el7.centos.x86_64.rpm
  https://linux-mirrors.fnal.gov/linux/centos/7.9.2009/os/x86_64/Packages/centos-logos-70.0.6-3.el7.centos.noarch.rpm

```
Remove rpms that define the system as "SL". You can't use yum for this because that will remove almost the entire system leaving an unrecoverable system.
```
sudo rpm -e sl-release sl-logos yum-conf-sl7x yum-conf-extras yum-conf-repos --nodeps
```

Install the previously downloaded CentOS-7 rpms. Some directories need to be removed first and replaced by plain files for this to work.
```
sudo rmdir /usr/share/redhat-release /usr/share/doc/redhat-release
sudo touch /usr/share/redhat-release /usr/share/doc/redhat-release
sudo rpm -i centos*.rpm
```
It will probably give a warning similar to:
```
warning: centos-logos-70.0.6-3.el7.centos.noarch.rpm: Header V3 RSA/SHA256 Signature, key ID f4a80eb5: NOKEY
```
but this is harmless.

Upgrade the rest of the rpms to CentOS-7 rpms:
```
sudo yum distro-sync -y
```

Now reboot into the CentOS-7 system!
```
sudo systemctl reboot
```
Even though the GRUB menu still shows SL7 entries, the new system when booted will be CentOS-7.

Verify that rpm and yum system is not terribly broken.
```
sudo yum upgrade
```
should complete without errors.


## Continue to Upgrade to AlmaLinux-8

Follow the instructions here: https://wiki.almalinux.org/elevate/ELevate-quickstart-guide.html

```
sudo yum install -y http://repo.almalinux.org/elevate/elevate-release-latest-el$(rpm --eval %rhel).noarch.rpm
sudo yum install -y leapp-upgrade leapp-data-almalinux
sudo leapp preupgrade
```
This results in a report about the likelyhood of upgrade success. My system gave me this:
```
============================================================
                     UPGRADE INHIBITED                      
============================================================

Upgrade has been inhibited due to the following problems:
    1. Inhibitor: Missing required answers in the answer file
Consult the pre-upgrade report for details and possible remediation.

============================================================
                     UPGRADE INHIBITED                      
============================================================


Debug output written to /var/log/leapp/leapp-preupgrade.log

============================================================
                           REPORT                           
============================================================

A report has been generated at /var/log/leapp/leapp-report.json
A report has been generated at /var/log/leapp/leapp-report.txt

============================================================
                       END OF REPORT                        
============================================================

```

There was only one issue preventing upgrade noted in `leapp-report.txt`:
```
Title: Module pam_pkcs11 will be removed from PAM configuration
```
The file `/var/log/answerfile` should be edited to confirm deletion of this module so it looks like this:
```
# =================== remove_pam_pkcs11_module_check.confirm ==================
# Label:              Disable pam_pkcs11 module in PAM configuration? If no, th$
# Description:        PAM module pam_pkcs11 is no longer available in RHEL-8 si$
# Reason:             Leaving this module in PAM configuration may lock out the$
# Type:               bool
# Default:            None
# Available choices: True/False
# Unanswered question. Uncomment the following line with your answer
confirm = True
```
Then the upgrade is performed:
```
sudo leapp upgrade
```

After a bunch of grinding, it will indicate that the system should be rebooted. From the command line, you can use this command:
```
sudo systemctl reboot
```

The system will reboot and begin installing a lot of packages. It will need to reboot at least once more. One of those reboots will be to relabel the disk for SElinux.

At the end of all of that, the system will reboot and come up running AlmaLinux 8.

Verify that the dnf system is not terribly broken.
```
sudo dnf upgrade
```
should complete without errors.

# Caveats
This was tested on a system without other repositories such as `epel`. If your original system is running a desktop based on packages from `epel` or kernel packages from other sources (ELrepo or nVidia) there may be trouble. See below for an example converting a system that had been in active use.


## One experience converting a working SL7 system

The conversion from SL7 to CentOS7 when flawlessly.

The leapp upgrade script had problems inhibitin the upgrade.

```
Title: OpenSSH configured to allow root login
Summary: OpenSSH is configured to deny root logins in match blocks, but not explicitly enabled in global or "Match all" context. This update changes the default to disable root logins using paswords so your server might become inaccessible.
Remediation: [hint] Consider using different user for administrative logins or make sure your configration file contains the line "PermitRootLogin yes" in global context if desired.
Key: 28a7143121e421ec71b80bd7a652173f5561f757
----------------------------------------
Risk Factor: high (inhibitor)
Title: Multiple devel kernels installed
Summary: DNF cannot produce a valid upgrade transaction when multiple kernel-devel packages are installed.
Remediation: [hint] Remove all but one kernel-devel packages before running Leapp again.
[command] yum -y remove kernel-devel-3.10.0-1160.95.1.el7 kernel-devel-3.10.0-1160.99.1.el7
Key: 8ceea81afbbb1a329b7d82ca7212b21509e5b256
----------------------------------------
Risk Factor: high (inhibitor)
Title: Missing required answers in the answer file
Summary: One or more sections in answerfile are missing user choices: remove_pam_krb5_module_check.confirm
For more information consult https://leapp.readthedocs.io/en/latest/dialogs.html
Remediation: [hint] Please register user choices with leapp answer cli command or by manually editing the answerfile.
[command] leapp answer --section remove_pam_krb5_module_check.confirm=True
Key: 03f345f78710ed205db60143ad9ff58f37658fbf
----------------------------------------
Risk Factor: high (inhibitor)
Title: Missing required answers in the answer file
Summary: One or more sections in answerfile are missing user choices: remove_pam_pkcs11_module_check.confirm
For more information consult https://leapp.readthedocs.io/en/latest/dialogs.html
Remediation: [hint] Please register user choices with leapp answer cli command or by manually editing the answerfile.
[command] leapp answer --section remove_pam_pkcs11_module_check.confirm=True
Key: d35f6c6b1b1fa6924ef442e3670d90fa92f0d54b
----------------------------------------
```

I added PermitRootLogin="yes" to `/etc/ssh/sshd_config` though it is against policy but I did not restart `sshd` so it did not take effect.
I confirmed the requested changes in the `answerfile`

```
[remove_pam_krb5_module_check]
# Title:              None
# Reason:             Confirmation
# ==================== remove_pam_krb5_module_check.confirm ===================
# Label:              Disable pam_krb5 module in PAM configuration? If no, the upgrade process will be interrupted.
# Description:        PAM module pam_krb5 is no longer available in RHEL-8 since it was replaced by SSSD.
# Reason:             Leaving this module in PAM configuration may lock out the system.
# Type:               bool
# Default:            None
# Available choices: True/False
# Unanswered question. Uncomment the following line with your answer
confirm = True

[remove_pam_pkcs11_module_check]
# Title:              None
# Reason:             Confirmation
# =================== remove_pam_pkcs11_module_check.confirm ==================
# Label:              Disable pam_pkcs11 module in PAM configuration? If no, the upgrade process will be interrupted.
# Description:        PAM module pam_pkcs11 is no longer available in RHEL-8 since it was replaced by SSSD.
# Reason:             Leaving this module in PAM configuration may lock out the system.
# Type:               bool
# Default:            None
# Available choices: True/False
# Unanswered question. Uncomment the following line with your answer
confirm = True
```

The second run of leapp upgrade failed because of file conflicts between
AlmaLinux8 rpms and existing files in python36 modules that I had installed
on the SL7 system. These took the form of

```
  file /usr/lib64/python3.6/site-packages/gi/__pycache__/_constants.cpython-36.opt-1.pyc from install of python3-gobject-base-3.28.3-2.el8.x86_64 conflicts with file from
package python36-gobject-base-3.22.0-6.el7.x86_64
```
and many others.

Also files from newer versions of `cmake` that I had installed on the SL7 system conflicted with the AlmaLinux8 versions
```
  file /usr/bin/ccmake3 from install of cmake-3.20.2-5.el8.x86_64 conflicts with file from package cmake3-3.17.5-1.el7.x86_64
  file /usr/bin/cmake3 from install of cmake-3.20.2-5.el8.x86_64 conflicts with file from package cmake3-3.17.5-1.el7.x86_64
  file /usr/bin/cpack3 from install of cmake-3.20.2-5.el8.x86_64 conflicts with file from package cmake3-3.17.5-1.el7.x86_64
  file /usr/bin/ctest3 from install of cmake-3.20.2-5.el8.x86_64 conflicts with file from package cmake3-3.17.5-1.el7.x86_64
```

 I removed the conflicting packages with `dnf` and restarted the upgrade.

This time the script completed installed some packages and initiating a
reboot.

The system reboots into a procedure which installs the bulk of the new
packages. After a long time installing and cleaning up about 4000 rpms it
rebooted again.

This time the reboot failed ending in a `grub>` prompt.

I booted a install disk in rescue mode to look at the log file. It indicated
that the boot loader couldn't be installed because there was no room in the
boot region. I manually tried to install grub with grub2-install and it
repeated the error message about being unable to install the boot loader. I
looked at the disk partition layout with fdisk. There was a Dell utility
partition starting at sector 63 which is an unusually small number. I didn't
need that the Dell partition so I deleted it creating more space. This time
grub2-install worked. I was able to reboot the system.

The system booted up into a login screen.

I normally log into the system with my Kerberos password but that wasn't
working. I didn't have a local password on the system. I booted the rescue
system again to create a local password. Then I was able to log in and
install the Fermilab custom rpms.

So finally, the system is usable, running in AlmaLinux-8. I could log into
it with ssh. The main missing capability is the ability to log in with a
Kerberos password.

