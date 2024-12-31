---
description: >-
  This section combines the techniques from the ARP & DNS spoofing, and
  demonstrates how an attacker would be able to leverage the MitM position to
  intercept and modify HTTP data/traffic.
---

# HTTP MitM attack

In an effort to understand the dangers of a public network, and the risks of ARP spoofing attacks, I have came up with a simple simulation example.

{% embed url="https://jarrett.gohxiangzheng.com/projects/http-mitm-attack/" %}
Demonstration
{% endembed %}

### Overview of attack

<mark style="color:red;">**Attacker**</mark>&#x20;

1. Run a HTTP proxy server to intercept and sniff or modify data
2. Run a rogue DNS server on the attacker machine, to resolve queries for a few selected or all the host names, to the IP address of the attacker. Special hosts file rules can be added for fake subdomains that does not exist
3. Run ARP spoof attack on the victim machine, sending ARP replies to the victim and router gateway:
   * **ARP reply to the gateway**: \<victim\_ip><mark style="color:red;">is-at</mark>\<attacker\_mac\_addr>
   * **ARP reply to the victim machine**: \<gateway\_ip><mark style="color:red;">is-at</mark>\<attacker\_mac\_addr>
4. Sends a phishing email to the victim, claiming to be from a trusted website. The email contains a seemingly legitimate HTTP link of a fake subdomain of the trusted root domain

<mark style="color:green;">**Victim**</mark>

1. Receive an email claiming to be from a trusted website
2. Clicks on the link provided in the email
3. Provides login credentials to "authenticate" with the website
4. Performs a certain action

<mark style="color:red;">**Attacker**</mark>

At this point, the user have gained control of the user's connection to the seemingly trusted website, and have stolen the login credentials

The attacker can exploit the user's trust in the website, and trick the user in performing actions

### Parameters

<mark style="color:green;">**Victim**</mark>

* Windows
* 10.0.2.4

<mark style="color:red;">**Attacker**</mark>

* Kali Linux
* 10.0.2.5

**Gateway**

* 10.0.2.1

### Attacking commands

**Terminal 1**&#x20;

* Enable IPv4 forwarding

```bash
$ sudo sysctl -w net.ipv4_ip_forward=1
```

* Create a firewall rule to redirect traffic to TCP port 80 (HTTP) to port 8080

```bash
$ sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 8080
```

* Compile and run the HTTP proxy

```bash
$ gcc mitm_proxy_http-hijacK.c -o http-proxy && sudo ./http-proxy
```

{% embed url="https://github.com/Jarrettgohh/c-sockets/blob/main/mitm_proxy_http_hijack.c" %}

**Terminal 2**

* Drop all DNS query destined for the actual gateway

```bash
$ sudo iptables -A FORWARD -d 10.0.2.1 -p udp --dport 53 -j DROP
```

* Load DNS hosts file

```bash
$ cat hosts.spoof
10.0.2.5 *
```

* Run DNS server

```bash
$ sudo dnsspoof -i eth0 -f hosts.spoof
```

**Terminal 3**

* Run the ARP spoof attack

```bash
$ sudo arpspoof -i eth0 -t 10.0.2.4 10.0.2.1 -r
```

**Craft and send a malicious phishing email**

...



## msftconnecttest.com

The address <mark style="color:blue;">http://www.msftconnecttest.com/redirect</mark> is automatically opened on the default browser based on a few factors:

1. ...

Notice that the website used in the Windows NCSI configurations are in HTTP, rather than HTTPS. If these factors could be replicated by an attacker on the same network as a user, it could trick the browser to open this address, which can be intercepted and modified.

### Factors that trigger the browser to open the msfconnecttest.com/redirect address (theory)

1.  **Proxy server connection is detected**

    * Based on the TUN/TAP interface?
    * How to trick the user device to believe it is connected to a proxy server?


2. **NCSI probs failure**
   * how to make it fail?

Refer to " <mark style="color:blue;">Why default browser automatically opens</mark>" link below:

