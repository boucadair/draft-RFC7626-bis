%%%
    Title = "DNS Privacy Considerations"
    abbrev = "DNS Privacy Considerations"
    category = "std"
    docName= "draft-bortzmeyer-RFC7626-bis-00"
    ipr = "trust200902"
    area = "Internet Area"
    workgroup = "dprive"
    keyword = ["DNS"]
    date = 2018-06-21T00:00:00Z
    [pi]
    toc = "yes"
    compact = "yes"
    symrefs = "yes"
    sortrefs = "yes"
    subcompact = "no"
    [[author]]
    initials="S."
    surname="Bortzmeyer"
    fullname="Stephane Bortzmeyer"
    organization = "AFNIC"
      [author.address]
      email = "bortzmeyer+ietf@nic.fr"
      [author.address.postal]
      streets = ["1, rue Stephenson", "Montigny-le-Bretonneux"]
      city = "France"
      code = "78180"
    [[author]]
    initials="S."
    surname="Dickinson"
    fullname="Sara Dickinson"
    organization = "Sinodun IT"
        [author.address]
        email = "sara@sinodun.com"
        [author.address.postal]
        streets = ["Magdalen Centre", "Oxford Science Park"]
        city = "Oxford"
        code = "OX4 4GA"
        country = 'United Kingdom'
%%%


.# Abstract

   This document describes the privacy issues associated with the use of
   the DNS by Internet users.  It is intended to be an analysis of the
   present situation and does not prescribe solutions.
   
{mainmatter}

#  Introduction

   This document is an analysis of the DNS privacy issues, in the spirit
   of Section 8 of [RFC6973].

   The Domain Name System is specified in [RFC1034], [RFC1035], and many
   later RFCs, which have never been consolidated.  It is one of the
   most important infrastructure components of the Internet and often
   ignored or misunderstood by Internet users (and even by many
   professionals).  Almost every activity on the Internet starts with a
   DNS query (and often several).  Its use has many privacy implications
   and this is an attempt at a comprehensive and accurate list.

   Let us begin with a simplified reminder of how the DNS works.  (See
   also [DNS-TERMS].)  A client, the stub resolver, issues a DNS query
   to a server, called the recursive resolver (also called caching
   resolver or full resolver or recursive name server).  Let's use the
   query "What are the AAAA records for www.example.com?" as an example.
   AAAA is the QTYPE (Query Type), and www.example.com is the QNAME
   (Query Name).  (The description that follows assumes a cold cache,
   for instance, because the server just started.)  The recursive
   resolver will first query the root name servers.  In most cases, the
   root name servers will send a referral.  In this example, the
   referral will be to the .com name servers.  The resolver repeats the
   query to one of the .com name servers.  The .com name servers, in
   turn, will refer to the example.com name servers.  The example.com
   name server will then return the answer.  The root name servers, the
   name servers of .com, and the name servers of example.com are called
   authoritative name servers.  It is important, when analyzing the
   privacy issues, to remember that the question asked to all these name
   servers is always the original question, not a derived question.  The
   question sent to the root name servers is "What are the AAAA records
   for www.example.com?", not "What are the name servers of .com?".  By
   repeating the full question, instead of just the relevant part of the
   question to the next in line, the DNS provides more information than
   necessary to the name server.

   Because DNS relies on caching heavily, the algorithm described just
   above is actually a bit more complicated, and not all questions are
   sent to the authoritative name servers.  If a few seconds later the
   stub resolver asks the recursive resolver, "What are the SRV records
   of _xmpp-server._tcp.example.com?", the recursive resolver will
   remember that it knows the name servers of example.com and will just
   query them, bypassing the root and .com.  Because there is typically
   no caching in the stub resolver, the recursive resolver, unlike the
   authoritative servers, sees all the DNS traffic.  (Applications, like
   web browsers, may have some form of caching that does not follow DNS
   rules, for instance, because it may ignore the TTL.  So, the
   recursive resolver does not see all the name resolution activity.)

   It should be noted that DNS recursive resolvers sometimes forward
   requests to other recursive resolvers, typically bigger machines,
   with a larger and more shared cache (and the query hierarchy can be
   even deeper, with more than two levels of recursive resolvers).  From
   the point of view of privacy, these forwarders are like resolvers,
   except that they do not see all of the requests being made (due to
   caching in the first resolver).

   Almost all this DNS traffic is currently sent in clear (unencrypted). At the
   time of writing there is increasing deployment of DNS-over-TLS {{RFC7858}}
   and work underway on DoH {{}}. There are a few cases where there is some
   alternative channel encryption, for instance, in an IPsec VPN, at least
   between the stub resolver and the resolver.

   Today, almost all DNS queries are sent over UDP [thomas-ditl-tcp].
   This has practical consequences when considering encryption of the
   traffic as a possible privacy technique.  Some encryption solutions
   are only designed for TCP, not UDP.

   Another important point to keep in mind when analyzing the privacy
   issues of DNS is the fact that DNS requests received by a server are
   triggered by different reasons.  Let's assume an eavesdropper wants
   to know which web page is viewed by a user.  For a typical web page,
   there are three sorts of DNS requests being issued:

  Primary request: this is the domain name in the URL that the user
  typed, selected from a bookmark, or chose by clicking on an
  hyperlink.  Presumably, this is what is of interest for the
  eavesdropper.

  Secondary requests: these are the additional requests performed by
  the user agent (here, the web browser) without any direct
  involvement or knowledge of the user.  For the Web, they are
  triggered by embedded content, Cascading Style Sheets (CSS),
  JavaScript code, embedded images, etc.  In some cases, there can
  be dozens of domain names in different contexts on a single web
  page.

  Tertiary requests: these are the additional requests performed by
  the DNS system itself.  For instance, if the answer to a query is
  a referral to a set of name servers, and the glue records are not
  returned, the resolver will have to do additional requests to turn
  the name servers' names into IP addresses.  Similarly, even if
  glue records are returned, a careful recursive server will do
  tertiary requests to verify the IP addresses of those records.

   It can be noted also that, in the case of a typical web browser, more
   DNS requests than strictly necessary are sent, for instance, to
   prefetch resources that the user may query later or when
   autocompleting the URL in the address bar.  Both are a big privacy
   concern since they may leak information even about non-explicit
   actions.  For instance, just reading a local HTML page, even without
   selecting the hyperlinks, may trigger DNS requests.

   For privacy-related terms, we will use the terminology from
   [RFC6973].

