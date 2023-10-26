# SL7-to-AlmaLinux
Convert an SL7 to AlmaLinux

## First convert to CentOS-7

To Convert an SL7 system to AlmaLinux-8 using the ELevate tools, I first had to
convert to CentOS-7.
The procedure was a little tricky because doing something in the wrong order resulted
in an ususable system or a system with leftover SL bits that prevented a clean upgrade.

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
  https://linux-mirrors.fnal.gov/linux/centos/7.9.2009/updates/x86_64/Packages/centos-release-7-9.2009.1.el7.centos.x86_64.rpm \
  https://linux-mirrors.fnal.gov/linux/centos/7.9.2009/os/x86_64/Packages/centos-logos-70.0.6-3.el7.centos.noarch.rpm

```
Remove rpms that define the system as "SL":
```
sudo rpm -e sl-release sl-logos yum-conf-sl7x yum-conf-extras yum-conf-repos --nodeps
```
