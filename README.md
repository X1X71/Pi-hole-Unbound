# How to set up Unbound as a recursive DNS resolver on Pi-hole 
A guide to install Pi-hole with Unbound DNS resolver for enhanced privacy and security

Important Information:


This guide assues that you have a system running a Linux distribution with Pi-hole already installed and configured.

What Is Unbound?:


Unbound is a fast, secure, and privacy-focused DNS resolver that can recursively query domain names, cache results for faster performance, and validate DNS responses using DNSSEC for security. Here's a quick summary:
Key Features:

   Recursive DNS Resolution: Unbound resolves domain names by querying authoritative DNS servers starting from the root.
   Caching: It stores DNS responses in memory, speeding up subsequent queries.
   DNSSEC Validation: Ensures DNS responses are authentic and not tampered with.
   Privacy and Security: Runs locally to avoid third-party DNS providers, and supports DNS-over-HTTPS (DoH) and DNS-over-TLS (DoT) for encrypted queries.
   Customizable: You can configure cache size, query behavior, and security features based on your needs.

Benefits:

   Performance: Caching reduces query times for frequently accessed domains.
   Privacy: Running your own resolver improves privacy by not relying on external DNS providers.
   Security: DNSSEC validation helps protect against DNS spoofing and other attacks.

Unbound is ideal for improving DNS speed, privacy, and security, especially when used alongside tools like Pi-hole to block unwanted content.


How to install Unbound:


On your Debian 12 system, install Unbound with the following command:

```
sudo apt update
sudo apt install unbound
```

To perform recursive DNS queries for hosts that are not cached, Unbound must query the root DNS servers. By default, it includes a built-in list of root servers. It’s important to update this file approximately every 6 months.

To update the root hints file, run:

```
wget -O root.hints https://www.internic.net/domain/named.root
sudo mv root.hints /var/lib/unbound/
```
How to configure Unbound:


Create and edit the Unbound configuration file:

```
sudo nano /etc/unbound/unbound.conf.d/pi-hole.conf
```
Add the following content to the configuration file:

```
server:
    verbosity: 0
    interface: 127.0.0.1
    port: 5335
    do-ip4: yes
    do-udp: yes
    do-tcp: yes
    do-ip6: no
    prefer-ip6: no
    root-hints: "/var/lib/unbound/root.hints"
    harden-glue: yes
    harden-large-queries: yes
    harden-dnssec-stripped: yes
    edns-buffer-size: 1232
    rrset-roundrobin: yes
    cache-min-ttl: 300
    cache-max-ttl: 86400
    serve-expired: yes
    harden-algo-downgrade: yes
    harden-short-bufsize: yes
    hide-identity: yes
    identity: "Server"
    hide-version: yes
    do-daemonize: no
    neg-cache-size: 4m
    qname-minimisation: yes
    deny-any: yes
    minimal-responses: yes
    prefetch: yes
    prefetch-key: yes
    num-threads: 1
    msg-cache-size: 50m
    rrset-cache-size: 100m
    so-reuseport: yes
    so-rcvbuf: 4m
    so-sndbuf: 4m
    unwanted-reply-threshold: 100000
    log-queries: no
    log-replies: no
    log-servfail: no
    log-local-actions: no
    logfile: /dev/null
    private-address: 192.168.0.0/16
    private-address: 169.254.0.0/16
    private-address: 172.16.0.0/12
    private-address: 10.0.0.0/8
    private-address: fd00::/8
    private-address: fe80::/10
```

To write the changes to the file using nano, press Ctrl + O, then press Enter, and finally press Ctrl + X.

How to check Unbound Configuration for Errors:


This step is optional but recommended. You can check the configuration for errors using the following command:

```
unbound-checkconf /etc/unbound/unbound.conf.d/pi-hole.conf
```
It should return "no errors" if the configuration file is correct.

How to start the Unbound Service:


Run the following command:

```
sudo service unbound start
```
You can check if your DNS server is resolving domains correctly by running:

```
dig pi-hole.net @127.0.0.1 -p 5335
```
The first few queries may be slower, but subsequent queries should resolve in under 1 millisecond.

How to test DNSSEC Validation:


Run the following commands:

```
dig sigfail.verteiltesysteme.net @127.0.0.1 -p 5335
dig sigok.verteiltesysteme.net @127.0.0.1 -p 5335
```
The first query should return a SERVFAIL status with no IP address, while the second should return a NOERROR status along with an IP address.

How to configure Pi-hole to Use Unbound:


   Log in to the Pi-hole admin interface.
   Go to Settings → DNS.
   Select Custom 1 (IPv4) under Upstream DNS Servers and enter 127.0.0.1#5335.
   Uncheck all other DNS options in the Upstream DNS Servers section.
   Under the Advanced DNS Settings section, ensure that Never forward reverse lookups for private IP ranges and Never forward non-FQDNs are checked. Also, disable DNSSEC in Pi-hole as Unbound will handle it for you.
   In Interface Listening Behavior, select Listen on all interfaces unless you're setting up a VPN server. For a typical home network, the default should work fine.
   Click Save.


How to troubleshoot:


If Unbound fails to start or throws an error, use the following command to check its status:

```
sudo service unbound status
```

If you encounter an "anchor not OK" error, restart the Unbound service:

```
sudo service unbound restart
```
If Unbound is still failing or trying to listen on port 53 (which Pi-hole uses), check the configuration file for any issues:

```
cat /etc/unbound/unbound.conf.d/pi-hole.conf
```
To edit the configuration, use:

```
sudo nano /etc/unbound/unbound.conf.d/pi-hole.conf
```
Save the changes (Ctrl + O → Enter → Ctrl + X) and then reboot your system:

```
sudo shutdown now -r
```