#   Risks

   This document focuses mostly on the study of privacy risks for the
   end user (the one performing DNS requests).  We consider the risks of
   pervasive surveillance [RFC7258] as well as risks coming from a more
   focused surveillance.  Privacy risks for the holder of a zone (the
   risk that someone gets the data) are discussed in [RFC5936] and
   [RFC5155].  Non-privacy risks (such as cache poisoning) are out of
   scope.

##  The Alleged Public Nature of DNS Data

   It has long been claimed that "the data in the DNS is public".  While
   this sentence makes sense for an Internet-wide lookup system, there
   are multiple facets to the data and metadata involved that deserve a
   more detailed look.  First, access control lists and private
   namespaces notwithstanding, the DNS operates under the assumption
   that public-facing authoritative name servers will respond to "usual"
   DNS queries for any zone they are authoritative for without further
   authentication or authorization of the client (resolver).  Due to the
   lack of search capabilities, only a given QNAME will reveal the
   resource records associated with that name (or that name's non-
   existence).  In other words: one needs to know what to ask for, in
   order to receive a response.  The zone transfer QTYPE [RFC5936] is
   often blocked or restricted to authenticated/authorized access to
   enforce this difference (and maybe for other reasons).

   Another differentiation to be considered is between the DNS data
   itself and a particular transaction (i.e., a DNS name lookup).  DNS
   data and the results of a DNS query are public, within the boundaries
   described above, and may not have any confidentiality requirements.
   However, the same is not true of a single transaction or a sequence
   of transactions; that transaction is not / should not be public.  A
   typical example from outside the DNS world is: the web site of
   Alcoholics Anonymous is public; the fact that you visit it should not
   be.

