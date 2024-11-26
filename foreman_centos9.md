# Foreman / CentOS Stream 9
Installing Foreman on CentOS Stream 9 for managing my lab

## Prerequisites:
1. Install CentOS Stream 9
* static IP address
* minimal server
* default disk layout
* with a root password
* allow root access via SSH



## Install Foreman without Puppet, and with Ansible and Katello
1. ssh into server as root
1. `dnf -y update`
1. `dnf -y install https://yum.theforeman.org/releases/3.13/el9/x86_64/foreman-release.rpm`
1. `dnf -y install https://yum.theforeman.org/katello/4.15/katello/el9/x86_64/katello-repos-latest.rpm`
1. `dnf -y install https://yum.puppet.com/puppet8-release-el-9.noarch.rpm`
1. `dnf -y install foreman-installer-katello`
1. `dnf -y update`

1. `firewall-cmd --add-service foreman --add-service foreman-proxy --add-service=tftp && firewall-cmd --runtime-to-permanent`

1. 

```
foreman-installer --scenario katello \
--foreman-initial-organization "MJSX" \
--foreman-initial-location "MJSX Lab" \
--enable-foreman-plugin-ansible \
--enable-foreman-proxy-plugin-ansible \
--puppet-server=false \
--foreman-proxy-puppet=false \
--foreman-proxy-puppetca=false \
--no-enable-puppet \
--enable-foreman-plugin-discovery \
--enable-foreman-proxy-plugin-discovery  \
--foreman-proxy-plugin-discovery-enabled=true \
--foreman-proxy-plugin-discovery-install-images=true \
--foreman-proxy-dhcp=true \
--foreman-proxy-tftp=true \
--foreman-proxy-tftp-managed=true \
--foreman-proxy-tftp-servername=172.16.13.2 \
--foreman-proxy-dhcp=true \
--foreman-proxy-dhcp-managed=true \
--foreman-proxy-dhcp-range="172.16.13.130 172.16.13.250" \
--foreman-proxy-dhcp-interface=enp86s0 \
--foreman-proxy-dhcp-gateway=172.16.13.1 \
--foreman-proxy-dhcp-nameservers="172.16.13.2" \
--foreman-proxy-dns=true \
--foreman-proxy-dns-managed=true \
--foreman-proxy-dns-zone=lab.mjsx.io \
--foreman-proxy-dns-reverse=13.16.172.in-addr.arpa \
--foreman-proxy-dns-forwarders=172.16.13.1 \
--foreman-proxy-httpboot=true \
--foreman-proxy-http=true \
--foreman-proxy-dhcp-pxeserver=172.16.13.2 \
--enable-foreman-plugin-remote-execution-cockpit \
--foreman-proxy-bmc=true \
;
```
1. Create subnet
1. Create hostgroup
1. link OS, media, kickstart, pxe templates
1. copy foreman-proxy ssh key to foreman's root ssh authorized_keys
1. add foreman host object to subnet
