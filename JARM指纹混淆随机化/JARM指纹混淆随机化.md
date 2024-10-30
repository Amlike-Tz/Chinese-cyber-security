# JARM fingerprint obfuscation randomization

Author: Wind Up

## C2 Recognition Based on JARM Fingerprinting

 JARM works by proactively sending 10 specially constructed TLS Client Hello packets to the target TLS server to extract unique responses in the TLS server and capture specific attributes of the TLS Server Hello responses, and then hash the aggregated TLS server responses in a specific way to generate a JARM fingerprint.

Because the parameters in the Client Hello are different, the final Server Hello returned are different, by sending specially constructed Client Hello handshake packets to get the special Server Hello response corresponding to it, as a basis, the final TLS Server fingerprint is generated.

JARM sends different TLS versions, ciphers, and extensions in different orders to collect unique responses.

- Does the server support the TLS 1.3 protocol?
- Will it negotiate TLS 1.3 with 1.2 ciphers?
- If we sort the ciphers from weakest to strongest, which cipher will it choose?

These are the types of anomalous questions that JARM essentially asks the server to extract the most unique responses. These 10 responses are then hashed to produce a JARM fingerprint.

TLS Client Hello Request Traffic

Wireshark Filter.

``
ssl.handshake.type == 1
```

! [img](. /iamge/1.png)

TLS Server Hello response

! [img](. /iamge/t010872807d1a97f380.png)

In most cases JARM fingerprinting is used to corroborate the TLS service and tag it to associate the service. Of course, ideally, we would be able to uniquely point to a target C2 facility through JARM fingerprinting, but in practice JARM fingerprinting is not unique across C2 facilities deployed on different servers, and there are a number of factors that can affect the results of the scan. Therefore, we cannot directly associate the fingerprints with a C2 service as a behavioral mapping, but only serve as a supporting evidence.

In the process of developing RedGuard, I found that because the role of RG is to control the front-end traffic, so as to realize the invisibility of the back-end C2 services, when the blue team analyzes the traffic interactions, the scanning results of JARM fingerprints for RG are the same in most different environments, that is, in the process of analyzing the fingerprints, the fingerprints can be used to corroborate the attacking facilities, thus destroying the attacks we want. facility, thereby undermining the stealth we want to expect to achieve.

Infrastructure changes (e.g., IP address, hosting platform) do not affect the JARM signature, making it difficult to circumvent it in conventional ways.

Factors affecting server-side JARM fingerprinting:

- Operating system and version
- Libraries used and versions
- Order in which libraries are called
- Custom Configurations
- ........

That is to say, if we want to affect the final results of the JARM fingerprint scanning for the server side, we have to start from the above factors to do, there are currently two solutions:

First, replay the TLS Server Hello response

Second, change the TLS Server configuration CipherSuites encryption suite

The first way is also listening to a specific client Hello TCP server, and then in the C2 server to capture these specific Client Hello (each request has repeated bytes, using these bytes to identify each specific Client Hello handshake packet), the stability of the 10 special construction of the Client Hello response replay to achieve the effect of changing the real JARM fingerprint.

```
if bytes.Contains(request, [] byte { 
     0x00, 0x8c, 0x1a, 0x1a, 0x00, 0x16, 0x00, 0x33, 0x00, 
     0x67, 0xc0, 0x9e, 0xc0, 0xa2, 0x00, 0x9e, 0x00, 0x39 
     , 0x , 0xc0, 0x9f, 0xc0, 0xa3, 0x00, 0x9f, 0x00, 
     0x45, 0x00, 0xbe, 0x00, 0x88, 0x00, 0xc4, 0x00, 0x9a, . 
     ... ... 
}) { 
     fmt.Println("replaying: tls12Forward ”) 
     conn.Write([] byte { 
         0x16, 0x03, 0x03, 0x00, 0x5a, 0x02, 0x00, 0x00, 
         0x56, 0x03, 0x03, 0x17, 0xa6, 0xa3, 0x84, 0x80, 
         0x0b, 3,d, 0xbb, 0x 0xe9, 0x3e, 0x92, 0x65. 
         0x9a, 0x68, 0x7d, 0x70, 0xda, 0x00, 0xe9, 0x7c, 
         ... ... 
     }) 
}
```

After implementing replies to all ten different requests, the complete signature can be forged.

! [img](. /iamge/t01419e6c76ed33340d.png)

 This is a lazy approach, the final result can really confuse the results of the JARM scan, but I think it is too monotonous, it should be noted that the original response of the ServerHello he can not be arbitrarily modified, because in the development of the tool the normal interaction of the traffic is the most important, arbitrary modification of some of the above factors affecting the JARM scan may result in the failure of normal communication problems. problems. That is to say, he replayed the ServerHello packet is required to listen to a normal service JARM scanning response to achieve, and is not suitable for us to apply to the tool to achieve the effect of obfuscation, we need a simple and stable method.

Of course, there is an extension of this approach, that is, first of all, randomly obtain a list of normal sites for JARM fingerprinting scanning, so as to obtain their ServerHello response and then, directly as the recognition of the JARM scanning handshake packet response packet replay, so this is one of the most reasonable way.

 The second is also a way mentioned in the Amsterdam 2021 HITB issue “COMMSEC: JARM Randomizer: Evading JARM Fingerprinting”, where the author has partially open-sourced a derivative tool, JARM Randomizer, on github, and it is not difficult to see that by reading its code. In fact, it is very similar to the way I initially thought of, and the factors that ultimately affect JARM use the CipherSuites encryption suite (the suite of encryption algorithms CipherSuite consists of various types of basic encryption algorithms)**.

The implementation code:

! [img](. /iamge/t01fc5e9fcb985de1d5.png)

As shown above, for the reverse proxy's TLS Config CipherSuites setup, I provided 15 different ways of encryption suites, 1/2/3 .... different combinations of cipher suites ultimately get the JARM fingerprint results are not the same (too many cipher suites will lead to the metaphysical problem of not being able to communicate properly), here I have these 15 kinds of cipher suites for the random combination of the way, 1-2 random values for the combination of these CipherSuites, there are probably dozens of different random JARM fingerprints generated.

The final result is as follows:

! [img](. /iamge/t01bf1f398318998c43.png)

As you can see, the final effect applied to the tool is that every time the RG is started, its JARM fingerprint is not the same, and a brand new JARM fingerprint will be automatically generated based on the above obfuscation, which prevents spatial mapping from scanning as a way of correlating the deployment of RG infrastructure on the public network. Most of the mapping platforms in the market nowadays will integrate and scan JARM fingerprints by default, probably because of its fast scanning efficiency, low cost of scanning resources, and certain corroborative pointers to C2 facilities.

RedGuard is a C2 infrastructure front-end traffic control facility that circumvents Blue Teams, AVs, and EDRs checks.

## Reference Link:

https://github.com/wikiZ/RedGuard

https://github.com/netskopeoss/jarm_randomizer

https://grimminck.medium.com/spoofing-jarm-signatures-i-am-the-cobalt-strike-server-now-a27bd549fc6b

https://conference.hitb.org/hitbsecconf2021ams/sessions/commsec-jarm-randomizer-evading-jarm-fingerprinting/