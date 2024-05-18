+++
title = "Routing all of my home network through proxy"
date = "2024-05-16T01:00:00+0330"
+++

## Why?

I live in a country with heavy internet censorship. This means that
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

This initially started with me installing Debian on an old PC lying around at the back of my bed.
I was just goofing around, and I wanted to have a working home GNU/Linux server. Debian is great for
servers. It installed gracefully with no headaches and was ready to go in less than 15 minutes. I
always wanted to make the whole process of connecting family members to the free internet easier.

So, here I started my journey of looking for a solution.

# First configuration: clumsy redsocks + iptables setup

If I want to be honest, I initially just asked ChatGPT on what I wanted
to achieve, surprisingly it led me to the right path. I was presented with
a tool called [redsocks](https://darkk.net.ru/redsocks/). My knowledge on
the networking behind this tool is not enough yet, but I am guessing that
it essentially translates the TCP connections made by devices to SOCKS/HTTP
proxy.

This is exactly what I needed. Most of the GFW-resistent proxy protocols usually
run a SOCKS/HTTP proxy on your local machine that you can use it to route your traffic
through it. So you might ask:

### Why don't you just run the SOCKS/HTTP proxy on the machine and call it a day?

Well, there are two reason behind this:

1. I am just too geeky, and sometimes I have to do something for no logical reason.
   (Maybe you can call this OCD too)

2. Most of the devices don't provide a good experience when you set a _SOCKS/HTTP_
   proxy on them for some reason. And some don't even support them _at all_.

3. This would still put extra stress on mobile batteries and moreover, extra hassle
   on the users.

I don't really know 100% what is going behind the curtains in this process, so I
essentially had to try out different combinations of config files to find the working
solution.

Here is the redsocks configuration:

```c
base {
	// debug: connection progress & client list on SIGUSR1
	log_debug = on;

	// info: start and end of client session
	log_info = on;

	/* possible `log' values are:
	 *   stderr
	 *   "file:/path/to/file"
	 *   syslog:FACILITY  facility is any of "daemon", "local0"..."local7"
	 */
	log = "syslog:daemon";

	// detach from console
	daemon = on;

	/* Change uid, gid and root directory, these options require root
	 * privilegies on startup.
	 * Note, your chroot may requre /etc/localtime if you write log to syslog.
	 * Log is opened before chroot & uid changing.
	 */
	user = redsocks;
	group = redsocks;
	// chroot = "/var/chroot";

	/* possible `redirector' values are:
	 *   iptables   - for Linux
	 *   ipf        - for FreeBSD
	 *   pf         - for OpenBSD
	 *   generic    - some generic redirector that MAY work
	 */
	redirector = iptables;
}

redsocks {
	/* `local_ip' defaults to 127.0.0.1 for security reasons,
	 * use 0.0.0.0 if you want to listen on every interface.
	 * `local_*' are used as port to redirect to.
	 */
	local_ip = 0.0.0.0;
	local_port = 12345;

	// `ip' and `port' are IP and tcp-port of proxy-server
	// You can also use hostname instead of IP, only one (random)
	// address of multihomed host will be used.
	ip = 127.0.0.1;
	port = 2334;


	// known types: socks4, socks5, http-connect, http-relay
	type = socks5;
}

```

This will basically redirect any TCP connection that goes to the port
`12345` on this machine to the SOCKS server `127.0.0.1:2334`. So you will
have to have your proxy server running on `127.0.0.1:2334`.

Now that we have the redsocks configuration, we need to set up the iptables
rules so that the traffic that comes to the machine is redirected to the
redsocks.

```sh 
iptables -I INPUT -j ACCEPT 
iptables -A FORWARD -i wls32 -j ACCEPT 

iptables -t nat -A OUTPUT -d 192.168.0.0/16 -j RETURN 
iptables -t nat -A OUTPUT -d 0.0.0.0/8 -j RETURN 

#iptables -t nat -A PREROUTING -p tcp --dport 22 -j ACCEPT 
#iptables -t nat -A OUTPUT -p tcp --dport 22 -j ACCEPT 

#iptables -t nat -A PREROUTING -p tcp ! --dport 22 -j REDIRECT --to-ports 12345 
#iptables -t nat -A OUTPUT -p tcp ! --dport 22 -j REDIRECT --to-ports 12345 
#iptables -t nat -A PREROUTING -p udp ! --dport 22 -j REDIRECT --to-ports 10053 
#iptables -t nat -A OUTPUT -p udp ! --dport 22 -j REDIRECT --to-ports 10053 

iptables -t nat -A PREROUTING -p tcp --dport 443 -j REDIRECT --to-ports 12345 
iptables -t nat -A OUTPUT -p tcp --dport 443 -j REDIRECT --to-ports 12345 
iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-ports 12345 
iptables -t nat -A OUTPUT -p tcp --dport 80 -j REDIRECT --to-ports 12345 
iptables -t nat -A OUTPUT -p udp --dport 53 -j REDIRECT --to-ports 6450 
iptables -t nat -A PREROUTING -p udp --dport 53 -j REDIRECT --to-ports 6450 

#iptables -t nat -A PREROUTING -p tcp -j REDIRECT --to-ports 12345 
#iptables -t nat -A OUTPUT -p tcp -j REDIRECT --to-ports 12345 
iptables -t nat -A POSTROUTING -p tcp -o wls32 -j MASQUERADE 
```

This will basically redirect any TCP connection that goes to the port
`80` or `443` to the port `12345` which is the port that redsocks is listening.
Pay attention to the comments in the script, I just wanted to leave them here
to show how I was pulling my hair out while trying to figure out the correct
configuration.

One of the main issues with this setup was that for some reason devices connected
to this machine as a gateway were not able to do DNS queries. I just randomly
tried out routing the DNS queries (port `53`) from the devices to the port `6450` which is
the DNS server that `Hiddify-next` runs. Which 

Also, for some reason I had to **specify** the exact port I wanted to redirect and
it wouldn't work if I just redirected all the traffic to the port `12345`.

I would appreciate it if someone could explain to me why this is the case. Don't
hesitate to reach out to me via [email](mailto:miliaxe0@gmail.com).

Lastly I had to specify that the devices use this machine as a gateway. This can
be done by setting the gateway IP to the IP of the machine. You can either specify
the DHCP server to give the IP of the machine as the gateway or you can manually
set the gateway IP on the devices.

Now, this setup was working, but it was not the most elegant solution. Sometimes 
this setup would just stop working and I had to restart the machine to make it work 
and since I didn't have much knowledge on networking. This didn't satisfy my OCD.

Hereby I present you the second configuration:

# Second configuration: OpenWRT + Passwall