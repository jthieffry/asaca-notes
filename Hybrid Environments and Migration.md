# Key Notes

## Border Gateway Protocol (BGP)
* Autonomous System (AS): Routers controlled by one entity. A network in BGP (actual physical implementation is abstracted - blackbox for BGP).
* ASN (AS Number) are unique and attributed by IANA (0-65535). Range 64512-65534 is private and can be used for private network.
* BGP operates over tcp/179 - It's reliable.
* Not automatic - Peering is manually configured. 
* BGP is path-vector protocol. It exchanges the best path to a destination between peers. This path is called ASPATH.
* iBGP - internal bgp, used for routing within an AS / eBGP - external bgp, used for routing between AS.
* A BGP routing table is present for each AS. One entry has the following field:
    - Destination (ip range): the wanted destination for a packet.
    - Next hop: ip address of the next hop to get to dest. 0.0.0.0 means it's the local network (stays within the AS).
    - ASPATH: the succession of ASN to take in order to get to DEST. For ex: 201,200,I. I is the origin - The network was learned from a from a locally connected network.
* An AS will advertise the shortest path it knows to all his peers.
* If a destination has multiple options, the shortest path is preferred. Network admin can leverage AS Path Prepending to make a path artificially looks longer and then not be preferred.

## IPSec VPN Fundamentals
* IPSEC is a group of protocols that sets up secure tunnels over insecure networks between two peers (local and remote).
* Provides authentication and encryption.
* Data in scope for this tunnel is so called "interesting traffic". 
* General overview, IPSEC has two main phases:
    1. IKE Phase 1 (Slow and heavy)
        - Authenticate (via preshared key/password or certificate)
        - Using asymmetric encryption to agree on and create a shared symmetric key.
        - IKE SA created (phase 1 tunnel)
    2. IKE Phase 2 (fast and agile)
        - Use the key agreed on phase 1.
        - Agree on encryption method and keys used for the bulk of data transfer.
        - Create IPSEC SA Phase 2 (technically running over the phase 1 tunnel)
* Two types of VPN can be set. Phase 1 tunnel stays while phase 2 are created as there is matching interesting traffic, otherwise destroyed.
    1. Policy-based VPNs:
        - rule sets match traffic => a pair of SA
        - different rules/security settings
        - So has potentially multiple SAs between one src and dst.
    2. Route-based VPNs:
        - target matching (prefix)
        - matches a single pair of SAs