<mark style="background-color:blue;">"In some cases, such as when you connect to a network that uses a proxy server to connect to the internet or when network restrictions prevent NCSI from completing its active probe process, Windows opens the MSN Portal page in the default browser."</mark>

### Factors that trigger the browser to open the msfconnecttest.com/redirect address (practical experimentation)

#### Assume the following:

1. Both the user and attacker machine are in a virtualbox NAT network
2. Default gateway would be the host machine at a particular address
3. DNS server address is inherited from the host machine &#x20;

<mark style="color:red;">**Attacker machine**</mark>**: Run usual ARP & DNS spoof techniques**

* &#x20;ip forwarding, drop DNS packets, etc.
* Resolve **www.msftconnecttest.com** to any address

<mark style="color:green;">**Windows user machine**</mark>: **Simulate change in network settings**

* Trigger disconnect of network interface on Windows machine

```powershell
$ ipconfig /flushdns
```

```powershell
PS C:\Windows\System32> netsh interface set interface <iface_name> disable
PS C:\Windows\System32> netsh interface set interface <iface_name> enable
```

&#x20;To flush the DNS cache, followed by disabling interface (disconnect from network), and wait a few seconds before connecting back again

#### Connect to VPN (expected to fail) for a few seconds, before disconnecting again

This will trigger the NCSI probes process:

1. DNS query for **www.msftconnecttest.com**
2. If step 1 succeeds, HTTP request to **http://www.msftconnecttest.com/connecttest.txt**
3. If step 2 fails to connect or doesn't return a valid content of "<mark style="color:green;">**Microsoft Connect Test**</mark>", the default browser with address **http://www.msftconnecttest.com/redirect** would be automatically opened



<mark style="color:red;">**Attacker machine**</mark>**: Run a HTTP proxy to intercept traffic**

Upon receiving HTTP request for **http://www.msftconnecttest.com/connecttest.txt,** either ignore the request, or reply with an invalid content (anything besides "<mark style="color:green;">**Microsoft Connect Test**</mark>") as mentioned above.

Subsequently, the default browser would be opened with HTTP GET request to **http://www.msftconnecttest.com/redirect** , where the content would be controlled by the attacker machine's HTTP proxy

NOTE:&#x20;

* **ARP spoof**: To redirect DNS & HTTP traffic during the NCSI probe process to the attacker machine
* **DNS spoof:** To resolve **www.msftconnecttest.com**&#x20;
* **HTTP proxy:** To send content when request is sent to **http://www.msftconnecttest.com/redirect**

#### <mark style="color:yellow;">Why doesn't the same behavior happen right after the device is reconnected to the network, without having to have any interactions with the VPN?</mark>

The NCSI probes are sent immediately after the device reconnects, this means that the ARP poisoned entry is not in use yet, and the initial DNS query for **www.msftconnecttest.com** would be resolved by the actual gateway instead, followed by the HTTP request, which would resolve correctly and allow the NCSI probe process to succeed. Hence, the **http://www.msftconnecttest.com/redirect** address would not triggered on the default browse&#x72;**.**

**Solution: Drop DNS packets returning from the gateway that is intended for the device**

```bash
$ sudo iptables -A FORWARD -p udp --sport 53 -j DROP
```

This would prevent the initial real DNS response from returning to the device. Subsequently, the DNS queries would be sent to the attacker device instead, due to the ARP spoof.

However, this seems to not be enough for the default browser to be automatically opened.

### Authentication and automatic sign-in page

How to make the device believe that the network requires authentication, to trigger the sign-in page to be opened on the browser?





{% embed url="https://learn.microsoft.com/en-us/windows-server/networking/ncsi/ncsi-frequently-asked-questions" %}

{% embed url="https://learn.microsoft.com/en-us/troubleshoot/windows-client/networking/internet-explorer-edge-open-connect-corporate-public-network#ncsi-active-probes-and-the-network-status-alert" %}
&#x20;Why default browser automatically opens
{% endembed %}

{% embed url="https://www.comparitech.com/net-admin/fix-msftconnecttest-redirect-error/" %}
