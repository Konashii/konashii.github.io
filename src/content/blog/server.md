---
layout: ../../layouts/BlogLayout.astro
title: 'Setting Up My Homelab'
pubDate: 2026-08-13
description: 'A post about setting up my own personal server'
---

# Introduction

Hello, this post will be updated overtime as I continue to work on my home server. This is so that there won't be a bunch of reposts

## Turning an old laptop into a server

I've always been into the idea of turning older hardware that could be in your attic or basement, and giving it a new life by turning it into some homelab server where you will have more opportunities to experiment and learn lots of new things.


## What I will be transforming

A friend of mine was trying to buy some used chromebook for a relatively cheap price and it gave me the inspiration to do the same. I went on eBay and saw a listing of a **Lenovo B590** only for $20, and the only issue listed was that it took too long to bootup to Windows 7.

This finding is DEFINITELY a steal, especially if that's the only issue wrong with it. I decided to take the bait and purchased the laptop. It arrived just under 48 hours and there it was, with even the charging cable, which I was worried it wouldn't come with.

When I turned on the laptop it still booted up just fine into Windows, sure it takes a long time but when you have a physical hard drive it's something you get used to. I definitely need to swap it with an SSD when I can.

## Preparation

As soon as I had the laptop in my hands, I decided to get my USB flashdrive ready to boot Ubuntu Server. To be honest, as much as I would love to use Arch Linux or Gentoo, I know I'm going to spend a ridiculous amount hours figuring out the most basic problems instead of implementing all the services I've been planning.

With Ubuntu Server installed, before I do anything I wanted to make sure I have OpenSSH and adjust the power settings so that I can already have the lid closed and set on my desk to be on 24/7. I connect it to my network switch, I configure the `/etc/systemd/logind.conf` file so it won't power off when I shut the laptop. Now, I can set it aside and not bother with it, as I can simply SSH into it within my desktop system.

## Setting up Docker

I will be running a few of my services using Docker. I have NEVER used Docker before so this is going to be a new experience. It took me a while to kind of even figure out how it should be setup, before I decided to do a whole docker-compose.yml file for simplicity's sake.

As of this post's latest update, I will mainly be running **Pi-Hole**, and Squid **Web Proxy**.

### Pi-Hole

I build Pi-Hole via docker using the image `pihole/pihole:latest`

It's actually pretty straight forward. No need to mess with configuration files, simply just allowing the correct ports through the firewall and such.

### Squid Web Proxy

The image is built with `ubuntu/squid`

For this one, the configuring `squid.conf` is more like setting up a firewall, as for me it's mostly just allowing certain local IP addresses and port 3128. On top of that since this proxy will actually be using the Pi-Hole DNS service, I had to route all traffic through that DNS server as well. 

It's very important to have a default-deny rule by default, for security reasons. Then after that you can implement specific IP's that are allowed through this proxy, which would be the Pi-Hole service, and any other device strictly within my private network range.

