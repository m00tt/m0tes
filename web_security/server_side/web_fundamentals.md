# Basic Fundamentals to Web Security

1. [DNS](#dns)
   - [Domain Hierarchy](#domain-hierarchy)
   - [Record Types](#dns-record-types)
   - [DNS Request](#dns-request)
   - [Useful Command](#useful-commands)

2. HTTP

# DNS
DNS stands for Doamin Name System and help you to communicate with devices on the internet without remembering its IP Addresses. <br />
Basically, DNS records link a domain name with its IP address (and more). <br />
So instead of remembering 10.10.10.10, you can remember m00tt.com instead that will be translated by a DNS Server.

<br />

## Domain Hierarchy
![Domain Hierarchy](/assets/domain-hierarchy.jpg)
<br />
A Domain Hierarchy is a system used to sort the parts of a domain according to their importance, is divided into 3 levels
- Root Domain
    - The Root Domain (“.”) is the highest hierarchy level of any domain name system.
- TLD (Top-Level Domains)
    - A TLD is the most righthand part of a domain name, is the end part of a URL. <br />
    There are two types of TLD :
        - gTLD (Generic Top Level) : historically was meant to tell the user the purpose of the domain name; for example, a .com would be for commercial purposes, .org for an organization, .edu for education and .gov for government.
        - ccTLD (Country Code Top Level Domain) : was used for geographical purposes, for example, .ca for sites based in Canada, .co.uk for sites based in the United Kingdom and so on.
- Second-Level Domains
    - The Second-Level Domain represents the domain name, when registering a domain the second-level domain is limited to 63 characters (except TLD) and you can only use lowercase letters (a-z), numbers (0-9) and hyphens (cannot start or end with hyphens or have consecutive hyphens).
    - A subdomain is a subset of a specific website. It allows you to categorize your website into one or more sections. Is the leftmost part in the url and you use a dot (.) to separate the subdomain from Second-Level Domain.<br />
    A subdomain name has the same restrictions of creating a Second-Level Domain, also you can use multiple subdomains, separated by a dot (.), the important thing is that you do not exceed 253 characters

<br />

## DNS Record Types
As anticipated before, DNS is not only used for translate domain into IP Adrressed. <br /> 
There are several types of DNS records.

### Commonly Used

| Record Type                                                    | Description                                                                |
|----------------------------------------------------------------|----------------------------------------------------------------------------|
| A     | The "A" in A record stands for "address". An A record shows the IPv4 address for a specific hostname or domain.        |
| AAAA  | Like A record, point to the IP address for a domain. However, this DNS record type is different in the sense that it points to IPv6 addresses. |
| CNAME | A canonical name (CNAME) record redirects a domain to a different domain. |
| PTR   | A pointer (PTR) record is the reverse of an A or AAAA record. A PTR record resolves IPv4 or IPv6 addresses to domain names. Reverse DNS lookups use PTR records. |
| NS    | A name server (NS) record provides a list of the authoritative DNS servers (also called name servers) responsible for the domain that you’re querying. At the end of the DNS query chain, an authoritative name server is the final arbiter for DNS resource records. |
| SOA   | Every DNS zone requires a start of authority (SOA) record. RFC 1035 specifies SOA record formats. The SOA record stores important information about the zone, such as its primary authoritative name server and the administrator’s email address. |
| MX    | A mail exchanger (MX) record stores the domain names of mail servers responsible for receiving emails on behalf of a domain. |
| TXT   | A text (TXT) record can store any type of descriptive information in text format. |
| SRV   | Using this DNS record type, it's possible to store the IP address and port for specific services. |

<br />

### Used for DNSSEC

| Record Type                                                    | Description                                                                |
|----------------------------------------------------------------|----------------------------------------------------------------------------|
| DNSKEY     | A DNSKEY-record holds a public key that resolvers can use to verify DNSSEC signatures in RRSIG-records.       |
| DS         | DS-records are used to secure delegations (DNSSEC). A DS-record with the name of the sub-delegated zone is placed in the parent zone along with the delegating NS-records. This DS-record references a DNSKEY-record in the sub-delegated zone. |
| NSEC       | An NSEC-record links to the next record name in the zone (in DNSSEC sorting order) and lists the record types that exist for the record's name. |
| NSEC3      | An NSEC3-record links to the next record name in the zone (in hashed name sorting order) and lists the record types that exist for the name covered by the hash value in the first label of the NSEC3 -record's own name. These records can be used by resolvers to verify the non-existence of a record name and type as part of DNSSEC validation. NSEC3-records have the same functionality as NSEC-records, except NSEC3 uses cryptographically hashed record names to prevent enumeration of the record names in a zone. |
| NSEC3PARAM | An NSEC3PARAM-record is used by authoritative DNS servers to calculate and determine which NSEC3-records to include in responses to DNSSEC requests for non-existing names/types. |
| RRSIG      | An RRSIG-record holds a DNSSEC signature for a record set (one or more DNS records with the same name and type). Resolvers can verify the signature with a public key stored in a DNSKEY-record. |

<br />

### Less Commonly Used
| Record Type                                                    | Description                                                                |
|----------------------------------------------------------------|----------------------------------------------------------------------------|
| AFSDB    | An AFSDB-record maps a domain name to an AFS (Andrew File System) database server. The server name points to an A-record for the database server, and the sub-type indicates server type: 1. AFS version 3.0 volume location server for the named AFS cell. 2. DCE authenticated server. |
| CAA       | Allows a DNS domain name holder to specify one or more Certification Authorities (CAs) authorized to issue certificates for that domain. |
| CERT      | CERT-records store certificates and related revocation lists (CRL) for cryptographic keys. |
| DHCID     | DHCID-records store Dynamic Host Configuration Protocol (DHCP) Information, and can be created and used by some DHCP servers and clients. |
| DNAME     | A DNAME-record is used to map / rename an entire sub-tree of the DNS name space to another domain. It differs from the CNAME-record which maps only a single node of the name space. |
| HINFO     | A HINFO-record specifies the host / server's type of CPU and operating system. This information can be used by application protocols such as FTP, which use special procedures when communicating with computers of a known CPU and operating system type. |
| HTTPS     | HTTPS-records allows browsers to efficiently obtain complete instructions for accessing a web-site for a domain name - including supported protocols (HTTP/1.1, 2, 3, etc.), ip address(es), port number, and public keys (all optional) - saving the browser from doing a number of DNS lookups and other protocol negotiation steps. |
| LOC       | This record type is used to specify geographical location information about hosts, networks, and subnets. |
| NAPTR     | NAPTR-records are used to store rules used by DDDS (Dynamic Delegation Discovery System) applications. |
| RP        | An RP-record specifies the mailbox of the person responsible for a host (domain name). A SOA-record defines the responsible person for an entire zone, but a zone may contain many individual hosts / domain names for which different people are responsible. |
| TLSA      | TLSA records are used to specify the keys used in a domain's TLS servers. | 

<br />

## DNS Request
What happens when you make a DNS request?

![DNS Lookup Diagram](/assets/dns-lookup-diagram.webp)

1. When you request a domain name, your computer first checks its local cache to see if you've previously looked up the address recently; if not, a request to your Recursive DNS Server will be made.

2. A Recursive DNS Server is usually provided by your ISP, but you can also choose your own. This server also has a local cache of recently looked up domain names. If a result is found locally, this is sent back to your computer, and your request ends here. If the request cannot be found locally, a journey begins to find the correct answer, starting with the internet's root DNS servers.

3. The root servers act as the DNS backbone of the internet; their job is to redirect you to the correct Top Level Domain Server, depending on your request. If, for example, you request www.m00tt.com, the root server will recognise the Top Level Domain of .com and refer you to the correct TLD server that deals with .com addresses.

4. The TLD server holds records for where to find the authoritative server to answer the DNS request. The authoritative server is often also known as the nameserver for the domain. You'll often find multiple nameservers for a domain name to act as a backup in case one goes down.

5. An authoritative DNS server is the server that is responsible for storing the DNS records for a particular domain name and where any updates to your domain name DNS records would be made. Depending on the record type, the DNS record is then sent back to the Recursive DNS Server, where a local copy will be cached for future requests and then relayed back to the original client that made the request. DNS records all come with a TTL (Time To Live) value. This value is a number represented in seconds that the response should be saved for locally until you have to look it up again. Caching saves on having to make a DNS request every time you communicate with a server.

<br />

## Useful Commands
1. DNS Type: <br />
 The content of the nslookup result are the IPv4 (four-figure) and IPv6 addresses (longer, divided with colons) of the example domain. 
 ```
 nslookup website.com
 ```

2. A <br />
The content of the nslookup result with record type A will be the ipv4 address of the example domain.
```
nslookup -type=A website.com
```

3. AAAA <br />
The content of the nslookup result with record type AAAA will be the ipv6 address of the example domain.
 ```
nslookup -type=AAAA website.com
```

3. CNAME <br />
The content of the nslookup result with record type CNAME will be the domain to which our subdomain redirects.
```
nslookup -type=CNAME subdomain.website.com
```

4. NS <br />
The content of the nslookup result with record type NS will be the authoritative server for a specific domain
```
nslookup -type=NS website.com
```

5. PTR <br />
You can verify if an IP address belongs to a domain name by performing a reverse DNS query. For this purpose, you will need to check the PTR record that links an IP address to a domain name. You will need to put the IP address in reverse (185.136.96.96 changes to 96.96.136.185), and you need to add in-addr.arpa because it is stored in arpa’s top-level-domain.
```
nslookup -type=PTR 96.96.136.185.in-addr.arpa
```

6. MX <br />
The content of the nslookup result with record type MX will be the Domain name mailserver (mail exchanger)
```
nslookup -type=MX website.com
```

7. Enable Debug Mode <br />
Debug mode provides important and detailed information both for the question and for the received answer.
```
nslookup -debug website.com
```

<br />

# HTTP