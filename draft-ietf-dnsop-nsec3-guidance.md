---
title: "Guidance for NSEC3 parameter settings"
abbrev: title
docname: draft-ietf-dnsop-nsec3-guidance-01
category: bcp
ipr: trust200902

stand_alone: yes
pi: [toc, sortrefs, symrefs, docmapping]

author:
  -
    ins: W. Hardaker
    name: Wes Hardaker
    org: USC/ISI
    email: ietf@hardakers.net
  -
    ins: V. Dukhovni
    name: Viktor Dukhovni
    org: Bloomberg, L.P.
    email: ietf-dane@dukhovni.org

normative:
  RFC2119:
  RFC5155:
  RFC4035:
  RFC4470:

informative:
  GPUNSEC3:
    title: GPU-Based NSEC3 Hash Breaking
    author:
      -
        ins: M. Wander
        name: M. Wander
      -
        ins: L. Schwittmann
        name: L. Schwittmann
      -
        ins: C. Boelmann
        name: C. Boelmann
      -
        ins: T. Weis
        name: T. Weis
    date: 2014
    seriesinfo:
      DOI: 10.1109/NCA.2014.27
  ZONEENUM:
    title: An efficient DNSSEC zone enumeration algorithm
    author:
      - 
        ins: Z. Wang
        name: Zheng Wang
      - 
        ins: L. Xiao
        name: Liyuan Xiao
      - 
        ins: R. Wang
        name: Rui Wang


--- abstract

NSEC3 is a DNSSEC mechanism providing proof of non-existence by
promising there are no names that exist between two domainnames within
a zone.  Unlike its counterpart NSEC, NSEC3 avoids directly disclosing
the bounding domainname pairs.  This document provides guidance on
setting NSEC3 parameters based on recent operational deployment
experience.

--- middle

# Introduction

As with NSEC {{RFC4035}}, NSEC3 {{RFC5155}} provides proof of
non-existence that consists of signed DNS records establishing the
non-existence of a given name or associated Resource Record Type
(RRTYPE) in a DNSSEC {{RFC4035}} signed zone.  In the case of NSEC3,
however, the names of valid nodes in the zone are obfuscated through
(possibly multiple iterations of) hashing via SHA-1. (currently only
SHA-1 is in use within the Internet).

NSEC3 also provides "opt-out support", allowing for blocks of unsigned
delegations to be covered by a single NSEC3 record.  Use of the
opt-out feature allow large registries to only sign as many NSEC3
records as there are signed DS or other RRsets in the zone -- with
opt-out, unsigned delegations don't require additional NSEC3 records.
This sacrifices the tamper-resistance proof of non-existence offered
by NSEC3 in order to reduce memory and CPU overheads.

NSEC3 records have a number of tunable parameters that are specified
via an NSEC3PARAM record at the zone apex.  These parameters are the
Hash Algorithm, processing Flags, the number of hash Iterations and
the Salt.  Each of these has security and operational considerations
that impact both zone owners and validating resolvers.  This document
provides some best-practice recommendations for setting the NSEC3
parameters.

## Requirements notation

   The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
   "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY",
   and "OPTIONAL" in this document are to be interpreted as described
   in BCP 14 {{RFC2119}} {{?RFC8174}} when, and only when, they appear
   in all capitals, as shown here.

# Recommendation for zone publishers

The following sections describe recommendations for setting parameters
for NSEC3 and NSEC3PARAM.

## Algorithms

The algorithm field is not discussed by this document.

## Flags

The NSEC3PARAM flags field currently contains no flags, but individual
NSEC3 records contain the "Opt-Out" flag {{RFC5155}}, which specifies
whether or not that NSEC3 record provides proof of non-existence or
not.  In general, NSEC3 with the Opt-Out flag enabled should only be
used in large, highly dynamic zones with a small percentage of signed
delegations.  Operationally, this allows for fewer signature creations
when new delegations are inserted into a zone.  This is typically only
necessary for extremely large registration points providing zone
updates faster than real-time signing allows or when using
memory-constrained hardware.  Smaller zones, or large but relatively
static zones, are encouraged to use a Flags value of 0 (zero) and take
advantage of DNSSEC's proof-of-non-existence support.

## Iterations

NSEC3 records are created by first hashing the input domain and then
repeating that hashing algorithm a number of times based on the
iterations parameter in the NSEC3PARM and NSEC3 records.  The first
hash is typically sufficient to discourage zone enumeration performed
by "zone walking" an NSEC or NSEC3 chain.  Only determined parties
with significant resources are likely to try and uncover hashed
values, regardless of the number of additional iterations performed.
If an adversary really wants to expend significant CPU resources to
mount an offline dictionary attack on a zone's NSEC3 chain, they'll
likely be able to find most of the "guessable" names despite any
level of additional hashing iterations.

Most names published in the DNS are rarely secret or unpredictable.
They are published to be memorable, used and consumed by humans.  They
are often recorded in many other network logs such as email logs,
certificate transparency logs, web page links, intrusion detection
systems, malware scanners, email archives, etc.  Many times a simple
dictionary of commonly used domain names prefixes (www, ftp, mail,
imap, login, database, etc) can be used to quickly reveal a large
number of labels within a zone.  Because of this, there are increasing
performance costs yet diminishing returns associated with applying
additional hash iterations beyond the first.

Although Section 10.3 of {{RFC5155}} specifies upper bounds for the
number of hash iterations to use, there is no published guidance for
zone owners about good values to select.  Because hashing provides
only moderate protection, as shown recently in academic studies of
NSEC3 protected zones {{GPUNSEC3}}{{ZONEENUM}}, this document
recommends that zone owners SHOULD use an iteration value of 0 (zero),
indicating that only the initial hash value should be placed into a
DNS zone's NSEC3 records.

