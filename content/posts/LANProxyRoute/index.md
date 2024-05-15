+++
title = "Routing all of my home network through proxy"
date = "2024-05-16T01:00:00+0330"
+++

## Why?

I live in a country with heavy internet censorship. This means that in order
to access the free internet, everyone in the home network has to turn on some sort
of proxy. This comes with a hefty cost of extra load on any device in our home, whether it
is:

- **Extra stress on mobile batteries**: constantly running VPN profiles on mobile phones make them
  run hotter and significantly impacts their battery life.

- **Poor proxy apps on different platforms**: There is numerous amount of headaches that follow with
  mobile proxy apps. Some don't work properly, some are bloated, some are too heavy on the device.

- **Compatibility**: Some devices like
  Android TVs don't support certain GFW-resistent protocols.

## How it all started

This initially started with me installing Debian on an old PC laying around at the back of my bed.
I was just goofing around and I wanted to have a working home GNU/Linux server. Debian is great for
servers. It installed gracefully with no headaches and was ready to go in less than 15 minutes. I
always wanted to make the whole process of connecting family members to the free internet easier.

So, here I started my journey of looking for a solution.

<!-- ```sh -->
<!-- iptables -I INPUT -j ACCEPT -->
<!-- iptables -A FORWARD -i wls32 -j ACCEPT -->

<!-- iptables -t nat -A OUTPUT -d 192.168.0.0/16 -j RETURN -->
<!-- iptables -t nat -A OUTPUT -d 0.0.0.0/8 -j RETURN -->

<!-- #iptables -t nat -A PREROUTING -p tcp --dport 22 -j ACCEPT -->
<!-- #iptables -t nat -A OUTPUT -p tcp --dport 22 -j ACCEPT -->

<!-- #iptables -t nat -A PREROUTING -p tcp ! --dport 22 -j REDIRECT --to-ports 12345 -->
<!-- #iptables -t nat -A OUTPUT -p tcp ! --dport 22 -j REDIRECT --to-ports 12345 -->
<!-- #iptables -t nat -A PREROUTING -p udp ! --dport 22 -j REDIRECT --to-ports 10053 -->
<!-- #iptables -t nat -A OUTPUT -p udp ! --dport 22 -j REDIRECT --to-ports 10053 -->

<!-- iptables -t nat -A PREROUTING -p tcp --dport 443 -j REDIRECT --to-ports 12345 -->
<!-- iptables -t nat -A OUTPUT -p tcp --dport 443 -j REDIRECT --to-ports 12345 -->
<!-- iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-ports 12345 -->
<!-- iptables -t nat -A OUTPUT -p tcp --dport 80 -j REDIRECT --to-ports 12345 -->
<!-- iptables -t nat -A OUTPUT -p udp --dport 53 -j REDIRECT --to-ports 6450 -->
<!-- iptables -t nat -A PREROUTING -p udp --dport 53 -j REDIRECT --to-ports 6450 -->

<!-- #iptables -t nat -A PREROUTING -p tcp -j REDIRECT --to-ports 12345 -->
<!-- #iptables -t nat -A OUTPUT -p tcp -j REDIRECT --to-ports 12345 -->
<!-- iptables -t nat -A POSTROUTING -p tcp -o wls32 -j MASQUERADE -->
<!-- ``` -->
