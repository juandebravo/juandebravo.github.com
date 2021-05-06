---
layout: post
title: "SSH tunnel to connect to a remote host"
categories: [connectivity, ssh]
---

I discovered [this explanation about SSH tunnels](https://unix.stackexchange.com/a/115906) some years ago 
and I found it fascinating. The sketches are super useful to understand the different options provided 
by an SSH tunnel to connect either to a "local port in the remote host" or just use the tunnel as a 
hop to connect to another "faraway" host.

Prior to 2020 when having business trips was a commont thing, I visited China to meet some gaming studios, and
also took the opportunity to have some fun riding from Beijing to the Great Wall :sun-glasses:

One of the issues when visiting China is the big amount of restrictions you have for accessing common websites
/ products here in Spain. In my case, I'm ok not using social networks but I hadn't access to Gmail, which was
an issue.

When something like that happens, the most common solution is using a VPN. But another solution if you don't want
to install a VPN client, trust the VPN provider and so on is using an SSH tunnel.

This approach has been super useful today again, not in China but in Barcelona, because there's an ongoing and
annoying network/connectivity issue that is preventing us to access github.com. 

This is what I came up with:


As you can see, all you need is:

- a free tier AWS instance running outside the country/network that you need to bypass
- basic SSH configuration for using the right certificate
- configure the local hosts file to avoid invalid certificate issues (you're accessing github.com instead of localhost)

