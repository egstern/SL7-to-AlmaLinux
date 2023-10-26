# SL7-to-AlmaLinux
Convert an SL7 to AlmaLinux

# WARNING!
# This is a dangerous procedure! Things can go wrong. You have been warned. This procedure was minimally tested on a virtual SL7 Workstation install with no extra repositories.

This procedure first converts an SL7 system to CentOS-7 and then upgrades to AlmaLinux-8 using the ELevate tools https://wiki.almalinux.org/elevate/.

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

# Caveats
This was tested on a system without other repositories such as `epel`. If your original system is running a desktop based on packages from `epel` or kernel packages from other sources there may be trouble.



