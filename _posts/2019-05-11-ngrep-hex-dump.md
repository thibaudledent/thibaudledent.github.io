---
title: "Hex dump of network messages exchanged across the wire"
published: true
---

Our project consists in exchanging messages (in various formats, including binary) with other systems, sometimes very old. As part of our tests, it became interesting to check the messages exchanged at the network level, before they reached the application.

We were able to do this very simply with `ngrep` ("network grep"). One interesting feature of `ngrep` is its ability to display the packets it observes in a hexadecimal format, which is more effective for inspecting binary content messages.

First, we need to get the network interface name of our machine:

```bash
$ ip route show to match 10.10.10.10
default via 123.12.12.12 dev eno123 proto dhcp metric 100
```

In this example, the interface is `eno123`.

Then, to see messages exchanged with the IP `99.99.99.99` and port `1234`:

```bash
ngrep -qxtd eno123 '.' 'host 99.99.99.99 and port 1234'
```

Where the `qxtd` is for:

```
-q     Be quiet
-x     Dump packet contents as hexadecimal as well as ASCII
-t     Print a timestamp in the form of YYYY/MM/DD HH:MM:SS.UUUUUU
-d     Force ngrep to listen on interface dev
```

It outputs a packet like this cross the wire:

```
T 2019/05/11 08:52:37.611605 123.12.12.12:1234 -> 99.99.99.99:1234 [A]
  ff d8 ff e0 00 10 4a 46    49 46 00 01 02 01 00 48    ......JFIF.....H
  00 48 00 00 ff ed 13 ba    50 68 6f 74 6f 73 68 6f    .H......Photosho
  70 20 33 2e 30 00 38 42    49 4d 03 ed 00 00 00 00    p 3.0.8BIM......
  00 10 00 48 00 00 00 01    00 01 00 48 00 00 00 01    ...H.......H....
  ...
```

# References

- [ngrep usage (SourceForge)](http://ngrep.sourceforge.net/usage.html)

