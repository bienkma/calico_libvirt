# calico_libvirt
calico libvirt

# Resource
- Centos 8
- libvirt 8.0.0
- calicoctl v3.20.6 4b4d6b45
- calico-felix v3.27.3 638464f946657417dd4900724112eb844ce5be03

# Architecture
<table>
<tr><th>Node Name</th><th>Node Description</th><th>Node Networks</th></tr>
<tr><td>lab01,lab02,lab03</td><td>linux machine playing the role of a router</td><td><ul><li>10.110.68.113</li><li>10.110.68.114</li><li>10.110.68.115</li></ul></td></tr>
<tr><td>lab04</td><td>linux machine playing the role of a hypervisor</td><td><ul><li>10.110.68.116</li></ul></td></tr>
<tr><td>lab05</td><td>linux machine playing the role of a hypervisor. It contains vm01</td><td><ul><li>10.110.68.117</li></ul></td></tr>
<tr><td>lab06</td><td>linux machine playing the role of a router.It contains vm02</td><td><ul><li>10.110.68.118</li></ul></td></tr>
</table>

# Setup
* Install Cumulus Quagga packages on router node
```bash
! /etc/quagga/bgpd.conf
!
router bgp 65001
 bgp router-id 10.110.68.118
 neighbor 10.110.68.116 remote-as 65002
 neighbor 10.110.68.117 remote-as 65003
 !
 address-family ipv4 unicast
  neighbor 10.110.68.116 activate
  neighbor 10.110.68.117 activate
 exit-address-family
!
line vty
! 
```

* Install etcd cluster on etcd node
  * etcd01
  * etcd02
  * etcd03
* Install calico-felix on compute node
  * lab04
```bash
# /etc/calico/felix.cfg
FELIX_DATASTORETYPE=etcdv3
FELIX_LOGFILEPATH=/var/log/felix.log
FELIX_PROMETHEUSMETRICSPORT=9091
FELIX_PROMETHEUSMETRICSENABLED=true
FELIX_PROMETHEUSPROCESSMETRICSENABLED=true
FELIX_PROMETHEUSWIREGUARDMETRICSENABLED=false
FELIX_VXLANENABLED=false
FELIX_HEALTHHOST=10.110.68.116
FELIX_PROMETHEUSMETRICSHOST=10.110.68.116
FELIX_DEVICEROUTESOURCEADDRESS=10.110.68.116
FELIX_DISABLECONNTRACKINVALIDCHECK=false
FELIX_ETCDCAFILE=/etc/calico/ssl/ca.pem
FELIX_ETCDCERTFILE=/etc/calico/ssl/etcd.pem
FELIX_ETCDKEYFILE=/etc/calico/ssl/etcd-key.pem
ETCD_CERT_FILE=/etc/calico/ssl/etcd.pem
ETCD_KEY_FILE=/etc/calico/ssl/etcd-key.pem
ETCD_CA_CERT_FILE=/etc/calico/ssl/ca.pem
FELIX_FAILSAFEINBOUNDHOSTPORTS=2379,2380
FELIX_FAILSAFEOUTBOUNDHOSTPORTS=2379,2380
FELIX_LOGSEVERITYFILE=Info
FELIX_LOGSEVERITYSCREEN=Info
FELIX_LOGSEVERITYSYS=Info
FELIX_REPORTINGINTERVALSECS=30
FELIX_IPTABLESREFRESHINTERVAL=120
FELIX_ROUTEREFRESHINTERVAL=120
ETCD_ENDPOINTS=https://10.110.68.113:2379,https://10.110.68.114:2379,https://10.110.68.115:2379
FELIX_ETCDENDPOINTS=https://10.110.68.113:2379,https://10.110.68.114:2379,https://10.110.68.115:2379
```
```azure
ip tuntap add dev calic0a8fe0a mode tap
ip link set dev calic0a8fe0a up
```
```azure
<interface type='ethernet' name='calic0a8fe0a'>
  <start mode='onboot'/>
  <protocol family='ipv4'>
    <ip address='192.168.25.10' prefix='24'/>
    <route gateway='192.168.25.1'/>
  </protocol>
  <protocol family='ipv6'>
    <autoconf/>
  </protocol>
  <alias name='eth0'/>
</interface>
```
  * lab05
```bash
# /etc/calico/felix.cfg
FELIX_DATASTORETYPE=etcdv3
FELIX_LOGFILEPATH=/var/log/felix.log
FELIX_PROMETHEUSMETRICSPORT=9091
FELIX_PROMETHEUSMETRICSENABLED=true
FELIX_PROMETHEUSPROCESSMETRICSENABLED=true
FELIX_PROMETHEUSWIREGUARDMETRICSENABLED=false
FELIX_VXLANENABLED=false
FELIX_HEALTHHOST=10.110.68.117
FELIX_PROMETHEUSMETRICSHOST=10.110.68.117
FELIX_DEVICEROUTESOURCEADDRESS=10.110.68.117
FELIX_DISABLECONNTRACKINVALIDCHECK=false
FELIX_ETCDCAFILE=/etc/calico/ssl/ca.pem
FELIX_ETCDCERTFILE=/etc/calico/ssl/etcd.pem
FELIX_ETCDKEYFILE=/etc/calico/ssl/etcd-key.pem
ETCD_CERT_FILE=/etc/calico/ssl/etcd.pem
ETCD_KEY_FILE=/etc/calico/ssl/etcd-key.pem
ETCD_CA_CERT_FILE=/etc/calico/ssl/ca.pem
FELIX_FAILSAFEINBOUNDHOSTPORTS=2379,2380
FELIX_FAILSAFEOUTBOUNDHOSTPORTS=2379,2380
FELIX_LOGSEVERITYFILE=Info
FELIX_LOGSEVERITYSCREEN=Info
FELIX_LOGSEVERITYSYS=Info
FELIX_REPORTINGINTERVALSECS=30
FELIX_IPTABLESREFRESHINTERVAL=120
FELIX_ROUTEREFRESHINTERVAL=120
ETCD_ENDPOINTS=https://10.110.68.113:2379,https://10.110.68.114:2379,https://10.110.68.115:2379
FELIX_ETCDENDPOINTS=https://10.110.68.113:2379,https://10.110.68.114:2379,https://10.110.68.115:2379
```
```azure
ip tuntap add dev calic0a8fe0a mode tap
ip link set dev calic0a8fe0a up
```
```azure
<interface type='ethernet' name='calic0a8fe0a'>
  <start mode='onboot'/>
  <protocol family='ipv4'>
    <ip address='192.168.25.11' prefix='24'/>
    <route gateway='192.168.25.1'/>
  </protocol>
  <protocol family='ipv6'>
    <autoconf/>
  </protocol>
  <alias name='eth0'/>
</interface>
```