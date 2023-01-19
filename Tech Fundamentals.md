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

## DDoS Attacks

* Three main types: Application Layer (HTTP flood), Protocol Attack (SYN flood), Volumetric (DNS amplification).
* HTTP flood for example, takes advantage of the imbalance between a small request payload but a huge response payload.
* SYN flood just initiates a 3-way handshake with a fake SRC IP so the server will keep the connection in a hung state.
* DNS amplification uses DNS servers to make small requests payload with huge response payload but fake the SRC IP so the DNS reply is actally sent to legitimate servers and clog their network resources.

## SSL/TLS

* TLS is the "new" SSL.
* Part of a TLS session has multiple phase: 1. Cipher suites, 2. Authentication, 3. Key Exchange
* On Cipher Suites phase, Client/server agree on a set of ciphers to use. Server replies with its server certificate.
* Server certificate contains DNS of the server, the public key of the server and info about the CA.
* The CA is the authority the OS/Browser trusts and which has signed a CSR for the server which thus became a proper Certificate.
* On Authentication phase, the client confirms that the certificate is valid by checking the signature by the CA, the cert date, DNS name, etc. The client verifies then that the public key received matches the private key the server has.
* The Client generates pre-master key and send it to server via the assymetric encryption. It then becomes the master key that will be used for symmetric encryption between the client and the server from now on.

## VLANs

* Frame tagging (802.1q & 802.1ad) adds a new field in the Ethernet header. For 802.1q it's the VLAN TAG. For 802.1ad it's the customer VLAN TAG and the Service VLAN TAG.
* VLAN create separate L2 network segment.
* They are isolated, we have traffic isolation.
* It can be different customers / different networks and have different broadcast domains.
* 802.1q is VLAN and from the switch perspective, have two type of ports: Access (no VLAN tagging on the server end but added at the switch level) or trunk (port has VLAN tagging).
* 802.1ad is nested QinQ VLANs and allow a local LAN VLAN to span to a remote LAN via WAN with another TAG on the frame: the Service TAG which is a TAG provided by the ISP which identifies each different customers.

## Hashing

* DATA + Hashing Function = Hash
* Can't get the original data from Hash
* Same data = same hash for a given hashing algorithm
* Different data = different hash unless collision
* Older hashing alrogithm (i.e MD5) are more likely to create collision than more recent (sha2-256, to be used in prod).

## Digital Signatures

* Verify integrity (what) and authenticity (who).
* Hash of the data is taken, original data remains unaltered (integrity).
* Digitally sign the hash (using private key). Authenticates the hash.
* Using public key, we can verify that the hash has been created by the correct person.
* By hashin the original data with the same hashing algorithm, we can compare the hashes and confirm it hasn't been altered.

## DNS

* Domain registrar and DNS hosting provider are conceptually different, even though they can be provided by the same company.
* Registrar allows you to purchase a domain, the hosting provider will host the zone corresponding to that domain.
* If it's a different company, the registrar will ask you the NS info from the hosting provider so it can then pass this info to the TLD NS for registration of the new zone.

## DNSSEC

* It's a layer on top of DNS -> There is compatibility for non-DNSSEC enabled device.
* It adds data origin authentication (this data is coming from this zone) and data integrity protection (this data wasn't modified in transit). DNS chain of trust.
* RRSET = set of all resource records with the same name and same type (ex: icann.org MX).
* Two sets of key pairs: ZSK and KSK. Private ZSK and KSK are NOT in the zone, they are stored in a safe offline location.
* Private ZSK will digitally sign RRSET, and put that value in a RRSIG corresponding of the RRSET (for ex. icann.org MX).
* The public ZSK is also in its own zone record.
* The public ZSK is digitally signed by the private kSK and stored in RRSIG DNSKEY record.
* The public KSK has also a record in the zone. And it is directly referenced by the parent zone. The KSK is the POINT OF TRUST for the zone.
* Having two sets of keys allow us to rotate the ZSK often without having to update the reference link from parent zone (for ex. .org).
* The chain of trust and the reference is built in the following way: The public KSK in icann.org has its hash computed in the parent (.org) zone and stored in a record (called DS records). This/theses records are themselves RRSET, and will thus be digitally signed by .org zone private ZSK. We have same the same process as in the child. The chain goes up to the root zone, where the public KSK and private KSK are explicitely trusted by DNSSEC enabled resolvers (trust anchor).

## RPO/RTO

* RPO - How much data (MAX TIME) a business can lose.
* ...worst case = time between successful backups.
* ...more frequent backups = more cost = lower RPO.
* RTO - How long restore time a business can tolerate.
* ...end to end from identification through to handover.
* Reduce via planning, monitoring, notification, process, spare hardware, training, more efficient systems.
* Different businnes and systems will have difference RPO and RTO values.