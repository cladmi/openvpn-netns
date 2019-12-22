Openvpn-netns
=============

Run openvpn inside a network namespace so it can be used only by some
applications.

I wrote this version after reading
[pekman/openvpn-netns](https://github.com/pekman/openvpn-netns)

There was a lot of handling in the 'netns' script that I thought could be done
with the `--iproute` option.
Now only the 'resolv.conf' handling is kept as '--up' script.
