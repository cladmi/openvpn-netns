Openvpn-netns
=============

Run openvpn inside a network namespace so it can be used only by some
applications.

I wrote this version after reading
[pekman/openvpn-netns](https://github.com/pekman/openvpn-netns)

There was a lot of handling in the 'netns' script that I thought could be done
with the `--iproute` option.
Now only the 'resolv.conf' handling is kept as '--up' script.

Usage
-----

    sudo openvpn \
        --setenv NETNS vpnns \
        --iproute ./netns-ip \
        --up ./netns-nameserver \
        --script-security 2 \
        --config vpn.config

Design
------

Creating the network namespace, running the application, running the VPN are
3 different steps.

I would like to have them as 3 different systemd scripts so they could be
re-started.

Current state
-------------

* Openvpn configuration seems to be working

Implementation
--------------

* openvpn uses an '--iproute' script to redirect all commands to the namespace.
  It also create the namespace for simplicity.
* The 'resolv.conf' file is handled by the '--up' script.
  'domain/search' configuration is ignored as I do not need it.

TODO
----

* Run an application and allow accessing one tcp port.
* Systemd files
  * netns creation?
  * openvpn
  * application
* Automation for testing?
