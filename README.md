Openvpn-netns
=============

Run openvpn inside a network namespace so it can be used only by some
applications.

I wrote this version after reading
[pekman/openvpn-netns](https://github.com/pekman/openvpn-netns)

There was a lot of handling in the 'netns' script that I thought could be done
with the `--iproute` option.
Now only the 'resolv.conf' handling is kept as '--up' script.

https://discourse.nixos.org/t/run-systemd-service-in-network-namespace/3179/6
https://github.com/systemd/systemd/issues/2741 Many examples

Usage
-----

    sudo openvpn \
        --setenv NETNS vpnns \
        --iproute ./netns-ip \
        --up ./netns-nameserver \
        --script-security 2 \
        --config vpn.config

Put the 'services' files in '/usr/lib/systemd/system'
Edit your service to add the configuration from 'template/service-netns.conf'

Design
------

Creating the network namespace, running the application, running the VPN are
3 different steps.

I would like to have them as different systemd units so they could be
re-started.

Current state
-------------

* Openvpn configuration seems to be working
  * No Systemd integration yet
* Create network namespace as a unit
* Provide "tcp port forwarding" to the namespace using socat
* Template to run a service inside the namespace

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

* Systemd files
  * openvpn
* Automation for testing?