##  Data in the DNS Request

   The DNS request includes many fields, but two of them seem
   particularly relevant for the privacy issues: the QNAME and the
   source IP address. "source IP address" is used in a loose sense of
   "source IP address + maybe source port", because the port is also in
   the request and can be used to differentiate between several users
   sharing an IP address (behind a Carrier-Grade NAT (CGN), for instance
   [RFC6269]).

   The QNAME is the full name sent by the user.  It gives information
   about what the user does ("What are the MX records of example.net?"
   means he probably wants to send email to someone at example.net,
   which may be a domain used by only a few persons and is therefore
   very revealing about communication relationships).  Some QNAMEs are
   more sensitive than others.  For instance, querying the A record of a
   well-known web statistics domain reveals very little (everybody
   visits web sites that use this analytics service), but querying the A
   record of www.verybad.example where verybad.example is the domain of
   an organization that some people find offensive or objectionable may
   create more problems for the user.  Also, sometimes, the QNAME embeds
   the software one uses, which could be a privacy issue.  For instance,
   _ldap._tcp.Default-First-Site-Name._sites.gc._msdcs.example.org.
   There are also some BitTorrent clients that query an SRV record for
   _bittorrent-tracker._tcp.domain.example.

   Another important thing about the privacy of the QNAME is the future
   usages.  Today, the lack of privacy is an obstacle to putting
   potentially sensitive or personally identifiable data in the DNS.  At
   the moment, your DNS traffic might reveal that you are doing email
   but not with whom.  If your Mail User Agent (MUA) starts looking up
   Pretty Good Privacy (PGP) keys in the DNS [DANE-OPENPGPKEY], then
   privacy becomes a lot more important.  And email is just an example;
   there would be other really interesting uses for a more privacy-
   friendly DNS.

   For the communication between the stub resolver and the recursive resolver,
  the source IP address is the address of the user's machine. Therefore, all the
  issues and warnings about collection of IP addresses apply here. For the
  communication between the recursive resolver and the authoritative name
  servers, the source IP address has a different meaning; it does not have the
  same status as the source address in an HTTP connection. It is now the IP
  address of the recursive resolver that, in a way, "hides" the real user.
  However, hiding does not always work. Sometimes EDNS(0) Client subnet
  [RFC7871] is used (see its privacy analysis in [denis-edns-client-subnet]).
  Sometimes the end user has a personal recursive resolver on her machine. In
  both cases, the IP address is as sensitive as it is for HTTP [sidn-entrada].

   A note about IP addresses: there is currently no IETF document that
   describes in detail all the privacy issues around IP addressing.  In
   the meantime, the discussion here is intended to include both IPv4
   and IPv6 source addresses.  For a number of reasons, their assignment
   and utilization characteristics are different, which may have
   implications for details of information leakage associated with the
   collection of source addresses.  (For example, a specific IPv6 source
   address seen on the public Internet is less likely than an IPv4
   address to originate behind a CGN or other NAT.)  However, for both
   IPv4 and IPv6 addresses, it's important to note that source addresses
   are propagated with queries and comprise metadata about the host,
   user, or application that originated them.

### Data in the DNS payload

At the time of writing there are no standardized client identifiers contained in
the DNS payload itself (ECS [RFC7871] while widely used is only of Category
Informational). 

