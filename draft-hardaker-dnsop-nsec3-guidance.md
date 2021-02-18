---
title: "Guidance for NSEC3 parameter settings"
abbrev: title
docname: draft-hardaker-dnsop-nsec3-guidance-00
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

informative:

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
delegations to be covered by a single NSEC3 record.  Opt-out blocks
allow large registries to only sign as many NSEC3 records as there are
signed DS or other RRsets in the zone -- with opt-out, unsigned
delegations don't require additional NSEC3 records.  This sacrifices 
the tamper-resistance proof of non-existence offered by NSEC3 in order
to reduce memory and CPU overheads.

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

The flags field currently contains a single flag, that of the
"Opt-Out" flag {{RFC5155}}, which specifies whether or not NSEC3
records provide proof of non-existence or not.  In general, NSEC3 with
the Opt-Out flag enabled should only be used in large, highly dynamic
zones with a small percentage of signed delegations.  Operationally,
this allows for less signature creations when new delegations are
inserted into a zone.  This is typically only necessary for extremely
large registration points providing zone updates faster than
real-time signing allows.  Smaller zones, or large but relatively
static zones, are encouraged to use a Flags value of 0 (zero) and take
advantage of DNSSEC's proof-of-non-existence support.

## Iterations

Generally increasing the number of iterations offers little improved
protections for modern machinery.  Although Section 10.3 of
{{RFC5155}} specifies upper bounds for the number hash iterations to
use, there is no published guidance on good values to select.  Because
hashing provides only moderate protection, as shown recently in
academic studies of NSEC3 protected zones (tbd: insert ref), this
document recommends using an iteration value of 0 (zero).  This leaves
the creating and verifying hashes with just one application of the
hashing algorithm. 

## Salt

Salts add yet another layer of protection against offline, stored dictionary
attacks by incorporating randomly generated values into the hashed names in the
NSEC3 chain.

It should be noted the the hashed names already include the fully-qualified
name of each zone, so no single pre-computed set of tables works to speed up
dictionary attacks against multiple target zones.  An attacker must perform any
such precomputations separately for each target zone.

Consequently, the additional salt field is only effective when changed
regularly, by forcing a would-be dictionary attacker to repeatedly compute new
tables (or just do trial and error without the benefits of precomputation).

Changing the salt value requires the construction of a complete new chain
for the domain, either when resigning the entire file, or incrementally
in the background, with the new salt only activated once every name
in the chain has been rederived based on the new salt.

Most users of NSEC3 publish static salt values that never change.  These are of
no security benefit, and are best avoided by leaving the salt value empty (zero
length, represented as a "-" in master zone file format).

# Best-practice for zone publishers

In short, for most zones, the recommended NSEC3 parameters are as
shown below:

    ; SHA-1, no opt-out, no extra iterations, empty salt:
    ;
    bcp.example. IN NSEC3PARAM 1 0 0 -

For very large (e.g. 10 million plus unsigned delegations) and only sparsely
signed zones, where the majority of the records are insecure delegations, use
of opt-out may be justified.  In such (large TLD or similar) zones the
alternative parameters are:

    ; SHA-1, with opt-out, no extra iterations, empty salt:
    ;
    example. IN NSEC3PARAM 1 1 0 -

# Recommendation for validating resolvers

Because there has been a large growth of open (public) DNSSEC
validating resolvers that are subject to compute resource constraints
when handling requests from anonymous clients, this document
recommends that validating resolvers should change their behaviour
with respect to large iteration values.  Validating resolvers SHOULD
return a SERVFAIL when processing NSEC3 records with iterations larger
than 100.  Note that this significantly decreases the requirements
originally specified in Section 10.3 of {{RFC5155}}.

# Security Considerations

This entire document discusses security considerations with various
parameters selections of NSEC3 and NSEC3PARAM fields.

# Operational Considerations

This entire document discusses operational considerations with various
parameters selections of NSEC3 and NSEC3PARAM fields.

--- back

# Acknowledgments

dns-operations discussion participants

# Github Version of this document

While this document is under development, it can be viewed, tracked,
issued, pushed with PRs, ... here:

https://github.com/hardaker/draft-hardaker-dnsop-nsec3-guidance
