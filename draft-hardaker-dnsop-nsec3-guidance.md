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

## Requirements notation

   The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
   "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY",
   and "OPTIONAL" in this document are to be interpreted as described
   in BCP 14 {{RFC2119}} {{?RFC8174}} when, and only when, they appear
   in all capitals, as shown here.

# Parameter guidance

## Algorithms

## Flags

## Iterations

Generally increasing the number of iterations offers little improved
protections for modern machinery.  Thus, this document recommends
using an iteration value of 0 (zero), which leaves the creating and
verifying hashes with just one pass.

## Salt Length

--- back

# Acknowledgments

dns-operations discussion participants

# Github Version of this document

While this document is under development, it can be viewed, tracked,
issued, pushed with PRs, ... here:

https://github.com/hardaker/draft-hardaker-dnsop-nsec3-guidance