DNS Cookies [RFC7873] are a lightweight DNS transaction security mechanism that
provides limited protection against a variety of increasingly common
denial-of-service and amplification/forgery or cache poisoning attacks by
off-path attackers. It is noted, however, that they are designed to just verify
IP addresses (and should change once a client's IP address changes), they are
not designed to actively track users (like HTTP cookies).

There are anecdotal accounts of MAC addresses and even user ids
being inserted in non-standard EDNS(0) options for stub to resolver
communications to support proprietary functionality implemented at the resolver
(e.g. parental filtering).

##  Cache Snooping

   The content of recursive resolvers' caches can reveal data about the
   clients using it (the privacy risks depend on the number of clients).
   This information can sometimes be examined by sending DNS queries
   with RD=0 to inspect cache content, particularly looking at the DNS
   TTLs [grangeia.snooping].  Since this also is a reconnaissance
   technique for subsequent cache poisoning attacks, some counter
   measures have already been developed and deployed.

##  On the Wire

### Unencrypted Transports

   For unencrypted transports, DNS traffic can be seen by an eavesdropper like
   any other traffic. (DNSSEC, specified in [RFC4033], explicitly excludes
   confidentiality from its goals.) So, if an initiator starts an HTTPS
   communication with a recipient, while the HTTP traffic will be encrypted, the
   DNS exchange prior to it will not be. When other protocols will become more
   and more privacy-aware and secured against surveillance (e.g. [TLS13, QUIC]),
   the use of unencrypted transports for DNS may become "the weakest link" in
   privacy. It is noted that there is on-going work attempting to encrypt the
   SNI in the TLS handshake but that this is a non-trivial problem [].

   An important specificity of the DNS traffic is that it may take a
   different path than the communication between the initiator and the
   recipient.  For instance, an eavesdropper may be unable to tap the
   wire between the initiator and the recipient but may have access to
   the wire going to the recursive resolver, or to the authoritative
   name servers.

   The best place to tap, from an eavesdropper's point of view, is
   clearly between the stub resolvers and the recursive resolvers,
   because traffic is not limited by DNS caching.

   The attack surface between the stub resolver and the rest of the
   world can vary widely depending upon how the end user's computer is
   configured.  By order of increasing attack surface:

      The recursive resolver can be on the end user's computer.  In
      (currently) a small number of cases, individuals may choose to
      operate their own DNS resolver on their local machine.  In this
      case, the attack surface for the connection between the stub
      resolver and the caching resolver is limited to that single
      machine.

      The recursive resolver may be at the local network edge.  For
      many/most enterprise networks and for some residential users, the
      caching resolver may exist on a server at the edge of the local
      network.  In this case, the attack surface is the local network.
      Note that in large enterprise networks, the DNS resolver may not
      be located at the edge of the local network but rather at the edge
      of the overall enterprise network.  In this case, the enterprise
      network could be thought of as similar to the Internet Access
      Provider (IAP) network referenced below.

      The recursive resolver can be in the IAP premises.  For most
      residential users and potentially other networks, the typical case
      is for the end user's computer to be configured (typically
      automatically through DHCP) with the addresses of the DNS
      recursive resolvers at the IAP.  The attack surface for on-the-
      wire attacks is therefore from the end-user system across the
      local network and across the IAP network to the IAP's recursive
      resolvers.

      The recursive resolver can be a public DNS service.  Some machines
      may be configured to use public DNS resolvers such as those
      operated today by Google Public DNS or OpenDNS.  The end user may
      have configured their machine to use these DNS recursive resolvers
      themselves -- or their IAP may have chosen to use the public DNS
      resolvers rather than operating their own resolvers.  In this
      case, the attack surface is the entire public Internet between the
      end user's connection and the public DNS service.

### Encrypted Transports

The use of encrypted transports directly mitigates passive surveillance of the
DNS payload, however there are still some privacy attacks possible.

These are cases where user identification, fingerprinting or correlations may be
possible due to the use of certain transport layers or clear text features.
These issues are not specific to DNS, but DNS traffic is susceptible to these
attacks when using specific transports.

There are some general examples, for example, certain studies have highlighted
that IP TTL/Hoplimit values can be used to fingerprint client OS's.

The use of clear text transport options to decrease latency may also identify a
user e.g. using TCP Fast Open [].

More specifically, (since the deployment of encrypted transports is not
widespread at the time of writing) users wishing to use encrypted transports for
DNS may in practice be limited in the resolver services available. Given this,
the choice of a user to configure a single resolver (or a fixed set of
resolvers) and an encrypted transport to use in all network environments can
actually serve to identify the user as one that desires privacy and can provide
an added mechanism to track them as they move across network environments.

Users of encrypted transports are also highly likely to re-use sessions for
multiple DNS queries to optimise performance (e.g. via DNS pipelining or HTTPS
multiplexing ). Certain configuration options for encrypted transports could
also in principle fingerprint a user, for example session resumption, the
maximum number of messages to send or a maximum total connection time before
closing a connections and re-opening.

Whilst there are known attacks on older versions of TLS the most recent
recommendations [RFC7525] and developments [TLS1.3] in this area largely
mitigate those.