## Salt

Salts add yet another layer of protection against offline, stored
dictionary attacks by combining the value to be hashed (in our case, a
DNS domainname) with a randomly generated value.  This prevents
adversaries from building up and remembering a dictionary of values
that can translate a hash output back to the value that it derived from.

In the case of DNS, it should be noted that the hashed names placed in
NSEC3 records already include the fully-qualified domain name from
each zone.  Thus, no single pre-computed table works to speed up
dictionary attacks against multiple target zones.  An attacker is
required to compute a complete dictionary per zone, which is expensive
in both storage and CPU time.

To protect against a dictionary being built and used for a target
zone, an additional salt field can be included and changed on a
regular basis, forcing a would-be attacker to repeatedly compute a new
dictionary (or just do trial and error without the benefits of
precomputation).

Changing a zone's salt value requires the construction of a complete
new NSEC3 chain.  This is true both when resigning the entire zone at
once, or incrementally signing it in the background where the new salt
is only activated once every name in the chain has been completed.

Most users of NSEC3 publish static salt values that never change.
This provides no added security benefit (because the complete fully
qualified domain name is already unique).  If no rotation is planned,
operators are encouraged to forgo the salt entirely by using a
zero-length salt value instead (represented as a "-" in the 
presentation format).

# Recommendations for deploying and validating NSEC3 records

The following subsections describe recommendations for the different
operating realms within the DNS.

## Best-practice for zone publishers

First, if the operational or security features of NSEC3 are not
needed, then NSEC SHOULD be used in preference to NSEC3. NSEC3
requires greater computational power for both authoritative servers
and validating clients.  Specifically, there is a non trivial
complexity in finding matching NSEC3 records to randomly generated
prefixes within a DNS zone.  NSEC mitigates this concern, and if NSEC3
must be used then selecting a low iterations count will help alleviate
this computational burden.  Note that deploying NSEC with minimally
covering NSEC records [RFC4470] also incurs a cost, and zone owners
should measure the computational difference in deploying both RFC4470
or NSEC3.

In short, for all zones, the recommended NSEC3 parameters are as shown
below:

    ; SHA-1, no extra iterations, empty salt:
    ;
    bcp.example. IN NSEC3PARAM 1 0 0 -

For small zones, the use of opt-out based NSEC3 records is NOT
RECOMMENDED.

For very large and sparsely signed zones, where the majority of the
records are insecure delegations, opt-out MAY be used.

Since the NSEC3PARAM RR is not used by validating resolvers (see
[RFC5155] section 4) the iterations and salt parameters can be changed
without the need to wait for RRsets to expire from caches.  A complete
new NSEC3 chain needs to be constructed and the zone resigned.

## Recommendation for validating resolvers

Because there has been a large growth of open (public) DNSSEC
validating resolvers that are subject to compute resource constraints
when handling requests from anonymous clients, this document
recommends that validating resolvers should change their behavior with
respect to large iteration values.  Validating resolvers SHOULD return
an insecure response when processing NSEC3 records with iterations
larger than 100.  Validating resolvers MAY return SERVFAIL when
processing NSEC3 records with iterations larger than 500.  Note that
this significantly decreases the requirements originally specified in
Section 10.3 of {{RFC5155}}.

Note that a validating resolver MUST still validate the signature over
the NSEC3 record to ensure the iteration count was not altered since
record publication (see {{RFC5155}} section 10.3).

Validating resolvers returning an insecure or SERVFAIL answer in this
situation SHOULD return an Extended DNS Error (EDE) {RFC8914} EDNS0
option of value [TBD].

## Recommendation for Primary / Secondary relationships

Primary and secondary authoritative servers for a zone that are not
being run by the same operational staff and/or using the same software
and configuration must take into account the potential differences in
NSEC3 iteration support.

Operators of secondary services should advertise the parameter limits
that their servers support. Correspondingly, operators of primary
servers need to ensure that their secondaries support the NSEC3
parameters they expect to use in their zones.  To ensure reliability,
after primaries change their iteration counts, they should query their
secondaries with known non-existent labels to verify the secondary
servers are responding as expected.

# Security Considerations

This entire document discusses security considerations with various
parameters selections of NSEC3 and NSEC3PARAM fields.

# Operational Considerations

This entire document discusses operational considerations with various
parameters selections of NSEC3 and NSEC3PARAM fields.

# IANA Considerations

This document requests a new allocation in the "Extended DNS Error
Codes" of the "Domain Name System (DNS) Parameters" registration
table with the following characteristics:

+ INFO-CODE: [TBD]
+ Purpose: Unsupported NSEC3 iterations value
+ Reference: [this document]

--- back

# Acknowledgments

The authors would like to thank the dns-operations discussion
participants, which took place on mattermost hosted by DNS-OARC.

Additionally, the following people contributed text or review comments
to the draft:

+ Tony Finch
+ Alexander Mayrhofer
+ Florian Obser

# Github Version of this document

While this document is under development, it can be viewed, tracked,
issued, pushed with PRs, ... here:

https://github.com/hardaker/draft-hardaker-dnsop-nsec3-guidance

# Implementation Notes

The following implementations have implemented the guidance in this
document.  They have graciously provided notes about the details of
their implementation below.

## OpenDNSSEC

The OpenDNSSEC configuration checking utility will alert the user
about nsec3 iteration values larger than 100.

## PowerDNS

PowerDNS 4.5.2 changed the default value of nsec3-max-iterations to 150.

## Knot DNS and Knot Resolver

Knot DNS 3.0.6 warns when signing with more than 20 NSEC3 iterations.
Knot Resolver 5.3.1 treats NSEC3 iterations above 150 as insecure.

## Google Public DNS Resolver

Google Public DNS treats NSEC3 iterations above 100 as insecure since
September 2021.
