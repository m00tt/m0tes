# Basic Fundamentals to Web Security

1. [DNS](#dns)
   - [Domain Hierarchy](#domain-hierarchy)
   - [Record Types](#dns-record-types)
   - [DNS Request](#dns-request)
   - [Useful Command](#useful-commands)

2. [HTTP](#http)
   - [Requests and Responses](#requests-and-responses)
   - [Making a Request](#making-a-request)
   - [HTTP Methods](#http-methods)
   - [HTTP Status Codes](#http-status-codes)
   - [Headers](#headers)
   - [Cookies](#cookies)
       - [Session](#session-cookie)
       - [Persistent](#persistent-cookie)
       - [Third-Part](#third-part-cookie)

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
HTTP (HyperText Transfer Protocol) is the set of rules used for communicating with web servers for the transmitting of webpage data, whether that is HTML, Images, Videos, etc.
<br />
<br />
HTTPS (HyperText Transfer Protocol Secure) is the secure version of HTTP. It uses SSL certificates to encrypt client-server communication.
In this way, thank to RSA crittography, gives also you assurances that you're talking to the correct web server and not something impersonating it.

## Requests and Responses
When we access a website, your browser will need to make requests to a web server for assets such as HTML, Images, and download the responses. Before that, you need to tell the browser specifically how and where to access these resources, this is where URLs will help.

![HTTP Url](/assets/http_url.png)

- <b>Scheme or Protocol</b>: This instructs on what protocol to use for accessing the resource such as HTTP, HTTPS, FTP and so on.

- <b>User and Password</b>: Some services require authentication to log in, you can put a username and password into the URL to log in.

- <b>Host</b>: The domain name or IP address of the server you wish to access. Check [DNS](#dns) section eventually.

- <b>Port</b>: The Port that you are going to connect to, usually 80 for HTTP and 443 for HTTPS, but this can be hosted on any port between 1 - 65535.
In case of use a not standard port, you must specify a connection port on the URL.

- <b>Path</b>: The file name or location of the resource you are trying to access, manly the web page you are looking for.

- <b>Query String</b>: Extra bits of information that can be sent to the requested path. For example, /blog?id=1 would tell the blog path that you wish to receive the blog article with the id of 1.

- <b>Fragment</b>: This is a reference to a location on the actual page requested. It refers to HTML id tag  `<div id="task3">`

<br />

## Making a request
<br />

![HTTP Request](/assets/http_request.png)

Request example
```http
GET / HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0 Firefox/87.0
Referer: https://example.com/

```

- <b>Line 1</b>: This request is sending the GET method, request the home page with / and telling the web server we are using HTTP protocol version 1.1.

- <b>Line 2</b>: We tell the web server we want the website example.com

- <b>Line 3</b>: We tell the web server we are using the Firefox version 87 Browser

- <b>Line 4</b>: We are telling the web server that the web page that referred us to this one is https://example.com

- <b>Line 5</b>: <u>HTTP requests always end with a blank line to inform the web server that the request has finished</u>.

Response example using Postman
<br />

![HTTP Request](/assets/http_get.png)



## HTTP Methods
<br />

HTTP methods are a way for the client to show their intended action when making an HTTP request. There are a lot of HTTP methods but the most common are:
- <b>GET Request</b>
    - This is used for getting information and resources from a web server.
```http
GET /page2 HTTP/1.1

GET / HTTP/1.1

```
```bash
curl https://example.org
```

- <b>POST Request</b>
    - This is used for submitting data and resources to the web server and potentially creating new records
```http
POST /index.html HTTP/1.1
username=user1&password=secret

POST / HTTP/1.1
param1=value1&param2=value2

```
```bash
curl -X POST  https://example.org/ -d "username=user1&password=secret"
```

- <b>PUT Request</b>
    - This is used to store data and resources on a web server (used for example to update information)
```http
PUT /existing.html/json HTTP/1.1
Content-type: application/json
username=user2

PUT / HTTP/1.1
Content-type: application/json
param1=value1

```
```bash
curl -X PUT https://example.org -H "Content-Type: application/json" -d '{"username": "user2"}'
```
- <b>DELETE Request</b>
    - This is used for deleting information nd resources from a web server.
```http
DELETE /new.html HTTP/1.1

DELETE / HTTP/1.1

```
```bash
curl -X DELETE https://example.org
```
- <b>HEAD Request</b>
    - This is used to recover only the response header without the resource
```http
HEAD /new.html HTTP/1.1

HEAD / HTTP/1.1

```
```bash
curl -I https://example.org
curl --head https://example.org
curl -X HEAD https://example.org
```
- <b>OPTIONS Request</b>
    - This is used to require the list of the methods allowed by the server.
```http
OPTIONS /index.html HTTP/1.1

OPTIONS * HTTP/1.1
```
```bash
curl -X OPTIONS https://example.org -i
```

`-i` params is used for include the response HTTP headers in result data.


<br />

## HTTP Status Codes
<br />

| Code Ranges                                                           | Description                                                        |
|----------------------------------------------------------------|--------------------------------------------------------------------|
| 100-199 - Information Response | These are sent to tell the client the first part of their request has been accepted and they should continue sending the rest of their request. These codes are no longer very common. |
| 200-299 - Success | This range of status codes is used to tell the client their request was successful. |
| 300-399 - Redirection | These are used to redirect the client's request to another resource. This can be either to a different webpage or a different website altogether. |
| 400-499 - Client Errors | Used to inform the client that there was an error with their request. |
| 500-599 - Server Errors | This is reserved for errors happening on the server-side and usually indicate quite a major problem with the server handling the request. |

<br />

| Common Codes                                                          | Description                                                        |
|----------------------------------------------------------------|--------------------------------------------------------------------|
| 200 - OK | The request was completed successfully. |
| 201 - Created | A resource has been created (for example a new user or new blog post). |
| 301 - Permanent Redirect | This redirects the client's browser to a new webpage or tells search engines that the page has moved somewhere else and to look there instead. |
| 302 - Temporary Redirect | This redirects the client's browser to a new webpage or tells search engines that the page has moved somewhere else and to look there instead. |
| 400 - Bad Request | This tells the browser that something was either wrong or missing in their request. This could sometimes be used if the web server resource that is being requested expected a certain parameter that the client didn't send. |
| 401 - Not Authorised | You are not currently allowed to view this resource until you have authorised with the web application, most commonly with a username and password. |
| 403 - Forbidden | You do not have permission to view this resource whether you are logged in or not. |
| 405 - Method Not Allowed | The resource does not allow this method request, for example, you send a GET request to the resource /create-account when it was expecting a POST request instead. |
| 404 - Page Not Found | The page/resource you requested does not exist. |
| 500 - Internal Service Error | The server has encountered some kind of error with your request that it doesn't know how to handle properly. |
| 503 - Service Unavailable | This server cannot handle your request as it's either overloaded or down for maintenance. |


<br />

## Headers
<br />

Headers are additional information produced by the interaction between the browser and the server.
There are two types of headers:
- <b>Headers related to the request</b>: these are headers that are sent form the client to the server. These headings are characterized by:
    - <b>Host</b>
        - Specifies the server where the requested resource is hosted
    - <b>User-Agent</b>
        - This is your browser software and version number, telling the web server your browser software helps it format the website properly for your browser and also some elements of HTML, JavaScript and CSS are only available in certain browsers.
    - <b>Content-Length</b>
        - When sending data to a web server such as in a form, the content length tells the web server how much data to expect in the web request. This way the server can ensure it isn't missing any data.
    - <b>Accept-Encoding</b>
        - Tells the web server what types of compression methods the browser supports so the data can be made smaller for transmitting over the internet.
    - <b>Cookie</b> 
        - Data sent to the server to help remember your information
- <b>Headers related to the response</b>: these are headers that are sent by the server to the client in response to a request
    - <b>Set-Cookie</b>
        - Information to store which gets sent back to the web server on each request 
    - <b>Cache-Control</b>
        - How long to store the content of the response in the browser's cache before it requests it again.
    - <b>Content-Type</b>
        - This tells the client what type of data is being returned, i.e., HTML, CSS, JavaScript, Images, PDF, Video, etc. Using the content-type header the browser then knows how to process the data.
    - <b>Content-Encoding</b>
        - What method has been used to compress the data to make it smaller when sending it over the internet.





<br />

## Cookies
<br />

 Cookies are just a small piece of data that is stored on your computer. Cookies are saved when you receive a `Set-Cookie` header from a web server. Then every further request you make, you'll send the cookie data back to the web server. Because <b>HTTP is stateless</b> (doesn't keep track of your previous requests), cookies can be used to remind the web server who you are, some personal settings for the website or whether you've been to the website before.

<br />

![Cookie Flow](/assets/cookie_flow.png)

<br />

 There are three types of computer cookies: session, persistent, and third-party.  These virtually invisible text files are all very different.  Each with their own mission, these cookies are made to track, collect, and store any data that companies request.

 ### Session Cookie
 Session cookies are temporary cookies that memorize your online activities.  Since websites have no sense of memory, without these cookies, your site browsing history would always be blank.  In fact, with every click you would make, the website would treat you as a completely new visitor.

A good example of how session cookies are helpful is online shopping.  When you’re shopping online, you can check-out at any time.  That’s because session cookies track your movement.  Without these cookies, whenever you would go to check-out, your cart would be empty.

Ultimately, session cookies help you maneuver through the internet by remembering your actions, and they expire as soon as you close out of a web page.

### Persistent Cookie
Persistent cookies (also known as first-party cookies) work by tracking your online preferences.  When you visit a website for the first time, it is at its default setting.  But if you personalize the site to fit your preferences, persistent cookies will remember and implement those preferences the next time you visit the site.  This is how computers remember and store your login information, language selections, menu preferences, internal bookmarks, and more.

Persistent, permanent, and stored cookies are terms used interchangeably as these cookies are stored in your hard disk for (typically) a long period of time.  The cookie’s timeline will vary depending on the expiration date.  But, once that date is reached, the cookie will be deleted, along with everything you customized.  Luckily, websites prefer to employ a long-life span so that users can make the most of their personal preferences.

### Third-Part Cookie
Third-party cookies, also referred to as tracking cookies, collect data based on your online behavior.  When you visit a website, third-party cookies collect various types of data that are then passed on or sold to advertisers by the website that created the cookie.  Tracking your interests, location, age, and search trends, these cookies collect information so that marketers can provide you with custom advertisements.  These are the ads that appear on websites you visit and display content relevant to your interests.

By tracking your habits and providing targeted ads, third-party cookies serve a useful purpose for marketers but can seem pesky and intrusive to internet users.  That’s why you have the option to block them.

<br />

<b>Pay attention</b>: In recent years, the use of cookies has been standardized and regulated. Each user must be informed about the use of web cookies and can change the settings at their pleasure. <br />
You can find more here: [GDPR](https://gdpr-info.eu/)


