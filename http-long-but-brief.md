# HTTP, Web and Application Layer Protocols: Long, but Brief --- A Markdown Codex by Honorable Charlse Foster Kane, Phd, Md, DDS

Use this menu to navigate to each section. It is not guaranteed that the sections will be self-contained..

* [Preface](#preface)
* [Errata](#errata)
* [Internet and The Internet](#internet-and-the-internet)
* [Protocol Stacks](#protocol-stacks)
* [Application-Layer Protocols in TCP/IP](#application-layer-protocols-in-tcpip)
* [Gartner Model of the Web and its Evoltion](#gartner-model-of-the-web-and-its-evoltion)
* [The Structure of HTTP](#the-structure-of-http)
* [WWW and DNS](#www-and-dns)
* [Web Frameworks and Web Services](#web-frameworks-and-web-services)
* [A Very Simple and Inadequate HTTP Framework Implementation](#a-very-simple-and-inadequate-http-framework-implementation)
* [CGI, RPC via HTTP, SSL/TSL and HTTP Proxies](#cgi-rpc-via-http-ssltsl-and-http-proxies)
* [Finishing Thoughts](#finishing-thoughts)

## Preface

This tutorial spun out of control. But I am content, as it may help someone, and even if it helps one person, I am happy. The audience of this tutorial can range from the very beginner to someone who is interested in web dev but needs a primer on most of its aspects. It is not thorough, web is like a large fishing net and to cover all aspects of it, you must read a text book. There's oh so many books I can recommend but I won't be a dick and recommend the best one, that is, No Starch Press' *Network Programming in Go*.

I just wrote several application-layer protocols in Golang for fun, it is called ProtoGen and you can find it [here](https://github.com/Chubek/Protogen/). I thought why should I not write a tutorial about relatively the biggest one of them out there, that is, HTTP, the protocol of the web, of which people often interchangeably refer to when they use the term 'the internet' and 'the web'. I will also explain a little about designing application-layer protocols. 

## Errata
This section is reserved for when people who know better fix my errors, and I add them here. Please report errors to my contacts in the profile main page.

## Internet and The Internet
The Internet is de facto the largest of internets out there. There can be dozens and dozens of internets, often referred to as interAnet, but the largest internet is called The Internet. The Internet is built upon what is referred to as 'The Backbone'. Remember that both The Internet and both The Backbone have a definite article and start with a capital letter. Because there can be several dozen internets each having their own backbone. But The Internet is the one that has The Backbone, and this backbone is comprised of several hundreds of thousands of cables carrying data across the ocean from one host center to one host center, each host having their own network which broadcasts with an Exterior Boarder Gateway Protocol to another, and within these hosts there are Interior Gateway Protocols that handle the network from within. In other word, any internet is a series of interconnected networks, a network of local-area networks connected to a wide-area network connected to an ISP etc etc. The Internet happens to be global.

## Protocol Stacks
To manage the data within this interconnected network the packets must be carefully structured. You can, theoretically, set up a LAN network in your house and send a packet to your brother that says 'come to dinner' but that would not work unless your PC is directly connected to your brother's PC. If you are connected through a router, the router must know what is going on in the packet coming from your computer. As such structures have been defined for these networks, called stacks, the first of which was OSI which was never implemented. OSI has 7 layers and is extremely verbose and impractical in the real world. Another such stack structure is the IP stack. IP stack is comprised of 4 layers. The first layer is the hardware, or *link layer*. The second layer is the *internet protocol layer* or the IP layer. These layers are mandatory. Without these IP stack would not be. But after the IP layer comes two other layers, the *transport layer* and the *application layer*. With these you are free to maneuver.

Each layer builds upon the other layer. Link layer contain the IP layer, the IP layer contains the link layer and the IP layer and the transport layer, and the transport layer contains all the previous and the application layer and the application layer contains all 4. 

```
|-------|----------------|----------|
|  Link |   IP  |  Trans |    App   |   <--- Application-layer #4
|       |       |        |          |
------------------------------------
|-------|----------------|
|  Link |   IP  |  Trans |             <--- Transport layer #3
|       |       |        |
--------------------------
|-------|--------
|  Link |   IP  |                      <--- IP layer #2
|       |       |
-----------------
| Link  |
|       |                             <--- Link layer #1
|-------
```

These are all made out of bytes. The protocol rules give shape to them. For example the IP layer, version 4 or IPv4 that is, has the following structure:

```

0b----------4b--------8b---------16b------------------32b
|   Ver     |  IHL    |   TOS    |    Total Length     |
|-------------------------------------------------------
|           Identification       |  flags | Offset     |
--------------------------------------------------------
|    TTL    | Protocol |       Header Checksum         |
--------------------------------------------------------
|                     Source Addr                      |
--------------------------------------------------------
|                     Dest Addr                        |
--------------------------------------------------------
|                      Options                         |
--------------------------------------------------------
|     The transport layer header and app layer data    |
----------------------------------------------------- --
 ```

There is a thing known as the *Internet Protocol Suite* which has some 175 layer-3 and layer-4 protocols. Some of these protocols such as ICMP, the protocol you use when you type `ping google.com` in terminal in Linux or command-line on Windows, don't rely on a fourth level and are entirely one header contained within level 3. But level 3 as a whole is mostly comprised of a header.

There exists 2 major transport-layer protocols with the Internet Protocol Suite that are used with raw packets. One is Transfer Control Protocol or TCP and the other is User Datagram Protocol or UDP. HTTP 1.1 and HTTP 2 run on TCP but HTTP 3.0 which is the latest version and only ran on 5% of the servers runs on QUIC which is itself based on UDP. The problem with UDP is, it does not rely on a connection so to speak. You just connect to the endpoint and send, and receive back. This makes it prime for Peer-to-Peer protocols. But TCP is based on connection, and a 3-way handshake that is comprised of three signals: **SYN**, **SYN-ACK**, **ACK**. After the connection has been established, the data is transferred within one burst, and the FIN signal is sent and the connection is closed. This makes TCP slower, but more reliable because TCP knows until FIN is sent it should expect data so lost packets are retrievable. However with UDP that is impossible. QUIC protocol seeks to fix that. 

## Application-Layer Protocols in TCP/IP
Let's assume we have chosen our transport protocol to be TCP because we need a burst-style connection, and not any sort of streams. We would also like the protocol to handle the arrival of packets and not us. 

Every OS allows for TCP/IP socket programming and most programming languages that have an INET/INET6 networking interface support sending packets through TCP. To send and receive TCP packets, we can use **netcat**. Netcat is a POSIX utility found in Linux, Mac, BSD and so on. On Windows you can find a precompiled version from [here](https://eternallybored.org/misc/netcat/) or build it yourself from source using a C compiler for Windows. But that website should be enough. *NOTE* the `echo` style command that I'm using here only works for POSIX systems. On windows just cd to the folder where `nc64` is and run it through the commandline or shell, then communicate with the REPL.

Anyways in my project, ProtoGen, which I linked above, I have three application-layer protocols. One, ProtoDir, uses UDM which uses a socket and is only local. But the other two, ProtoQuote and ProtoMath, use TCP and they are currently both up on my server at horatio.chubakbidpaa.com ports 8888 and 9999 respectively. ProtoQuote only serves a response upon receiving a connection, sending back a random quote that is refreshed every 30 minutes or so. ProtoMath accepts a request, in form a mathematical equation --- only the *, +, - and / and parses the equation based on PEMDAS, and parenthesis too. I have wrote the Shunting Yard algorithm that parses this myself, for an old project. Anyways let's send a request to my server at port 8888 (careful not to bring my server down pls) and see what the quote at that interval is:

```
echo '' | nc horatio.chubakbidpaa.com 8888
```

Currently this is the response that I am getting:

```
PTQP v1 SimplePRTQTv0.1

Now = 2023-02-22 17:06:49.86667104 +0000 UTC m=+2278.336472220
Author = Miguel de Cervantes
Quote = "Be slow of tongue and quick of eye."
```

The first line is the response protocol and version control. Every application-layer protocol that is worth a damn must have a name and version control. HTTP has a name and version control too which we'll get to later. The third word in the first line is name of the server. I just added it theoretically. In HTTP name of server is sent through headers, but it's optional, and it can be several servers as the response passes through possibly several frameworks. 

Interesting tidbit, ProtoQuote is a throwback to Quote Of the Day Protocol, a now defunct member of the Internet Protocol Suite. It used to be online on port 17 of most servers. If you try and look you may find old servers that have not closed this port but because this protocol did not rely on a transport protocol it was extremely dangerous to run it so most admins close this port.

Anyways after the header there is a CLRF. What is a CRLF? Basically, newlines come in two types. CRLF and LF. In CRLF the ASCII byte for control, that is, 13 or 0xD is inserted before ASCII byte for newline, that is 10 or 0xA. On Windows newline for text files is CRLF but in Linux it's LF. In HTTP, the three parts of the response, which we'll get to, are separated by two CRLFs and every field of the header is separated by an LF. In ProtoQuote the body is separated from each other using a single LF or 0xA. The first line of the response is `Now` which is used to denote the current time. The second is `Author` which is used to denote the author and the last one is quote itself. Notice that every field is separated from it's keyname by a space, that is, byte 32 or or 0x20, a `=` that is byte 61 or 0x3D and another space. Every byte in protocol is important! The person who is writing a client for your protocol must know what the hell is going on. For example in HTTP headers are separated from their key by a `:` that is a 58 or 0x3A and a space, and strings are enclosed withing double quotes. We'll talk about HTTP headers later. But also remember the bit about version control that I mentioned before. If I later decide that in PTQP, it should be a single `:` that separates the fields from their keys, then I should name that version something else. 

Another thing about application-layer protocols is data serialization. If an application-layer protocol carries data, it must mention the format, and the encoding of the format, in a universal text coded, so a chicken-or-egg scenario won't be created. Usually this universal codec is ASCII. That is why all HTTP headers are in ASCII. 

Anyways with ProtoQuote we did not send any requests, just a response. Just like responses, requests, too, need version control and protocol naming. In fact it is more vital in requests to have VC and NC, as you'll soon see in ProtoMath I have omitted VC and NC from the response to lessen the load on my server. 

A request to ProtoMath looks like this: `PTMPv1 <equation>`. As mentioned before ProtoMath accepts the four primary operations of +, -, / and * with PEMDAS in mind and also accepts reordering of operations through parenthesis. Let us send a request to ProtoMath running on my server. Again don't tank it please. 

```
echo 'PTMPv1 2 * 2' | nc horatio.chubakbidpaa.com 9999
```

The response is:

```
100 PARSE WAS SUCCESSFUL

4.0000

```

The response is comprised of the message and status code, one LF, the answer, and another LF. Status codes are very important in making of a protocol. Every outcome must be accounted for and the response must be 1- clear 2- parsable. Parsability is extremely important. Don't put the author of the client for protocol through hell! For example ProtoMath has 5 status codes:

```
51 EQUATION PARSE FAILED
52 REQUEST PARSE FAILED
53 UNALLOWED BYTE DETECTED
54 DETECTED UNSUPPORTED OPERATION

100 PARSE WAS SUCCESSFUL
```

For example:
```
echo 'PTMPv1 2 ** 2' | nc horatio.chubakbidpaa.com 9999
```

Will yield:

```
54 DETECTED UNSUPPORTED OPERATION

```

If you notice all the messages fit neatly in 3 words. That makes the parsing simpler. I have made provisions for when people use `**` for exponentiation which my Shunting Yard parser does not support. Or when any character except the operators and a number is sent, it will send status 53. Request parse will fail if the NC and NV are not sent. And finally equation will fail to parse, and 51 is sent.

Always group in your status messages. We are all familiar with how HTTP handles its status messages, with success being in 200, further actions in 300, access being in 400, server being in 500 and so on and so forth.

Now that we are more familiar with application-layer protocols, let's talk a bit about HTTP as promised.

## Gartner Model of the Web and its Evoltion

There exists a system for modelling protocols and the such called the Garner Model. Gartner model is composed of three components: Presentation, Application Logic and Data. There are five major Garner models, for example, below you see the *remote presentation model* which protocols such as Secure Shell (SSH) adhere to:

```
|-----------------|
|  Presentation   |
|-----------------|

|-----------------|
|      Logic      |
|-----------------|
|-----------------|
|      Data       |
|-----------------|

```

So when something is separate from the rest in Gartner model, it means that concept is decoupled, or remote, and the parts residing next to each other are in one place.

With web, notice that we're talking about web, with the web pages and the browsers and the whatnot, and not HTTP as a protocol, which could mean things such as RPC via HTTP and REST APIs, we come up with this model:

```
|-----------------|
|  Presentation   |
|-----------------|
|-----------------|
|      Logic      |
|-----------------|

|-----------------|
|      Logic      |
|-----------------|
|-----------------|
|      Data       |
|-----------------|

```

And this is an apt representation. Again, notice that we are only considering web, not HTTP as a whole. Par example, let's consider a website from the late 90s. In the 90s web was still in its infancy and we did not have the stupidly dynamic pages of today. In such website, we had presentation, through HTML and JS coupled with logic, through the browser's inner engine, remotely connecting to a serve running, say, Apache Web Server or nginx, that being the logic, and serving the user the raw HTML, that being the data.

So for a website in late 1990s:
```
|-----------------|
|     Web Page    |  <--- Presentation
|-----------------|
|-----------------|
|  Browser Logic  |  <---- Client-side logic
|-----------------|

|-----------------|
|  Server Logic   | <---- Server-side logic
|-----------------|
|-----------------|
|  Static Files   | <---- Database
|-----------------|

```

Now, the web has evolved since the late 1980s, when a fella at CERN called Tim Berners Lee wrote the first webserver so people could have a directory access to each other's computers. One cool story about the inception of web. These days web and HTTP are ubiquitous with TCP/IP and in the case of HTTP 3.0, QUIC/IP. But in CERN in the late 80s and early 90s, the network equipment was a far cry from the standards of TCP/IP. CERN at the time ran on a legacy LAN network, connected to a WAN that was in turn connected to several other labs around the globe. But the infrastructure of this network did not relegate back to ARPANET and as such was not on The Internet, let alone, any internet so to speak. Now I heard this on this channel called RetroBytes. I could be wrong about the extent to which this is true. Maybe this was only true at the very beginning and CERN quickly upgraded its network equipment, or it could have lasted longer.

Most of the credit for Web goes to the Netescape navigator. They were the ones who standardized HTML, a task later relegated to W3C, and HTTP/1.1 was standardized in 1996 by IETF, via RFC#2616. From then on several other RFCs have proposed new features for the protocol and two major versions since then have come out, HTTP/2.0 and HTTP/3.0. Whereas 60% of webservers support 2.0 only 5% support 3.0. A major switch from TCP to QUIC was made in HTTP/3.0. QUIC is a protocol first proposed at Google in 2012 with the first RFC coming out at 2021 in RFC#2021. QUIC, as previously mentioned, is based on UDP, but it's not completely connectionless. As mentioned in the RFC:


```
  QUIC is a connection-oriented protocol that creates a stateful
   interaction between a client and server.

   The QUIC handshake combines negotiation of cryptographic and
   transport parameters.  QUIC integrates the TLS handshake [TLS13],
   although using a customized framing for protecting packets.  The
   integration of TLS and QUIC is described in more detail in
   [QUIC-TLS].  The handshake is structured to permit the exchange of
   application data as soon as possible.  This includes an option for
   clients to send data immediately (0-RTT), which requires some form of
   prior communication or configuration to enable.

   Endpoints communicate in QUIC by exchanging QUIC packets.  Most
   packets contain frames, which carry control information and
   application data between endpoints.  QUIC authenticates the entirety
   of each packet and encrypts as much of each packet as is practical.
   QUIC packets are carried in UDP datagrams [UDP] to better facilitate
   deployment in existing systems and networks.

   Application protocols exchange information over a QUIC connection via
   streams, which are ordered sequences of bytes.  Two types of streams
   can be created: bidirectional streams, which allow both endpoints to
   send data; and unidirectional streams, which allow a single endpoint
   to send data.  A credit-based scheme is used to limit stream creation
   and to bound the amount of data that can be sent.
```

QUIC in other terms, is a stateful, connection-oriented protocol that uses a stream of packets to communicate between endpoint-to-endpoint. This endpoint can be user-server, server-user, or user-user. Everything related to keepalive, ttl, and packet revival is done now at the fourth layer instead of the third as per TCP. QUIC has also allowed for painless P2P, which has allowed libraries such as libp2p which is the basis for most modern blockchains to shine. This usually is what people mean by Web 3.0. Web 3.0 is crypto, but not the crypto that you think, necessarily. Web 3.0 means a more secure, peer-to-peer web that is less reliant on The Backbone and more targeted towards smaller servers. Security is also a factor in Web 3.0.

Another thing mostly associated with Web 3.0 is Web Assembly. We don't wish to focus on JavaScript, WebAssembly, HTML and CSS in this document. The first webpage, which you can see [here](http://info.cern.ch/hypertext/WWW/TheProject.html) used a very primitive XML-like data serialization and this was taken further in Netescape navigator, who invented HTML and wrote a renderer for it. To allow the user to execute certain actions, they invented JavaScript. It must be noted that back then, Java, the language, was not very popular, so I doubt that they had this language in mind when they named JavaScript. Java was only released in 1993 and JavaScript was invented in 1995. 2 years is barely any time for a language to build enough hootspa so another language will use its name as a reference. JavaScript is, or at least, used to be, the assembly of web. JavaScript could only access your browser, and nothing outside, stopping anyone form performing malicious actions via webpages. As of lately, there is a contender for JS in form of WASM or WebAssembly. If JS was 'like' the assembly of web, this language  IS the assembly of web! WASM reads like a very low-level, bare-metal language. WASM can access your system, in a limited way, only if the user allows. WASM is very adept, and powerful. CSS is used to make web pages pretty, what else can I say, I hate that thing. I'm not much of a web developer and I have no taste for web design. But these technologies are breads and butters of many people.

Web has moved from a loose collective of small, decentralized webpages to big websites, mostly social media, and the move was because the aforementioned technologies, intended for the end-user, were slowly becoming harder and harder. It was with the rollout of HTML 5.0 that most end-users running a small blog said screw it, and joined FaceScroll or whatever. 

Web is now, unlike at its infancy, dynamic. This dynamism is achieved through REST APIs through HTTP JavaScript. Another thing of note, Web is not secure, at least as much it can be. JavaScript now supports crpytography and TSL, successor to SSL, encrypt the packets through another set of handshakes. There exists barely any non-TSL-supporting websites as browsers outright refuse to navigate to these pages.

## The Structure of HTTP

Now that we know more about web, let's talk about its medium. I think this is the second time I'm promising to talk about HTTP but what do you expect from a bipolar dude doped on his 6th Ritalin of the day? I'm just writing this not to kill myself #notyourblog yeah yeah whatevs. Anyway before I show you the structure of HTTP, I am going to teach you how to retrieve it first.

So I have set up two HTML pages on my server via nginx. [This](http://horatio.chubakbidpaa.com:7777/stat_up/index.html) and [this](http://horatio.chubakbidpaa.com:7777/index.html). As you can see the server is listening on port 7777, and the first page is at location `/index.html` and the second is at a lower level, `/stat_up/index.html`.

If you open these pages on your browser you'll be greeted with the rendered HTML page. But that's just the Presentation layer of the Gartner model. If you so want to see the headers and the such you can press F12 and go to the network tab. But what if you wish to view the raw layer? What then?

Well netcat has you covered. This time we are not going to use `echo`. Just type:

```
nc horatio.chubakbidpa.com 7777
```

And in the REPL type these lines, just press enter to create a newline, don't use any other key.

```
GET /index.html HTTP/1.1
Host: horatio.chubakbidpaa.com
Accept: text/html
```

Now press enter twice.

In the request you sent to my server, you first determined that the method you wish your request to be is `GET`. Of the available methods are `POST`, `PUT` and `DELETE` also. There are several other methods that are supported by niche servers. One, for example, can add a `KITTEN` method. It all depends on the server, but HTTP/1.1 standard only allows for these four methods.

Now the next line are the request headers. HTTP request mirrors the HTTP response. An HTTP response and HTTP request only theoretically differ on the first line. Both contain three parts: the request/response line (for us it's GET, what is called the *URI* (/index.html) and the NC/VC), the header and the body. The former two which are seperated by two CRLFs from each other.

The headers are simple: our host is horatio.chubakbidpaa.com and we wish to accept `text/html` formatted data. 

The response which we'll receive is this:

```
HTTP/1.1 200 OK
Server: nginx/1.14.0 (Ubuntu)
Date: Wed, 22 Feb 2023 19:28:51 GMT
Content-Type: text/html
Content-Length: 204
Last-Modified: Wed, 22 Feb 2023 19:15:42 GMT
Connection: keep-alive
ETag: "63f669de-cc"
Accept-Ranges: bytes

<h1> Welcome to a World of Misery! </h1>

<p> This is an HTML document served by my server to demonstrate HTTP structure. </p>

<strong> Green Peas? In my soup? It's more ickier than you think! </strong>

```
The first line is the VC/NC of the response and the status code/success. Then comes the headers. And the body, an HTML plaintext, separated by the aforementioned two CRLFs. Had we specified an encoding, an encoding field would have been added. Server is nginx and Date is there too. 

These all rely on the standards set forward by IETF for HTTP/1.1. So what is so especially revolutionary about HTTP that has made it so dominant? Well it's mostly the URI directive. This directive which is the second word in the request after the method defines which resource on the server we wish to access. Compared to all the other protocols running on TCP that are concerned with file access, HTTP is the most straight forward, and easiest to parse, and plus, the simplicity of the URI leads to a concept known as URL or cross-resource linkage. 

Let's try another URI:


```
GET /stat_up/index.html HTTP/1.1
Host: horatio.chubakbidpaa.com
Accept: text/html
```

We'll get this:

```
HTTP/1.1 200 OK
Server: nginx/1.14.0 (Ubuntu)
Date: Wed, 22 Feb 2023 19:56:38 GMT
Content-Type: text/html
Content-Length: 285
Last-Modified: Wed, 22 Feb 2023 19:24:45 GMT
Connection: keep-alive
ETag: "63f66bfd-11d"
Accept-Ranges: bytes

<h1> Welcome to a World of Misery! </h1>

<p> This is an HTML document served by my server to demonstrate HTTP structure. </p>

<strong> Green Peas? In my soup? It's more ickier than you think! </strong>
<br />
<b> This is the same location in the server, except one lever deeper </b>
```

This is another resource on my server, on the same port. URIs point to unique resources. This is another file on my server. You can twist your server to refer the same resource with the same URI though, that is easy. But generally each URI = each resource. 

Also notice that HTTP response header gives you the length of the body. In the early 90s you would not have had such a nifty standard. A good, ubiquitous standard that everyone follows that allows for transfer of hypertext and hypermedia.

In one of the [earliest webpages](http://info.cern.ch/hypertext/WWW/WhatIs.html), Tim Berners Lee defined hypertext as:

```
Hypertext is text which is not constrained to be linear.

Hypertext is text which contains links to other texts. The term was coined by Ted Nelson around 1965 (see History ).

HyperMedia is a term used for hypertext which is not constrained to be text: it can include graphics, video and sound , for example. Apparently Ted Nelson was the first to use this term too.

Hypertext and HyperMedia are concepts, not products.
```

So according to Word of God, we may not only use HTTP to convey text, or non-linear, serialized data in the later-to-be-defiend HTML format, but we can also convey media such as images, videos, vector graphics and so on or so forth. What the browser allows, we can send. 

HTTP allows for chaining of URIs through URL links. Whenever someone clicks on a link on a webpage, a `Referer` field is added to the response that brings forth the new page. This is the rule, that is, and browsers may or may not implement it. Through the Referrer field, a website owner will know where his patrons are coming from.

## WWW and DNS

Within a network, servers are identified by an IP. Now that IP may have gone through a NAT, that is, Network Address Translation. A version 4 IP only allows for (2 ** n) - 1 unique addresses. If an IP address authority were to give a single network a unique IP for every single one of their hosts, then that would mean we would have run out of them ages ago. That is why translation is used. The destination address in the IP header is most of the times directed towards your network, not you as a single host. And then your network takes that address, translates it using a lookup table, recreates the checksum and forwards it towards you. This process can happen in several networks.

Or as it is with INET6 or IPv6, there can be (2 ** 128) - 1 addresses so we should not worry about running out of them yet. But they are very long. There should be a better way for accessing a server or a host.

That is why DNS has been invented. DNS is a protocol just like HTTP is. DNS is distributed, and within a single internet there could be unique domains that have counterparts in The Internet. But as it currently pertains, the authority for DNS are the 12 root servers. The same authority who controls the root servers also controls the TLDs, or top-level domains. One must kill oneself to get a TLD registered. But as of now most people just use .com. Besides that .com domain I have another .ir domain too.

Now a domain can contain subdomains. For example my subdomain for my server is horatio, just a random nod to the Bard. But most websites use the subdomain `www` to refer to the web-ish side of their server. 

## Web Frameworks and Web Services

We mentioned that within an HTTP request, the request structure mirrors that of a response structure.

```
REQ/RESP LINE   <--- method, uri, vc/nc, status
HEADERS         <--- properties of request/response
                <--- two CRLFs
BODY            <--- the body, optional in both
                <--- another two CRLFs
```

Ok so our last response had a body. How about sending requests with bodies?

In this section and the upcoming section I will implement an HTTP framework to not only show how to send a body along with the request, but to apply what we have learned.

Now this is the chance to combine two topics. First is sending bodies along requests, and second is using HTTP as a service, and not just a way to send hypertext and hypermedia for human viewing. In other words, HTTP is a good, good way to send data that is only data, not just data that is data AND information. I hope I'm making myself clear. An HTML plaintext is a mix of data and information but a web service communicating with another web service is just data.

There are two major schisms for sending data to, and from a web service. SOAP and REST, they are called, and REST is the clear winner of these two as most people use that one instead. The main difference between SOAP and REST is in the (de)serialization of data. Whereas in REST you are free to use any format, which most people use JSON for, in SOAP you can only use XML. In REST, also, your request format can be different from your response format. In SOAP, obviously not. 

These two are not just used for web services. They are used in webpages to communicate with the web server as well. In 2004 AJAX standard was created using XmlHttpRequest to partly update webpages. Before that you needed to refresh the entire page to change it. This evolved into using REST APIs to update the page partly.

Now REST APIs could operate in many ways. You can put the entire input to the API inside the request URI. Or you could put it inside the headers, or you could put it in the body, or you can share it across! That is the main difference between SOAP and REST, the former is very rigid in that regard.

So here's a question. How would a traditional webserver achieve this? A server such as nginx is based upon a series of rigid configurations. Even newer server software such as Caddy fail to come up with a solution for the question of 'HTTP-based API'.

So we find the answer in what is called a *web framework*. People often to struggle with differentiating web frameworks from web servers, at least linguistically speaking, but they have a very clear through-line which separates them. At least it does, in my opinion. 

A web server is a collection of configurations that map common HTTP directives and operations needed for servers, such as serving a resource, reverse-proxying, redirecting, etc, to a predefined custset of rules. So we have 1- a series of configurations 2- that map directives 3- to rules. 

But a web framework is a dynamic entity that is mostly used to send and receive data from the client, have the server process the data, and send it back. A web framework usually lacks features such as reverse proxy, and most of its redirects are temporary. It is not suggested to use a framework for operations that is well-suited better for a webserver. Web frameworks usually manifest themselves as third-party libraries. We have Express for NodeJS, Flask and FastAPI for Python, Rocket for Rust, Go's built-in net.http library and the many useless web frameworks for C because most C coders role their own joint, and so on and so forth.

A webserver or web framework must support concurrency. Before CPUs became multicore coroutines were used to spawn new workers that processed new TCP connections. This approach of coroutines is still alive and well in Python's Asyncio Streams API. 

## A Very Simple and Inadequate HTTP Framework Implementation

So as I said Python's Streams API allows for coroutine-based access to TCP, UDP and UDM. We are going to use TCP for now, and implement a very bad, bad HTTP framework. The aim of this application is to show how to implement a very simple HTTP framework. It does not show you how to parse HTTP properly. It dose not show you how to compose responses properly. I wrote this code in half an hour and debugged it in an hour. You can find another web framework I wrote in Rust, called [Samovar](https://github.com/chubek/Samovar) in my Github profile. That one is incomplete and inadequate too. Basically I write such things to learn more, it is not impractical to implement such applications yourselves as the pre-existing libraries trump anything that you will ever write. But if you, like me, are divorced completely from the world of backend development, it is entirely understandable if you decide to write a web server or a web framework. And if you never finished it, who cares.

So you can find my stinky hack-job of a 1-hour web framework, aptly called Stinky, [here](https://gist.github.com/Chubek/fd67206b69d69862d756a5b82310a9ce). I have already hosted it on my server. This time let us not use Netcat, let's use a proper HTTP client. We'll use cURL. You can also use Postman, which is basically like cURL but has an API. If you are on Windows, you can easily download the Windows build of cURL from its site.

So we invoke cURL like this:

```
curl -X POST horatio.chubakbidpaa.com:6666/morsetrans?to=morse  -H "Content-Type: application/json" -d '{"data": "Hello World"}' 
```

This basically sends this request to our framework:

```
POST /morsetrans?to=morse HTTP/1.1
Host: 0.0.0.0:1205
User-Agent: curl/7.81.0
Accept: */*
Content-Type: application/json
Content-Length: 23

{"data": "Hello World"}

```

What we do to parse this is, we first read the request until we reach the first CRLF tuplet (0xd 0xa 0xd 0xa or \r\n\r\n) which separates the body from the header. We the get the content length out of the header, and also return the request line. We read n bytes, n being the content length of the body. We then parse the body, the URI query (`?to=morse` which can also be `?to=latin`) and translate to Morse, or Latin. We then compose the response. We get this from cURL:

```
{"status_code": 220, "status_message": "Success translating Latin to Morse", "translated": "......-...-..--- .-----.-..-..-.."}
```

This is basically what a REST API is. We used JSON as the data format. We could have used XML or YAML or any other format. The aim was to carry an operation on the server and respond back. The URI in this case, `morsetrans`, is called an **endpoint**. If we send another endpoint that our framework won't support, we'll get a 404 message:

```
Resource Not Found
```

But this is body of our message that cURL is showing. In actuality our response looks like this:

```
HTTP/1.1 404 Not Found
Date: Thu, 23 Feb 2023 01:54:09 GMT
Server: CoolYourJets1.0
Content-Type: application/json
Content-Length: 19

Resource Not Found

```

Notice that we have an 0xa or `\n` at the end of our body. That is a part of our body. HTTP responses should not be terminated by anything except what the body terminates with. Anything after the CRLF-CRLF in the middle is our body.

And this goes without mentioning that with the non-erroneous response we get:

 ```
HTTP/1.1 200 Ok
Date: Thu, 23 Feb 2023 01:56:19 GMT
Server: CoolYourJets1.0
Content-Type: application/json
Content-Length: 128

{"status_code": 220, "status_message": "Success translating Latin to Morse", "translated": "......-...-..--- .-----.-..-..-.."}
```

By specifying `Content-Type: application/json` we help the client give context to the output. For example in a browser this would look like a pivot table and Postman will prettify it with JSON rules. Another thing is, if you try and use a different method than POST:

```
HTTP/1.1 405 Method Not Allowed
Date: Thu, 23 Feb 2023 02:17:35 GMT
Server: CoolYourJets1.0
Content-Type: application/json
Content-Length: 19

Method Not Allowed

```

**Warning**: As with any other HTTP frameworks that only serve one endpoint and are written in half an hour, Stinky will fail at many occasions. One of them is sending a request without a body. Don't do that! 

## CGI, RPC via HTTP, SSL/TSL and HTTP Proxies

Now we must mention two ways of executing programs via web. One of them is an old method called CGI or Common Gateway Interface. CGI allows for execution of scripts, say, Perl scripts, through an HTTP request. If you ever bought a cheap webhost you must remember a folder called `cgi-bin` being placed at the root of your file browser. That is where CGI scripts would go in such webhost. On the server you can configure it to be anywhere but most of the times it's in the same place. The Wikipedia [article](https://en.wikipedia.org/wiki/Common_Gateway_Interface) contains more info. I don't wanna delve into it much because honestly I am clueless about CGI, I tried to run a Perl blogging script back in the day and I was successful but besides that I have never dealt with it.

Another way to execute program commands via HTTP is called **RPC via HTTP** or Remote Procedural Call that is conveyed to the server through HTTP. Basically any protocol can be used to send remote calls but HTTP is relatively safe and sound.

One thing that makes HTTP safe is SSL or TSL. SSL stands for Secure Sockets Layer, and its cryptography is based on the RC4 stream cipher. RC4 is a cipher that allows for asymmetrical encryption and decryption of data through XOR'ing with a key that is send to the client through a handshake. The newer implementation of SSL is called TSL.  

HTTP also is used to proxy data through. There are two types of HTTP proxies, reverse and forward. Reverse proxy takes a request server-side and passes it to another URL or host within the same server. Forward proxy operates client-side, and receives the data filtered through an HTTP Proxy Server and renders it for the user. Forward proxies are very useful for anonymity and circumventing censorship. Mind you that such proxies don't necessary have to only proxy HTTP request .They can proxy all kinds of protocols IP-down.

## Finishing Thoughts

I don't think I need to add anything. HTTP good. HTTP important. Learn it. It is a simple protocol that has allowed the interne to flourish. It is not that ground-breaking. HTTP is like the lightbulb. Electricity existed before lightbulbs, but it was the lightulb that brought electricity into all abodes. And thus was it The Web, capital and definite, that brought The Internet, likewise capital and definite, into all homes.

If you have any questions you can find my Discord in my Github. Or ask me any questions you got here. And remember to remind me of any mistakes that I have made. 

Thanks.
