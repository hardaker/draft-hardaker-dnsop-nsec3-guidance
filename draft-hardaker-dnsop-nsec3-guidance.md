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
    org: Independent
    email: ietf-dane@dukhovni.org

normative:
  RFC2119:
  RFC5155:
  RFC4035:

informative:

--- abstract

NSEC3 records provides an alternate to NSEC records and offers
security-by-obscurity privacy protection of DNS zone records protected
by DNSSEC.  This document provides guidance with respect to setting
NSEC3 parameters based on recent operational deployment experience.

--- middle

# Introduction

NSEC3 {{RFC5155}} adds a second option for proof of non existence
support for DNSSEC specifications {{RFC4035}} through the creation of
NSEC3 records.  These records obfuscate linking domain names using
hashing algorithms.  NSEC3 also provides opt-in support, allowing for
ranges of records that do not fall into proof-of-non-existence ranges
typically deployed by large registration zones.

Parameters specifying how NSEC3 records are published within a zone
are published in an NSEC3PARAM record at the apex of their zone.
These paremeters are the Hash Algorithm, processing Flags, the number
of hash Iterations and the Salt.  Each of these has security and
operational considerations that impact both zone owners and validating
resolvers.  This document provides some recommendations on selecting
parameters considering these factors.

## Requirements notation

   The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
   "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY",
   and "OPTIONAL" in this document are to be interpreted as described
   in BCP 14 {{RFC2119}} {{?RFC8174}} when, and only when, they appear
   in all capitals, as shown here.

# Parameter guidance

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

Salts add yet another layer of protection against offline, stored
dictionary attacks by using randomly generated values when creating
new records.  The length and usage of a salt value has little
operational concerns beyond bandwidth requirements for transmitting
the salt.  Thus, the primary consideration is whether or not there is
a security benefit to deploying signed zones with salt values.
Operators may choose to use a salt for this reason, though it should
be noted that the use of salts doesn't prevent against guess based
approaches in offline attacks -- only against memorization hash based
lookups.  Thus, the added value is minimal enough that operators may
wish to deploy zones without a hash value at all.

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
