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

Continue 
