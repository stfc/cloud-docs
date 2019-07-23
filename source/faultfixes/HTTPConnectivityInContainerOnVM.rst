==================
Connectivity https services seem broken from inside a Docker Container
==================


#######
Symptoms
#######
Within a docker container, a “curl http://google.com” works fine, but a “curl https://google.com” would fail. However, outside of the container on the VM natively, both of these will work.

#########
Explanation:
#########
This is seen a lot on Cloud hosts where the MTU size is not the usual 1500 bytes, and so it is wise to “clamp” the initiating TCP session for an application to use the same segment size as the MTU of the local host – rather than assuming a possibly invalid (and too large) value.

#######
Solution
#######
Since version 17 of Docker, the following entry needs to be added to the iptables policy on the host that is running the containers:-
iptables -I FORWARD -p tcp --tcp-flags SYN,RST SYN -j TCPMSS --clamp-mss-to-pmtu
This sets the “maxi,um segment size” value of a TCP Syn packet when it initiates a connection as part of “path MTU” negotiation. This is intended to get past the issue of “icmp 3 code 4” packet blocking issues (by the sending of the original packet)  where the “Don’t fragment” flag is set on an IP packet, but the packet is too large and the sending router/host does not receive the icmp 3 code 4 packet to change and resend the “too large” packet as a smaller size. This is often an issue when using tunnelling or “packet sleeving” – GRE and IPSEC packets as well.

########
References
########
https://www.frozentux.net/iptables-tutorial/chunkyhtml/x4721.html
https://stackoverflow.com/questions/47551873/no-http-https-connectivity-inside-docker-container
