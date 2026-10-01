# Presentation-Transcript-updated
# Presentation Transcript: How Data Travels Across a Network

**A TCP/IP and OSI Walkthrough of Opening a Secure Website (HTTPS)** Estimated time: 18–20 minutes (about 1 minute per slide)

| Speaker | Slides |
| --- | --- |
| Udochukwu Godwin Chukwu (Team Lead) | 1–6 |
| Amarachi Madubuike | 7–12 |
| Sharon Omowumi | 13–19 |

---

## Part 1: Udochukwu

### Slide 1: Title

Good day, everyone. We are Group \[number\], and today's presentation is called *How Data Travels Across a Network*. I'm Udochukwu, the team lead, and with me are Amarachi and Sharon. We will follow one simple action, opening a secure website over HTTPS, and show exactly what happens inside the network from the moment you type the address to the moment the page appears.

### Slide 2: Scenario Overview and Objectives

Here is our scenario. A user opens a laptop and browses to a secure website. On the right you can see the journey: the laptop and browser, a Wi-Fi access point, a switch, a router with firewall and NAT, the Internet through ISP routers, and finally the web server at 203.0.113.10.

Our objectives are to follow one HTTPS request from start to finish using the TCP/IP model, map each TCP/IP layer to its OSI equivalent, show encapsulation on the way out and decapsulation on the way back, identify the protocols, addresses and ports involved, and finally diagnose a realistic connectivity failure using the same model. We also captured real traffic in a lab, so you will see actual Wireshark evidence along the way.

### Slide 3: The TCP/IP Model: Four Layers

The TCP/IP model has four layers. The **Application** layer builds and interprets the request and response. In our scenario the browser creates the HTTPS request and TLS encrypts it. The **Transport** layer handles end-to-end delivery between processes using ports, and here TCP opens a reliable connection to port 443. The **Internet** layer does logical addressing and routing, so IP addresses the packet from the laptop to the server. Finally, **Network Access** handles physical addressing and framing, so Ethernet or Wi-Fi frames carry the data across each hop.

### Slide 4: OSI Model and Its Mapping to TCP/IP

The OSI model has seven layers, and the table shows how they map onto TCP/IP. Layers 7, 6 and 5, Application, Presentation and Session, all fold into the single TCP/IP Application layer. In our scenario the browser builds the GET request at Layer 7, TLS encrypts the data at Layer 6 and manages the secure session at Layer 5. TCP at Layer 4 maps to Transport, IP at Layer 3 maps to Internet, and Layers 2 and 1, Data Link and Physical, map to Network Access. Next, we follow the packet journey, starting with DNS.

### Slide 5: Step 1, Resolving the Domain Name (DNS)

Before anything else can happen, the browser needs an IP address, because computers route by IP and not by name. So it sends a DNS query over UDP to destination port 53. The query goes to the configured DNS server, which may forward it to an upstream resolver, and the reply returns the address. We use UDP because the exchange is small and, if no answer arrives, the client can simply retry. On the right you can see the encapsulation: the DNS message goes into a UDP datagram, then an IP packet, then an Ethernet or Wi-Fi frame.

### Slide 6: Lab Capture, DNS Lookup in Practice

Here is the real version from our lab. On the left, we ran `nslookup wikipedia.org`. The client asked the DNS server at 10.0.2.3 on port 53 and received 195.200.68.224 as the IPv4 address, plus an IPv6 address as well. On the right is the Wireshark capture with the `dns` filter applied. You can see two queries, one for an A record and one for an AAAA record, and two responses, between our machine at 10.0.2.15 and the DNS server. The first query was only 73 bytes, which shows why UDP is a good fit here. I'll now hand over to Amarachi, who will take us through the connection and the packet itself.

---

## Part 2: Amarachi

### Slide 7: Step 2, Establishing the Connection (TCP + TLS)

Thank you, Udochukwu. With the IP address known, the browser must now connect. First comes the **TCP three-way handshake**: SYN, then SYN-ACK, then ACK. This creates a reliable, ordered channel between the client port and server port 443 before any data is sent. On top of TCP, the **TLS handshake** runs. The client sends a ClientHello, the server answers with a ServerHello and its certificate, and then the key exchange takes place. This validates the server's identity and creates a shared session key, so everything afterwards is confidential and authenticated.

### Slide 8: Lab Capture, TCP Handshake and TLS in Wireshark

This capture proves it. We filtered on `tcp.port == 443`. At the top you can see the SYN from 10.0.2.15, port 50992, to 151.101.129.91 on port 443, then the SYN-ACK, then the ACK. Straight after that is the TLS 1.3 Client Hello, followed by Change Cipher Spec, and from then on the payload is simply labelled Application Data, because it is encrypted. One note: this particular capture is a Firefox connection to ads.mozilla.org, but the handshake sequence is identical for any HTTPS site, including Wikipedia.

### Slide 9: Step 3, Encapsulating the HTTPS Request

Now let's see how the request is packaged. This is called encapsulation, and each layer adds its own header as the data goes down the stack. At the Application layer we have the encrypted HTTP GET request, called Data. The Transport layer adds a TCP header with the source and destination ports, and now it is a **Segment**. The Internet layer adds an IP header with the source and destination IP addresses, producing a **Packet**. Finally, Network Access adds the Ethernet or Wi-Fi header with MAC addresses, giving us a **Frame**.

