---
title: "Wireguard Basic Setup"
related: true
categories:
  - linux
  - hacks
tags:
  - VPN
  - wireguard
  - kernel module
  - linux
toc: true  
---

## Introduction

Wireguard is an alternative for VPN based tunnel such as OpenVPN, LT2P/IPSec.
The goal of wireguard was to make all the network tunnelling along with encryption of network packets to be done at the the kernel space level whereas other VPN tunnel based protocols occur at the user space level.
This should remove the overhead of encrypting the packets at the user space level and transferring to kernel space level. The important points to remember are,

* Wireguard is developed as a kernel module for linux system (will be soon merged into the mainline kernel).
* Support Only in UDP based tunnelling.
* Doesn't support TCP based tunnelling. ([Tunsafe Software](https://github.com/TunSafe/TunSafe) supports TCP - 3rd party).
* Supports both ipv4 and ipv6 protocols.
* Supports default gateway route as well as split tunnelling.

## Wireguard Installation

### Archlinux

```shell
# Install the dkms version of the wireguard
# so that it dynamically recompiles
# every time new kernel is installed.
$ pacman -S wireguard-dkms wireguard-tools
```

### Ubuntu

```shell
$ add-apt-repository ppa:wireguard/wireguard
$ apt-get update
$ apt-get install wireguard
```

### Buildroot (Source Compilation)

```shell
# Select "wireguard" package
# Select "bash" shell because
# wireguard (wg-quick) requires bash
```

## Wireguard configuration
* Following instructions are adapted from [Archlinux wiki](https://wiki.archlinux.org/index.php/WireGuard) and tested by me.
* Generate the required keys first

#### Server
```shell
# Generate public key first
$ wg genkey > pubkey
# Generate private key from public key
$ wg genkey | tee privatekey | wg pubkey > publickey
# Optional - generate pre-shared genkey
$ wg genpsk > preshared
```

#### Client
```shell
# Generate public key first
$ wg genkey > pubkey
# Generate private key from public key
$ wg genkey | tee privatekey | wg pubkey > publickey
# Optional - generate pre-shared genkey
$ wg genpsk > preshared
```
* Create the configuration file as follows

#### Server
```shell
/etc/wireguard/wg0.conf
------------------------------------
[Interface]
Address = 10.20.20.1/24
SaveConfig = true
ListenPort = 51820
PrivateKey = [SERVER PRIVATE KEY]
# MTU can be adjusted to improve VPN performance
MTU = 1420
# note - substitute eth0 in the following lines to match the Internet-facing interface
PostUp = echo 1 > /proc/sys/net/ipv4/ip_forward
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

[Peer]
# client foo
PublicKey = [FOOs PUBLIC KEY]
PresharedKey = [PRE-SHARED KEY]
AllowedIPs = 10.20.20.2/32

[Peer]
# client bar
PublicKey = [BARs PUBLIC KEY]
AllowedIPs = 10.20.20.3/32
```
#### Client

```shell
foo.conf

[Interface]
Address = 10.20.20.2/24
PrivateKey = [FOOs PRIVATE KEY]
DNS = 10.20.20.1

[Peer]
PublicKey = [SERVER PUBLICKEY]
PresharedKey = [PRE-SHARED KEY]
AllowedIPs = 0.0.0.0/0, ::/0
# Endpoint should be the public ip address or DNS
Endpoint = my.ddns.address.com:51820
```
* The above setup has been setup in my Orangepi Board with buildroot based linux system and tested.

* The performance is quite good. I was able achieve upto 140Mbps throughput on a 300Mbps connection.
