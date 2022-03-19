---
title: "DNSSEC"
date: 2022-03-19T08:19:22Z
draft: false
---

Cloudflare defines DNSSEC as:

> DNSSEC protects against forged DNS answers. DNSSEC protected zones are cryptographically signed to ensure the DNS records received
> are identical to the DNS records published by the domain owner.
>
> We will automatically enable DNSSEC for your domain in the next 24 hours once you click 'Enable DNSSEC'.

This small guide covers the [Cloudflare DNSSEC setup article][1], so you can ignore this and head on over to the experts to get your
own DNSSEC set up. To enable DNSSEC on Cloudflare, simply head to your DNS config and click the enable DNSSEC button. This enables
DNSSEC for you and adds a DNSSEC zone to your record set.

So what the hell does DNSSEC actually do? According to [ICANN][2]:

> DNS was designed in the 1980s when the Internet was much smaller, and security was not a primary consideration in its design. As a result,
> when a recursive resolver sends a query to an authoritative name server, the resolver has no way to verify the authenticity of the response.
> The resolver can only check that a response appears to come from the same IP address where the resolver sent the original query. But
> relying on the source IP address of a response is not a strong authentication mechanism, since the source IP address of a DNS response
> packet can be easily forged, or spoofed. As DNS was originally designed, a resolver cannot easily detect a forged response to one of its
> queries. An attacker can easily masquerade as the authoritative server that a resolver originally queried by spoofing a response that
> appears to come from that authoritative server. In other words an attacker can redirect a user to a potentially malicious site without
> the user realizing it.
>
> DNSSEC strengthens authentication in DNS using digital signatures based on public key cryptography. With DNSSEC, it's not DNS queries
> and responses themselves that are cryptographically signed, but rather DNS data itself is signed by the owner of the data.
>
> Every DNS zone has a public/private key pair. The zone owner uses the zone's private key to sign DNS data in the zone and generate
> digital signatures over that data. As the name "private key" implies, this key material is kept secret by the zone owner.
> The zone's public key, however, is published in the zone itself for anyone to retrieve. Any recursive resolver that looks up data
> in the zone also retrieves the zone's public key, which it uses to validate the authenticity of the DNS data. The resolver confirms
> that the digital signature over the DNS data it retrieved is valid. If so, the DNS data is legitimate and is returned to the user.
> If the signature does not validate, the resolver assumes an attack, discards the data, and returns an error to the user.
> 
> DNSSEC adds two important features to the DNS protocol:
>
> Data origin authentication allows a resolver to cryptographically verify that the data it received actually came from the zone where
> it believes the data originated. Data integrity protection allows the resolver to know that the data hasn't been modified in transit
> since it was originally signed by the zone owner with the zone's private key.

So DNSSEC just creates a DS record in your root domain and offers that as a resource to individuals querying your domain. Using an example
from Cloudflare themselves:

```
$ dig www.cloudflare.com +dnssec +short
104.16.124.96
104.16.123.96
A 13 3 300 20220320094822 20220318074822 34505 www.cloudflare.com. 3M+bn0HmaWkpx1iWcxJd3rYDdrk9S0la+nkplA1l3Ol2YIeVxoBm+bJz 7hw3x42lvqFr9/oM36cH0Z3MI1zdzw==
```

The RRSIG record above indicates that the domain uses DNSSEC and that records should be verified. Looking at the DNSKEY options:

```
$ dig DNSKEY www.cloudflare.com +short
257 3 13 mdsswUyr3DPW132mOi8V9xESWE8jTo0dxCjjnopKl+GqJxpVXckHAeF+ KkxLbxILfDLUT0rAK9iUzy1L53eKGQ==
256 3 13 oJMRESz5E4gYzS/q6XDrvU1qMPYIjCWzJaOau8XNEZeqCYKD5ar0IRd8 KqXXFJkqmVfRvMGPmM1x8fGAa2XhSA==
```

Record 256 is the Zone-Signing Key and is used to verify records for A, MX, CNAME, SRV and so on. Record 257 is called the Key Signing Key
and verifies DNSKEY, CDS, and CDNSKEY records records.

There's not a massive amount of troubleshooting that you can do with DNSSEC. It creates a DS record for you (eventually!) and sets up some
additional records in your domain. You can query these records freely. I think I for one will be back when DNSSEC is enabled on my own domain
and I can play around with it a little bit more.

[1]: https://developers.cloudflare.com/dns/additional-options/dnssec/
[2]: https://www.icann.org/resources/pages/dnssec-what-is-it-why-important-2019-03-05-en