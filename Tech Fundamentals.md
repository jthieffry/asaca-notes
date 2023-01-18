# Key Points

## Encryption

* Encryption-at-rest: usually only one entity involved, usually for storage. If storage is stolen, no compromission.
* Encryption in transit: usucally between multiple entities, usually for communication. Data encrypted during transit, usually via the setup of an encryption tunnel.
* Encryption objects: Algorithm (blowfish, aes, rc4, des, etc.) on key + plaintext -> cyphertext
* Symmetric encryption: Key used is the same for the encryption and decryption -> Cheap but how do you share the key between sender and receiver securely ?
* Asymmetric encryption: Private key used to decrypt, public key used to encrypt -> More expensive, but safe. Usually, communication starts with Asymmetric to share a symmetric key, then continues with symmetric encryption.
* Signing is to ensure somebody's identity. The party who wants to be identified will send a message encrypted with his private key. The receiver will decrypt with the sender's public key and then confirm that the sender is indeed who he claims to be.
* Steganography is hiding data into another piece of data, to hide the fact that an encrypted message has been sent.

## Network Basics

* L1 only deals with physical medium (voltage, etc). Hub are L1 device, so they don't have address, and just broadcase to all their IF. Single failure domain, since it also broadcast collisions.
* L2 (ex. Ethernet): adds addressing (MAC Address) and the concept of frames (with header, payload and CRC). Adds CSMA/CD. Swiches are smart L2 devices. In addition to hub, they work at the frame level and store a MAC address table that they populate based on their ports traffic and use it for forwaring frame to the appropriate IF. They don't forward collisions so they have failure domains equal to their port in use.
* L3 (ex. IP) allows routing of LAN. Adds IP addressing, etc. Encapsulates L4 data. L3 itself is encapsulated in L2 and the L3 content doesn't change (usually, except NAT) for the whole duration of the transport.
* L3 Routing is done via route table in which routers select the most specific route to a destination. The router has interface to other L2 network and can thus encapsulate the transiting L3 packet into a L2 frame by knowing the MAC address of the remote L2 network GW via ARP.
* For LAN communication with IP, routers are not needed. Hosts determine MAC from IP themselves via ARP. They can then build their L2 frames.
* L3 adds IP addresses, ARP, Route & Route Tables, Router (moves from SRC to DST, decapsulating/encapsulating in L2 along the way) & device to device communications over the Internet.
* L4 with TCP deals with Segments. The protocols adds SRC/DST ports, Window (flow control), Acknowledgment (error checks) and Sequence (correct order) flags, among other things. It is a "Connection" oriented protocol.
* L5 (Session) actually encompasses the IN channel and OUT channel from a TCP connection. It's mostly relevant when talking about firewalls. Stateless firewalls (ex. AWS NACL) don't understand the concept of sessions and we need to add two rules for a proper TCP communication to happen. Stateful FW (ex. Secgroups) understand that, so only the allowing the incoming rule is enough.

## NAT

* Translates private IPs to Public, used to overcome ipv4 shortage and has security benefits.
* 3 Types of NAT: Static NAT (SNAT, AWS Internet GW) -> 1 private to 1 (fixed) public, Dynamic NAT (DNAT) -> 1 private to the 1st available Public (pool), Port Address Translation (PAT) -> many privates to 1 public (via ports, standard setting in home setup. What AWS NATGW does).
* For SNAT: The router maintains a table that matches public / private IP address allocation, and changes DST or SRC (depending of the channel in the TCP connection) as the packet flows in/out.
* For DNAT: Similar to SNAT but with multiple available public IP address. It's still 1:1 mapping though, meaning that only one private address can take one public address at a given time.
* PAT: The NAT mapping table has each entry containing the following fields: private ip / private port / public ip / public port. It can then forward back the request from external to the correct private entity. In this case, multiple private addresses can take the same public IP, because the mapping is also done at the port level. But in this case, external entities CANNOT initiate the connection to private, since the mapping doesn't exist yet in the mapping table at the NAT level.

## Subnets

* Private ranges: 10.0.0.0 -> 10.255.255.255 (1x ClassA), 172.16.0.0 -> 172.31.255.255 (16x ClassB), 192.168.0.0 -> 192.168.255.255 (256x ClassC)
* CIDR allows us to break the traditional Class paradigm and design subnets the way we want thanks to the network prefix (NETMASK).