### Slide 10: Lab Capture, Encapsulation in a Real Packet

Here is a real packet showing exactly that. On the left, the Ethernet II header carries the source and destination MAC addresses, and inside it the IPv4 header shows 10.0.2.15 going to 10.0.2.3, protocol 17, which is UDP, with a time to live of 64. On the right, we see the UDP header with ports 40486 and 53, and inside it the DNS query. Wireshark even lists the nesting for us as eth, ethertype, ip, udp, dns. Each layer wraps the one above it.

### Slide 11: Step 4, Across the Network, and Back

On its way to the server the frame passes through several devices. The switch forwards it using MAC addresses only, which is Layer 2. The router decapsulates to the packet, applies NAT and firewall rules, then re-encapsulates it into a new frame. Every Internet router repeats this: strip the frame, read the IP, forward, build a new frame. The key concept is that the **IP address stays constant** end to end, apart from NAT, but the **MAC address changes at every hop**. The server then sends the response back through the same process in reverse.

### Slide 12: Lab Capture, MAC Changes, IP Stays the Same

We can see this in the capture. This is a reply packet. The source IP is 151.101.129.91, the remote web server, and the destination is our laptop, 10.0.2.15. But look at the Ethernet header: the source MAC is 52:55:0a:00:02:02. That is the local gateway, not the server. The server's own MAC address never reaches us. Wireshark even flags it as a locally administered address. The earlier DNS query used a different destination MAC, which shows that the Layer 2 address depends on the next hop, while the Layer 3 address identifies the real endpoints. I'll hand over to Sharon, who will bring it all together.

---

## Part 3: Sharon

### Slide 13: Packet Journey Diagram

Thank you, Amarachi. This diagram brings everything together. The request, an HTTPS GET, travels from the laptop through the access point, switch, router and firewall, and across the Internet to the web server. The response, HTTP 200 plus the page, comes back the other way, and the session is TLS-encrypted end to end. Notice that full encapsulation and decapsulation across all four layers happens only at the laptop and the web server. Intermediate devices process fewer layers.

### Slide 14: Lab Capture, The Page Loads

And here is the final result. The browser shows [https://www.wikipedia.org](https://www.wikipedia.org) with the padlock icon, which tells us the TLS session is in place. DNS, TCP and TLS all completed before the first byte of the page arrived, and each step took only a fraction of a second. The journey is complete: Application, Transport, Internet and Network Access on the way out, and the reverse on the way back.

### Slide 15: Protocol Analysis Table

This table summarizes every protocol we saw, layer by layer. At the Application layer: HTTP, HTTPS with TLS, and DNS, using domain names, to build and encrypt the request and resolve the hostname. At Transport: TCP, with UDP for DNS, using ports, for reliable ordered delivery. At Internet: IPv4, using IP addresses, to route the packet from laptop to server. At Network Access: Ethernet, Wi-Fi and ARP, using MAC addresses, to deliver the frame across each physical hop.

### Slide 16: TCP vs UDP

Two Transport protocols appeared in our scenario. **TCP** is connection-oriented, with a three-way handshake. It is reliable, retransmits lost segments, guarantees order, and includes error checking and congestion control. The cost is higher overhead. It is used for HTTPS, email and similar services. **UDP** is connectionless: no handshake, no retransmission, no ordering guarantee. It is lighter and faster, which makes it right for DNS queries and for real-time traffic like VoIP and video conferencing.

### Slide 17: When It Breaks, A Realistic Failure

Now let's apply the model to a real problem. A user says: "I can't open [https://www.example.com](https://www.example.com). It just times out, but other websites work fine." Looking at the symptoms, only this one site fails, and there is no certificate warning, which means the connection never completes. That points away from DNS or a browser or TLS problem, and toward something blocking traffic to this particular server. After diagnosis, the root cause was a recent firewall rule change that added an overly broad deny rule, blocking outbound TCP port 443 to this server's IP range.

### Slide 18: Troubleshooting Flow

Here is how we found it, step by step. **One**, define the problem: a timeout on one site, not a certificate warning. **Two**, check connectivity: pinging the default gateway succeeds, so the local network is fine. **Three**, verify DNS: `nslookup` resolves the name correctly. **Four**, check the port: `Test-NetConnection` to TCP 443 times out, so something is blocking it. **Five**, inspect devices: the firewall logs show the new deny rule. **Six**, fix and confirm: we add an allow rule for TCP 443, re-test, and the page loads. Notice that we worked up the layers, which is exactly why understanding the model matters.

### Slide 19: References

Our sources were the IETF's RFC 9110 on HTTP Semantics and RFC 9293 on TCP, Cisco Networking Academy course materials, the CompTIA Network+ study guide, and Microsoft Learn's TCP/IP protocol architecture.

To conclude: every web page you open travels through four layers, is wrapped and unwrapped at every hop, and relies on DNS, TCP, TLS and IP working together. Thank you for listening. We are happy to take any questions.

---

### Speaker tips

- Pause briefly when the slide changes, and let the audience read the title before you speak.
- On the lab slides, point at the screenshot (for example the SYN line or the MAC address) as you describe it.
- If you are short on time, slides 9 and 15 can be shortened to two sentences each.
