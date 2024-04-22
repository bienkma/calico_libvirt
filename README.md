# calico_libvirt
calico libvirt

# Resource
- Centos 8
- libvirt 8.0.0
- calicoctl v3.20.6 4b4d6b45
- calico-felix v3.27.3 638464f946657417dd4900724112eb844ce5be03
- execute the command on all nodes:
```bash
systemctl stop firewalld
systemctl disable firewalld
iptables -F && iptables -X
```

# Architecture
<table>
<tr><th>Node Name</th><th>Node Description</th><th>Node Networks</th></tr>
<tr><td>lab01,lab02,lab03</td><td>linux machine playing the role of a router</td><td><ul><li>10.110.68.113</li><li>10.110.68.114</li><li>10.110.68.115</li></ul></td></tr>
<tr><td>lab04</td><td>linux machine playing the role of a hypervisor</td><td><ul><li>10.110.68.116</li></ul></td></tr>
<tr><td>lab05</td><td>linux machine playing the role of a hypervisor. It contains vm01</td><td><ul><li>10.110.68.117</li></ul></td></tr>
<tr><td>lab06</td><td>linux machine playing the role of a router.It contains vm02</td><td><ul><li>10.110.68.118</li></ul></td></tr>
</table>

# Setup
* Install Cumulus Quagga packages on lab06
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
  * make ssl for etcd TLS
```bash
mkdir cfssl
cd cfssl
touch ca-config.json
touch ca-csr.json  
```
```json
//ca-config.json
{
    "signing": {
        "default": {
            "expiry": "87600h"
        },
        "profiles": {
            "server": {
                "expiry": "87600h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "server auth"
                ]
            },
            "client": {
                "expiry": "87600h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "client auth"
                ]
            },
            "peer": {
                "expiry": "87600h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "server auth",
                    "client auth"
                ]
            }
        }
    }
}
```
```json
//ca-csr.json
{
  "CN": "etcd-cluster",
  "hosts": [
    "calico-etcd1",
    "calico-etcd2",
    "calico-etcd3"
  ],
  "key": {
    "algo": "ecdsa",
    "size": 256
  },
  "names": [
    {
      "C": "US",
      "L": "CA",
      "ST": "San Francisco"
    }
  ]
}
```
```bash
cfssl gencert -initca ca-csr.json | cfssljson -bare ca -

echo '{"CN":"etcd-cluster","hosts":[""],"key":{"algo":"rsa","size":2048}}' | cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=peer -hostname="10.110.68.113,10.110.68.114,10.110.68.115,127.0.0.1,calico-etcd1,calico-etcd2,calico-etcd3" - | cfssljson -bare etcd
```
  * etcd01
```yaml
data-dir: /var/lib/etcd
name: calico-etcd1
initial-advertise-peer-urls: https://10.110.68.113:2380
listen-peer-urls: https://10.110.68.113:2380
advertise-client-urls: https://10.110.68.113:2379
listen-client-urls: https://10.110.68.113:2379
initial-cluster-state: new
auto-compaction-retention: '8'
initial-cluster: calico-etcd1=https://10.110.68.113:2380,calico-etcd2=https://10.110.68.114:2380,calico-etcd3=https://10.110.68.115:2380
client-transport-security:
  cert-file: /etc/etcd/ssl/etcd.pem
  key-file: /etc/etcd/ssl/etcd-key.pem
  trusted-ca-file: /etc/etcd/ssl/ca.pem
peer-transport-security:
  cert-file: /etc/etcd/ssl/etcd.pem
  key-file: /etc/etcd/ssl/etcd-key.pem
  trusted-ca-file: /etc/etcd/ssl/ca.pem
```
  * etcd02
```yaml
data-dir: /var/lib/etcd
name: calico-etcd2
initial-advertise-peer-urls: https://10.110.68.114:2380
listen-peer-urls: https://10.110.68.114:2380
advertise-client-urls: https://10.110.68.114:2379
listen-client-urls: https://10.110.68.114:2379
initial-cluster-state: new
auto-compaction-retention: '8'
initial-cluster: calico-etcd1=https://10.110.68.113:2380,calico-etcd2=https://10.110.68.114:2380,calico-etcd3=https://10.110.68.115:2380
client-transport-security:
  cert-file: /etc/etcd/ssl/etcd.pem
  key-file: /etc/etcd/ssl/etcd-key.pem
  trusted-ca-file: /etc/etcd/ssl/ca.pem
peer-transport-security:
  cert-file: /etc/etcd/ssl/etcd.pem
  key-file: /etc/etcd/ssl/etcd-key.pem
  trusted-ca-file: /etc/etcd/ssl/ca.pem
```
  * etcd03
```yaml
data-dir: /var/lib/etcd
name: calico-etcd3
initial-advertise-peer-urls: https://10.110.68.115:2380
listen-peer-urls: https://10.110.68.115:2380
advertise-client-urls: https://10.110.68.115:2379
listen-client-urls: https://10.110.68.115:2379
initial-cluster-state: new
auto-compaction-retention: '8'
initial-cluster: calico-etcd1=https://10.110.68.113:2380,calico-etcd2=https://10.110.68.114:2380,calico-etcd3=https://10.110.68.115:2380
client-transport-security:
  cert-file: /etc/etcd/ssl/etcd.pem
  key-file: /etc/etcd/ssl/etcd-key.pem
  trusted-ca-file: /etc/etcd/ssl/ca.pem
peer-transport-security:
  cert-file: /etc/etcd/ssl/etcd.pem
  key-file: /etc/etcd/ssl/etcd-key.pem
  trusted-ca-file: /etc/etcd/ssl/ca.pem
```
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
* setup workloadEndpoint, policy and ippool
```azure
calicoctl apply -f pool1.yaml
calicoctl apply -f calicoctl/wide-open.yaml
calicoctl apply -f calicoctl/workloadEndpoint-lab04.yaml
calicoctl apply -f calicoctl/workloadEndpoint-lab05.yaml
```

* setup libvirt on lab04
```azure
virt-install --name vm01 \
--os-type linux \
--os-variant ubuntu20.04 \
--ram 1024 \
--disk /kvm/disk/vm01.img,device=disk,bus=virtio,size=10,format=qcow2 \
--noautoconsole \
--graphics vnc,listen=10.110.68.116 \
--hvm \
--cdrom /kvm/iso/ubuntu-22.04.2-live-server-amd64.iso \
--boot cdrom,hd \
--network network=calic0a8fe0a
```
* setup libvirt on lab05
```azure
virt-install --name vm02 \
--os-type linux \
--os-variant ubuntu20.04 \
--ram 1024 \
--disk /kvm/disk/vm02.img,device=disk,bus=virtio,size=10,format=qcow2 \
--noautoconsole \
--graphics vnc,listen=10.110.68.117 \
--hvm \
--cdrom /kvm/iso/ubuntu-22.04.2-live-server-amd64.iso \
--boot cdrom,hd \
--network network=calic0a8fe0a
```