## Give an IP address for the router

Use this command when the router is on

```bash
ip addr add 10.0.0.1/24 dev eth0
ip link set eth0 up
```

to make the change permanent, copy this following snippet in `/etc/network/interfaces` or add this on `edit config` in the gns3 interface


```txt
auto eth0
iface eth0 inet static
	address 10.0.0.1
	netmask 255.255.255.0
	up ip link set eth0 up
```
## Setup the VXLAN tunnel

```Bash
# Create the VXLAN interface
ip link add vxlan10 type vxlan id 10 remote 10.0.0.2 local 10.0.0.1 dstport 4789 dev eth0
ip link add vxlan10 type vxlan id 10 remote 10.0.0.2 local 10.0.0.1 dstport 8472 dev eth0
ip link add vxlan10 type vxlan id 10 group 239.1.1.10 dev eth0 dstport 4789

# Bring it up
ip link set vxlan10 up

# Create a bridge
ip link add br0 type bridge
ip link set br0 up

# Add the host-facing interface to the bridge (replace eth1 with your actual interface name)
ip link set eth1 up
ip link set eth1 master br0

brctl addif br0 eth1
brctl addif br0 vxlan10
ip link set vxlan10 master br0

#table FDB
bridge fdb append 00:00:00:00:00:00 dev vxlan10 dst 10.1.1.2
```