Traffic analysis of unpadded encrypted traffic is also possible [Pitfalls of DNS
Encryption](https://www.ietf.org/mail-archive/web/dns-privacy/current/pdfWqAIUmEl47.pdf)
because the sizes and timing of encrypted DNS requests and responses can be
correlated to unencrypted DNS requests upstream of a recursive resolver.


##  In the Servers

   Using the terminology of [RFC6973], the DNS servers (recursive
   resolvers and authoritative servers) are enablers: they facilitate
   communication between an initiator and a recipient without being
   directly in the communications path.  As a result, they are often
   forgotten in risk analysis.  But, to quote again [RFC6973], "Although
   [...] enablers may not generally be considered as attackers, they may
   all pose privacy threats (depending on the context) because they are
   able to observe, collect, process, and transfer privacy-relevant
   data."  In [RFC6973] parlance, enablers become observers when they
   start collecting data.

   Many programs exist to collect and analyze DNS data at the servers --
   from the "query log" of some programs like BIND to tcpdump and more
   sophisticated programs like PacketQ [packetq] [packetq-list] and
   DNSmezzo [dnsmezzo].  In the case of DoH servers the server sees not 
   only the DNS traffic but also any headers attached to the traffic. 
   
   The organization managing the DNS server can
   use this data itself, or it can be part of a surveillance program
   like PRISM [prism] and pass data to an outside observer.

   Sometimes, this data is kept for a long time and/or distributed to
   third parties for research purposes [ditl] [day-at-root], security
   analysis, or surveillance tasks.  These uses are sometimes under some
   sort of contract, with various limitations, for instance, on
   redistribution, given the sensitive nature of the data.  Also, there
   are observation points in the network that gather DNS data and then
   make it accessible to third parties for research or security purposes
   ("passive DNS" [passive-dns]).

###  In the Recursive Resolvers

   Recursive Resolvers see all the traffic since there is typically no
   caching before them.  To summarize: your recursive resolver knows a
   lot about you.  The resolver of a large IAP, or a large public
   resolver, can collect data from many users.  You may get an idea of
   the data collected by reading the privacy policy of a big public
   resolver, e.g., https://developers.google.com/speed/public-dns/
   privacy.

#### Encrypted transports

Use of encrypted transports does not reduce the data available in the recursive
resolver and ironically can actually expose more information about users to
operators. As mentioned in (#on-the-wire) use of session based encrypted
transports (TCP/TLS) can add correlation data about users. Such concerns apply
equally to DNS-over-TLS and DoH which both use TLS as the underlying transport.

#### DoH vs DNS-over-TLS

The proposed specification for DoH [] includes a Privacy Considerations section
which highlights that HTTP and DNS are very different protocols. As a
deliberate design choice DoH inherits the privacy properties of the HTTPS stack
and as a consequence introduces new privacy concerns when compared with DNS over
UDP, TCP or TLS (RFC7858). The rationale for this decision is that retaining the
ability to leverage the full functionality of the HTTP ecosystem is more
important than placing any constraints on this new protocol based on privacy
considerations.

As a result there exists a natural tension between

* the wide practice in HTTP to use various headers to optimise HTTP
connections, functionality and behaviour (which can facilitate user
identification and tracking) 

* and the fact that currently the DNS payload is very tightly
encoded and contains no standardized user identifiers.

DNS-over-TLS, for example, would normally contain no client identifiers in the
DNS messages and a resolver would see only a stream of DNS queries originating
from a client IP address. Whereas if DoH clients commonly include several
headers in a DNS message (e.g. user-agent and accept-language) this could lead
to the DoH server being able to identify the source of individual DNS requests
not only to a specific end user device but to a specific application.

Additionally, depending on the client architecture, isolation of DoH queries
from other HTTP traffic may or may not be feasible or desirable. Depending on
the use case, isolation of DoH queries from other HTTP traffic may or may not
increase privacy.

The picture for privacy considerations and user expectations for DoH with
respect to what additional data may be available to the resolver compared to DNS
over UDP,TCP or TLS is complex and requires a detailed analysis for each
particular use case. In particular the choice of HTTPS functionality vs privacy
is specifically made an implementation choice and uses may well have
differencing privacy expectations depending on the DoH use case implementation.

At the extremes, there may be implementations that attempt to achieve parity
with DNS-over-TLS from a privacy perspective at the cost of using no
identifiable headers, there might be others that provide feature rich data flows
where the low-level origin of the DNS query is easily identifiable.

Users who have concerns about such additional data may choose not to use DoH and
to use DNS-over-TLS instead.

###  In the Authoritative Name Servers

   Unlike what happens for recursive resolvers, observation capabilities
   of authoritative name servers are limited by caching; they see only
   the requests for which the answer was not in the cache.  For
   aggregated statistics ("What is the percentage of LOC queries?"),
   this is sufficient, but it prevents an observer from seeing
   everything.  Still, the authoritative name servers see a part of the
   traffic, and this subset may be sufficient to violate some privacy
   expectations.

   Also, the end user typically has some legal/contractual link with the
   recursive resolver (he has chosen the IAP, or he has chosen to use a
   given public resolver), while having no control and perhaps no
   awareness of the role of the authoritative name servers and their
   observation abilities.

   As noted before, using a local resolver or a resolver close to the
   machine decreases the attack surface for an on-the-wire eavesdropper.
   But it may decrease privacy against an observer located on an
   authoritative name server.  This authoritative name server will see
   the IP address of the end client instead of the address of a big
   recursive resolver shared by many users.

   This "protection", when using a large resolver with many clients, is
   no longer present if [CLIENT-SUBNET] is used because, in this case,
   the authoritative name server sees the original IP address (or
   prefix, depending on the setup).

   As of today, all the instances of one root name server, L-root,
   receive together around 50,000 queries per second.  While most of it
   is "junk" (errors on the Top-Level Domain (TLD) name), it gives an
   idea of the amount of big data that pours into name servers.  (And
   even "junk" can leak information; for instance, if there is a typing
   error in the TLD, the user will send data to a TLD that is not the
   usual one.)

   Many domains, including TLDs, are partially hosted by third-party
   servers, sometimes in a different country.  The contracts between the
   domain manager and these servers may or may not take privacy into
   account.  Whatever the contract, the third-party hoster may be honest
   or not but, in any case, it will have to follow its local laws.  So,
   requests to a given ccTLD may go to servers managed by organizations
   outside of the ccTLD's country.  End users may not anticipate that,
   when doing a security analysis.

   Also, it seems (see the survey described in [aeris-dns]) that there
   is a strong concentration of authoritative name servers among
   "popular" domains (such as the Alexa Top N list).  For instance,
   among the Alexa Top 100K, one DNS provider hosts today 10% of the
   domains.  The ten most important DNS providers host together one
   third of the domains.  With the control (or the ability to sniff the
   traffic) of a few name servers, you can gather a lot of information.

###  Rogue Servers

   The previous paragraphs discussed DNS privacy, assuming that all the
   traffic was directed to the intended servers and that the potential
   attacker was purely passive.  But, in reality, we can have active
   attackers redirecting the traffic, not to change it but just to
   observe it.

   For instance, a rogue DHCP server, or a trusted DHCP server that has
   had its configuration altered by malicious parties, can direct you to
   a rogue recursive resolver.  Most of the time, it seems to be done to
   divert traffic by providing lies for some domain names.  But it could
   be used just to capture the traffic and gather information about you.
   Other attacks, besides using DHCP, are possible.  The traffic from a
   DNS client to a DNS server can be intercepted along its way from
   originator to intended source, for instance, by transparent DNS
   proxies in the network that will divert the traffic intended for a
   legitimate DNS server.  This rogue server can masquerade as the
   intended server and respond with data to the client.  (Rogue servers
   that inject malicious data are possible, but it is a separate problem
   not relevant to privacy.)  A rogue server may respond correctly for a
   long period of time, thereby foregoing detection.  This may be done
   for what could be claimed to be good reasons, such as optimization or
   caching, but it leads to a reduction of privacy compared to if there
   was no attacker present.  Also, malware like DNSchanger [dnschanger]
   can change the recursive resolver in the machine's configuration, or
   the routing itself can be subverted (for instance,
   [ripe-atlas-turkey]).

### Authentication of servers

Both Strict mode for DNS-over-TLS and DoH require authentication of the server
and therefore as long as the authentication credentials are obtained over a
secure channel then using either of these transports defeats the attack of
re-directing traffic to rogue servers. Of course attacks on these secure
channels are also possible, but out of the scope of this document.

### Blocking of services

   TODO: Active blocking of port 853, blocking or attacking recursive resolvers offering encrypted transports,

##  Re-identification and Other Inferences

   An observer has access not only to the data he/she directly collects
   but also to the results of various inferences about this data.

   For instance, a user can be re-identified via DNS queries.  If the
   adversary knows a user's identity and can watch their DNS queries for
   a period, then that same adversary may be able to re-identify the
   user solely based on their pattern of DNS queries later on regardless
   of the location from which the user makes those queries.  For
   example, one study [herrmann-reidentification] found that such re-
   identification is possible so that "73.1% of all day-to-day links
   were correctly established, i.e. user u was either re-identified
   unambiguously (1) or the classifier correctly reported that u was not
   present on day t+1 any more (2)."  While that study related to web
   browsing behavior, equally characteristic patterns may be produced
   even in machine-to-machine communications or without a user taking
   specific actions, e.g., at reboot time if a characteristic set of
   services are accessed by the device.

   For instance, one could imagine that an intelligence agency
   identifies people going to a site by putting in a very long DNS name
   and looking for queries of a specific length.  Such traffic analysis
   could weaken some privacy solutions.

   The IAB privacy and security program also have a work in progress
   [RFC7624] that considers such inference-based attacks in a more
   general framework.

##  More Information

   Useful background information can also be found in [tor-leak] (about
   the risk of privacy leak through DNS) and in a few academic papers:
   [yanbin-tsudik], [castillo-garcia], [fangming-hori-sakurai], and
   [federrath-fuchs-herrmann-piosecny].

#  Actual "Attacks"

   A very quick examination of DNS traffic may lead to the false
   conclusion that extracting the needle from the haystack is difficult.
   "Interesting" primary DNS requests are mixed with useless (for the
   eavesdropper) secondary and tertiary requests (see the terminology in
   Section 1).  But, in this time of "big data" processing, powerful
   techniques now exist to get from the raw data to what the
   eavesdropper is actually interested in.

   Many research papers about malware detection use DNS traffic to
   detect "abnormal" behavior that can be traced back to the activity of
   malware on infected machines.  Yes, this research was done for the
   good, but technically it is a privacy attack and it demonstrates the
   power of the observation of DNS traffic.  See [dns-footprint],
   [dagon-malware], and [darkreading-dns].

   Passive DNS systems [passive-dns] allow reconstruction of the data of
   sometimes an entire zone.  They are used for many reasons -- some
   good, some bad.  Well-known passive DNS systems keep only the DNS
   responses, and not the source IP address of the client, precisely for
   privacy reasons.  Other passive DNS systems may not be so careful.
   And there is still the potential problems with revealing QNAMEs.

   The revelations (from the Edward Snowden documents, which were leaked
   from the National Security Agency (NSA)) of the MORECOWBELL
   surveillance program [morecowbell], which uses the DNS, both
   passively and actively, to surreptitiously gather information about
   the users, is another good example showing that the lack of privacy
   protections in the DNS is actively exploited.


#  Legalities

   To our knowledge, there are no specific privacy laws for DNS data, in
   any country.  Interpreting general privacy laws like
   [data-protection-directive] or [GDPR] (European Union) in the context of DNS
   traffic data is not an easy task, and we do not know a court
   precedent here.  See an interesting analysis in [sidn-entrada].

#  Security Considerations

   This document is entirely about security, more precisely privacy. It just
   lays out the problem; it does not try to set requirements (with the choices
   and compromises they imply), much less define solutions. Possible solutions
   to the issues described here are discussed in other documents (currently too
   many to all be mentioned); see, for instance, 'Recommendations for DNS
   Privacy Operators' [I-D.draft-dickinson-dprive-bcp-op].



# Acknowledgments

   Thanks to Nathalie Boulvard and to the CENTR members for the original
   work that led to this document.  Thanks to Ondrej Sury for the
   interesting discussions.  Thanks to Mohsen Souissi and John Heidemann
   for proofreading and to Paul Hoffman, Matthijs Mekking, Marcos Sanz,
   Tim Wicinski, Francis Dupont, Allison Mankin, and Warren Kumari for
   proofreading, providing technical remarks, and making many
   readability improvements.  Thanks to Dan York, Suzanne Woolf, Tony
   Finch, Stephen Farrell, Peter Koch, Simon Josefsson, and Frank Denis
   for good written contributions.  And thanks to the IESG members for
   the last remarks.

{backmatter}