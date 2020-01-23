Openvpn-netns
=============

Run openvpn to configure network inside a network namespace so it can be used
only by some applications.

The idea is that openvpn is run on the host, but puts the tun interface `up`
inside the network namespace. The ip configuration will also be done in the
namespace.
For the dns, edit the the network namespace resolv.conf file stored in
`/etc/netns/NAMESPACENAME/resolv.conf`.

I wrote this version after reading
[pekman/openvpn-netns](https://github.com/pekman/openvpn-netns)

There was a lot of handling in the 'netns' script that I thought could be done
with the `--iproute` option.
Now only the 'resolv.conf' handling is kept as '--up' script.

https://discourse.nixos.org/t/run-systemd-service-in-network-namespace/3179/6
https://github.com/systemd/systemd/issues/2741 Many examples

Openvpn usage
-------------

    sudo ip netns add vpnns
    sudo openvpn \
        --setenv NETNS vpnns \
        --iproute ./openvpn-netns-ip \
        --up ./openvpn-netns-nameserver \
        --script-security 2 \
        --config vpn.conf
    # Wait for the vpn to be configured

### Checking ###

    # In another terminal show that the vpn is working inside:
    sudo ip netns exec vpnns ip route
    default via 4.3.2.1 dev tun0
    4.3.2.1/27 dev tun0 proto kernel scope link src 4.3.2.10

    sudo ip netns exec vpnns cat /etc/resolv.conf
    nameserver 4.3.2.1

    sudo ip netns exec vpnns ping -c 3 9.9.9.9
    PING 9.9.9.9 (9.9.9.9) 56(84) bytes of data.
    64 bytes from 9.9.9.9: icmp_seq=1 ttl=60 time=185 ms
    64 bytes from 9.9.9.9: icmp_seq=2 ttl=60 time=107 ms
    64 bytes from 9.9.9.9: icmp_seq=3 ttl=60 time=163 ms

    --- 9.9.9.9 ping statistics ---
    3 packets transmitted, 3 received, 0% packet loss, time 2001ms
    rtt min/avg/max/mdev = 107.032/151.707/185.270/32.892 ms

    # Traceroute goes through the vpn
    sudo ip netns exec vpnns traceroute 9.9.9.9
    traceroute to 9.9.9.9 (9.9.9.9), 30 hops max, 60 byte packets
     1  _gateway (4.3.2.1)  105.501 ms  105.840 ms  106.127 ms
     2  * * *
     3  * * *
     4  * * *
     5  dns9.quad9.net (9.9.9.9)  129.345 ms !X  49.028 ms !X  84.534 ms !X


Systemd integration
-------------------

The systemd integration is made of different units for each functionnality:

* Creating the network namespace
* openvpn service to have vpn networking inside the namespace
 * Implemented as an 'override.conf' file
* Run the application inside the workspace
* Optional:
 * socat tcp redirection to access on the host a socket in the namespace


Implementation detail
---------------------

A service creates the network namespace, and for simplicity, puts the loopback
interface up.
The application your want to run, uses this namespace using the
`NetworkNamespacePath=/var/run/netns/NETNSNAME` option and binds to the `netns`
unit.

Usage
-----

* Put the 'services' files in '/usr/lib/systemd/system'
* Edit your 'openvpn-client@CONFIG.service' to use the network namespace
 * See 'templates/openvpn-netns.conf'
* Edit your service to use the network namespace
 * See 'templates/service-netns.conf'


Current state
-------------

* Openvpn configuration
* Create network namespace as a unit
* Provide "tcp port forwarding" to the namespace using socat
* Template to run a service inside the namespace
* Example for a service that runs 'transmission' with a 'transmission'
  namespace and vpn configuration.

Implementation
--------------

* openvpn uses an '--iproute' script to redirect all commands to the namespace.
  It also create the namespace for simplicity.
* The 'resolv.conf' file is handled by the '--up' script.
  'domain/search' configuration is ignored as I do not need it.
* Network namespace configuration and port forwarding is done as separate units
  Putting the network up is still done at namespace creation for simplicity
* Forwarding of a TCP port is done using socat to not setup any extra v-eth

TODO
----

* Automation for testing?
* Documentation on how to test
