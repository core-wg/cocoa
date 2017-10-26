---
title: CoAP Simple Congestion Control/Advanced
abbrev: CoAP Simple CoCoA
docname: draft-ietf-core-cocoa-latest
date: 2017-10-26

stand_alone: true

ipr: trust200902
area: Internet
wg: CoRE Working Group
kw: Internet-Draft
cat: info

pi:
  toc: yes
  tocdepth: 4
  sortrefs: yes
  symrefs: yes

author:
      -
        ins: C. Bormann
        name: Carsten Bormann
        org: Universität Bremen TZI
        street: Postfach 330440
        city: Bremen
        code: D-28359
        country: Germany
        phone: +49-421-218-63921
        email: cabo@tzi.org
      -
        ins: A. Betzler
        name: August Betzler
        org: Fundació i2CAT
        street:
          - Mobile and Wireless Internet Group
          - C/ del Gran Capità, 2
        city: Barcelona
        code: 08034
        country: Spain
        email: august.betzler@entel.upc.edu
      -
        ins: C. Gomez
        name: Carles Gomez
        org: Universitat Politecnica de Catalunya/Fundacio i2CAT
        street:
          - Escola d'Enginyeria de Telecomunicacio i Aeroespacial
          - "    de Castelldefels"
          - C/Esteve Terradas, 7
        city: Castelldefels
        code: 08860
        country: Spain
        phone: +34-93-413-7206
        email: carlesgo@entel.upc.edu
      -
        ins: I. Demirkol
        name: Ilker Demirkol
        org: Universitat Politecnica de Catalunya/Fundacio i2CAT
        street:
          - Departament d'Enginyeria Telematica
          - C/Jordi Girona, 1-3
        city: Barcelona
        code: 08034
        country: Spain
        email: ilker.demirkol@entel.upc.edu

normative:
  RFC2119:
  RFC2914: ccprinciples
#  RFC5405: udpusage
  RFC7252: coap
  RFC8085: udpusage
  RFC8174:

informative:
  RFC7641: observe
#  RFC7959: block
  I-D.eggert-core-congestion-control: eggert
  I-D.bormann-core-congestion-control: bormann
  Betzler2013:
    author:
      -
        name: August Betzler
        ins: A. Betzler
      -
        name: Carles Gomez
        ins: C. Gomez
      -
        name: Ilker Demirkol
        ins: I. Demirkol
      -
        name: Josep Paradells
        ins: J. Paradells
    title: Congestion control in reliable CoAP communication
    seriesinfo:
      "ACM MSWIM'13": p. 365-372
      DOI: 10.1145/2507924.2507954
    date: 2013
  Betzler2015:
    author:
      - ins: A. Betzler
      - ins: C. Gomez
      - ins: I. Demirkol
      - ins: J. Paradells
    title: "CoCoA+: an Advanced Congestion Control Mechanism for CoAP"
    seriesinfo:
      "Ad Hoc Networks Vol. 33": pp. 126-139
      DOI: 10.1016/j.adhoc.2015.04.007
    date: 2015-10


--- abstract

The CoAP protocol needs to be implemented in such a way that it does
not cause persistent congestion on the network it uses.  The CoRE CoAP
specification defines basic behavior that exhibits low risk of
congestion with minimal implementation requirements.  It also leaves
room for combining the base specification with advanced congestion
control mechanisms with higher performance.

This specification defines some simple advanced CoRE Congestion
Control mechanisms, Simple CoCoA.  It is making use of input from
simulations and experiments in real networks.

--- middle

Introduction
============

The CoAP protocol needs to be implemented in such a way that it does
not cause persistent congestion on the network it uses.  The CoRE CoAP
specification defines basic behavior that exhibits low risk of
congestion with minimal implementation requirements.  It also leaves
room for combining the base specification with advanced congestion
control mechanisms with higher performance.

The present specification defines such an advanced CoRE Congestion
Control mechanism, with the goal of improving performance while
retaining safety as well as the simplicity that is appropriate for
constrained devices.  Hence, we are calling this mechanism Simple
CoCoA (Congestion Control/Advanced).

In the Internet, congestion control is typically implemented in a way
that it can be introduced or upgraded unilaterally.  Still, a new
congestion control scheme must not be introduced lightly.  To ensure
that the new scheme is not posing a danger to the network,
considerable work has been done on simulations and experiments in real
networks.  Some of this work will be mentioned in "Discussion" subsections in
the following sections; an overview is given in {{evidence}}.
Extended rationale for this specification can also be found in the
historical Internet-Drafts {{-bormann}} and {{-eggert}}, as well as in
the minutes of the IETF 84 CoRE WG meetings.

Terminology
-----------

This specification uses terms from {{-coap}}.  In addition, it defines
the following terminology:

Initiator:
: The endpoint that sends the message that initiates an exchange.
  E.g., the party that sends a confirmable message, or a
  non-confirmable message conveying a request.

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and
"OPTIONAL" in this document are to be interpreted as described in BCP
14 {{RFC2119}} {{RFC8174}} when, and only when, they appear in all
capitals, as shown here.

(Note that the present document is itself informational, but it is
discussing normative statements about behavior that makes the
congestion control scheme work in a safe manner.)

The term "byte", abbreviated by "B", is used in its now customary
sense as a synonym for "octet".

Context
=====

In the definition of the CoAP protocol {{-coap}}, an approach was
taken that includes a very simple basic scheme (lock-step with the
number of parallel exchanges usually limited to 1) in the base
specification together with performance-enhancing advanced mechanisms.

The present specification is based on the approved text in the
{{-coap}} base specification.  It is making use of the text that
permits advanced congestion control mechanisms and allows them to
change protocol parameters, including NSTART and the binary
exponential backoff mechanism.  Note that Section 4.8 of {{-coap}}
limits the leeway that implementations have in changing the CoRE
protocol parameters.

The present specification also assumes that, outside of exchanges,
non-confirmable messages can only be used at a limited rate without an
advanced congestion control mechanism (this is mainly relevant for
{{-observe}}).  It is also intended to address the {{-udpusage}} guideline
about combining congestion control state for a destination; and to
clarify its meaning for CoAP using the definition of an endpoint.

The present specification does not address multicast or dithering
beyond basic retransmission dithering.

Area of Applicability
=====================

The present algorithm is intended to be generally applicable.
The objective is to be "better" than default CoAP congestion control
in a number of characteristics, including achievable goodput for a
given offered load, latency, and recovery from bursts, while providing more predictable
stress to the network and the same level of safety from catastrophic congestion.
It does require three state variables per scope plus the state needed
to do RTT measurements, so it may not be applicable to the most
constrained devices (class 1 as per {{?RFC7228}}).

The scope of each instance of the algorithm in the current set of
evaluations has been the five-tuple, i.e., CoAP + endpoint (transport
address) for Initiator and Responder.  Potential applicability to
larger scopes needs to be examined.

Advanced CoAP Congestion Control: RTO Estimation {#rto}
================================================
<!-- Carles: clarify RTT vs. RTO -->

For an initiator that plans to make multiple requests to one
destination endpoint, it may be worthwhile to make RTT measurements in
order to obtain a better RTO estimation than that implied by the
default initial timeout of 2 to 3 s.  This is based on the usual
algorithms for RTO estimation {{?RFC6298}}, with appropriately
extended default/base values, as proposed in {{mod}}.  Note that
such a mechanism must, during idle periods, decay RTO estimates that
are shorter or longer than the basic RTO estimate back to the basic RTO
estimate, until fresh measurements become available again, as proposed
in {{aging}}.

One important consideration not relevant for TCP is the fact that a
CoAP round-trip may include application processing time, which may be
hard to predict, and may differ between different resources available
at the same endpoint.
Also, for communications with networks of constrained devices that
apply radio duty cycling, large and variable round-trip times are
likely to be observed.
Servers will only trigger their early ACKs (with a
non-piggybacked response to be sent later) based on the default
timers, e.g. after 1 s. A client that has arrived at a RTO estimate
shorter than 1 s SHOULD therefore use a larger backoff factor for
retransmissions to avoid expending all of its retransmissions in the
default interval of 2 to 3 s.  A proposal for a mechanism with variable
backoff factors is presented in {{mod}}.
<!-- Do we have that below already??? -->

It may also be worthwhile to do RTT estimates not just based on
information measured from a single destination endpoint, but also
based on entire hosts (IP addresses) and/or complete prefixes (e.g.,
maintain an RTT estimate for a whole /64).  The exact way this can be
used to reduce the amount of state in an initiator is for further
study.

## Blind RTO Estimate

The initial RTO estimate for an endpoint is set to 2 seconds (the
initial RTO estimate is used as the initial value for both E_weak_ and
E_strong_ below).

If only the initial RTO estimate is available, the RTO estimate for
each of up to NSTART exchanges started in parallel is set to 2 s times
the number of parallel exchanges, e.g. if two
exchanges are already running, the initial RTO estimate for an
additional exchange is 6 seconds.

## Measured RTO Estimate

The RTO estimator runs two copies of the algorithm defined in
{{!RFC6298}}, as modified in {{mod}}:
One copy for exchanges that complete on initial transmissions (the
"strong estimator", E_strong_), and one copy for exchanges that have run into
retransmissions, where only the first two retransmissions are
considered (the "weak estimator", E_weak_).  For the latter, there is some
ambiguity whether a response is based on the initial transmission or
the retransmissions.  For the purposes of the weak estimator, the time
from the initial transmission counts.  Responses obtained after the third retransmission are
not used to update an estimator.


The overall RTO estimate is an exponentially weighted moving average
(alpha = 0.5 and 0.25, respectively) computed of the strong and the weak estimator, which is
evolved after each contribution to the weak estimator (1) or to the strong
estimator (2), from the estimator that made the most recent contribution:

> RTO := 0.25 * E_weak_ + 0.75 * RTO  (1)

> RTO := 0.5 * E_strong_ + 0.5 * RTO  (2)

(Splitting this update into the two cases avoids making the
contribution of the weak estimator too big in naturally lossy
networks.)

### Modifications to the algorithm of RFC 6298 {#mod}

This subsection presents three modifications that must be applied to
the algorithm of {{?RFC6298}} as per this document.  The first two
recommend new parameter settings.  The third one is the variable
backoff factor mechanism.

The initial value for each of the two RTO estimators is 2 s.

For the weak estimator, the factor K (the RTT variance multiplier) is
set to 1 instead of 4.  This is necessary to avoid a strong increase
of the RTO in the case that the RTTVAR value is very large, which may
be the case if a weak RTT measurement is obtained after one or more
retransmissions.

If an RTO estimation is lower than 1 s or higher than 3 s, instead of
applying a binary backoff factor in both cases, a variable backoff
factor is used. For RTO estimations below 1 s, the RTO for a
retransmission is multiplied by 3, while for estimations above 3 s,
the RTO is multiplied only by 1.5 (this updated choice of numbers to
be verified by more simulations).  This helps to avoid that exchanges
with small initial RTOs use up all retransmissions in a short interval
of time and exchanges with large initial RTOs may not be able to carry
out all retransmissions within MAX_TRANSMIT_WAIT (93 s).

The binary exponential backoff is truncated at 32 seconds.
Similar to the way retransmissions are handled in the base specification, they
are dithered between 1 x RTO and ACK\_RANDOM\_FACTOR x RTO.

### Discussion

In contrast to {{RFC6298}}, this algorithm attempts to make use of
ambiguous information from retransmissions.  This is motivated by the
high non-congestion loss rates expected in constrained node networks,
and the need to update the RTO estimators even in the presence of
loss.  This approach appears to contravene the mandate in Section
3.1.1 of {{-udpusage}} that "latency samples MUST NOT be derived from
ambiguous transactions".  However, those samples are not simply
combined into the strong estimator, but are used to correct the
limited knowledge that can be gained from the strong RTT measurements
by employing an additional weak estimator.  Evidence that has been
collected from experiments appears to support that the overall effect
of using this data in the way described is beneficial ({{evidence}}).

Some evaluation has been done on earlier versions of this
specification {{Betzler2013}}.  A more recent (and more comprehensive)
reference is {{Betzler2015}}.

## Lifetime, Aging {#aging}

The state of the RTO estimators for an endpoint SHOULD be kept as long
as possible.  If other state is kept for the endpoint (such as a DTLS
connection), it is very strongly RECOMMENDED to keep the RTO state
alive at least as long as this other state.  It MUST be kept for at
least 255 s.

If an estimator has a value that is lower than 1 s, and it is left
without further update for 16 times its current value, the RTO
estimate is doubled.
If an estimator has a value that is higher than 3 s, and it is left
without further update for 4 times its current value, the RTO estimate
is set to be

>   RTO := 1 s + (0.5 * RTO)

(Note that, instead of running a timer, it is possible to implement
these RTO aging calculations cumulatively at the time the estimator is used next.)

Advanced CoAP Congestion Control: Non-Confirmables
==================================================

A CoAP endpoint MUST NOT send non-confirmables to another CoAP endpoint
at a rate higher than defined by this document.
Independent of any congestion control mechanisms, a CoAP endpoint can
always send non-confirmables if their rate does not exceed 1 B/s.

Non-confirmables that form part of exchanges are governed by the rules
for exchanges.

Non-confirmables outside exchanges (e.g., {{-observe}} notifications
sent as non-confirmables) are governed by the following rules:

1. Of any 16 consecutive messages towards this endpoint that aren't
   responses or acknowledgments, at least 2 of the messages must be
   confirmable.

2. The confirmable messages must be sent under an RTO estimator, as
   specified in {{rto}}.

3. The packet rate of non-confirmable messages cannot exceed 1/RTO,
   where RTO is the overall RTO estimator value at the time the
   non-confirmable packet is sent.

## Discussion

This is relatively conservative.  More advanced versions of this
algorithm could run a TFRC-style Loss Event Rate calculator {{?RFC5348}} and apply
the TCP equation to achieve a higher rate than 1/RTO.

{{-observe}}, Section 4.5.1, specifies that the rate of NONs SHOULD
NOT exceed 1/RTT on average, if the server can maintain an RTT
estimate for a client.  CoCoA limits the packet rate of NONs in this
situation to 1/RTO.
Assuming that the RTO estimation in CoCoA works as expected, RTO[k]
should be slightly greater than the RTT[k], thus CoCoA would be more
conservative.  The expectation therefore is that complying with the
NON rate set by CoCoA leads to complying with {{-observe}}.

IANA Considerations
===================

This document makes no requirements on IANA.
(This section to be removed by RFC editor.)

Security Considerations
=======================

The security considerations of, e.g., {{?RFC5681}},
{{RFC2914}}, and {{-udpusage}} apply.  Some issues are already discussed
in the security considerations of {{-coap}}.

<!-- Add text on attacks on the CC -->

--- back

# Supporting evidence {#evidence}

(Editor's note: The references local to this appendix may need to be
merged with those from the specification proper, depending on the
discretion of the RFC editor.)

CoCoA has been evaluated by means of simulation and experimentation in
diverse scenarios comprising different link layer technologies,
network topologies, traffic patterns and device classes. The main
overall evaluation result is that CoCoA consistently delivers a
performance which is better than, or at least similar to, that of
default CoAP congestion control. While the latter is insensitive to
network conditions, CoCoA is adaptive and makes good use of RTT
samples.

It has been shown over real GPRS and IEEE 802.15.4 mesh network
testebds that in these settings, in comparison to default CoAP, CoCoA
increases throughput and reduces the time it takes for a network to
process traffic bursts, while not sacrificing fairness. In contrast,
other RTT-sensitive approaches such as Linux-RTO or Peak-Hopper-RTO
may be too simple or do not adapt well to IoT scenarios,
underperforming default CoAP under certain conditions \[1]. On the
other hand, CoCoA has been found to reduce latency in GPRS and Wi-Fi
setups, compared with default CoAP \[2].

CoCoA performance has also been evaluated for non-confirmable traffic
over emulated GPRS/UMTS links and over a real IEEE 802.15.4 mesh
testbed. Results show that since CoCoA is adaptive, it yields better
packet delivery ratio than default CoAP (which does not apply
congestion control to non-confirmable messages) or Observe (which
introduces congestion control that is not adaptive to network
conditions) \[3, 4].


## Older versions of the draft and improvement

CoCoA has evolved since its initial draft version. Its core has
remained mostly stable since draft-bormann-core-cocoa-02. The
evolution of CoCoA has been driven by research work. This process,
including evaluations of early versions of CoCoA, as well as
improvement proposals that were finally incorporated in CoCoA, is
reflected in published works \[5-10].


## References

[1] A. Betzler, C. Gomez, I. Demirkol, J. Paradells, "CoAP congestion
control for the Internet of Things", IEEE Communications Magazine,
July 2016.

[2] F. Zheng, B. Fu, Z. Cao, “CoAP Latency Evaluation”,
draft-zheng-core-coap-lantency-evaluation-00, 2016 (work in progress).

[3] A. Betzler, C. Gomez, I. Demirkol, "Evaluation of Advanced
Congestion Control Mechanisms for Unreliable CoAP Communications",
PE-WASUN, Cancún, Mexico, 2015.

[4] A. Betzler, J. Isern, C. Gomez, I. Demirkol, J. Paradells,
"Experimental Evaluation of Congestion Control for CoAP Communications
without End-to-End Reliability", Ad Hoc Networks, Volume 52, 1
December 2016, Pages 183-194.

[5] A. Betzler, C. Gomez, I. Demirkol, J. Paradells, "Congestion
Control in Reliable CoAP Communication", 16th ACM International
Conference on Modeling, Analysis and Simulation of Wireless and Mobile
Systems (MSWIM'13), Barcelona, Spain, Nov. 2013.

[6] A. Betzler, C. Gomez, I. Demirkol, M. Kovatsch, "Congestion
Control for CoAP cloud services", 8th International Workshop on
Service-Oriented Cyber-Physical Systems in Converging Networked
Environments (SOCNE) 2014, Barcelona, Spain, Sept. 2014.

[7] A. Betzler, C. Gomez, I. Demirkol, J. Paradells, "CoCoA+: an
advanced congestion control mechanism for CoAP", Ad Hoc Networks
journal, 2015.

[8] Bhalerao, Rahul, Sridhar Srinivasa Subramanian, and Joseph
Pasquale. "An analysis and improvement of congestion control in the
CoAP Internet-of-Things protocol." 2016 13th IEEE Annual Consumer
Communications & Networking Conference (CCNC). IEEE, 2016.

[9] I Järvinen, L Daniel, M Kojo, "Experimental evaluation of
alternative congestion control algorithms for Constrained Application
Protocol (CoAP)", IEEE 2nd World Forum on Internet of Things (WF-IoT),
2015.

[10] Balandina, Ekaterina, Yevgeni Koucheryavy, and Andrei
Gurtov. "Computing the retransmission timeout in coap." Internet of
Things, Smart Spaces, and Next Generation Networking. Springer Berlin
Heidelberg, 2013. 352-362.



Acknowledgements
================
{: numbered="no"}

The first document to examine CoAP congestion control issues in detail
was {{-eggert}}, to which this draft owes a lot.

Michael Scharf did a review of CoAP congestion control issues that
asked a lot of good questions.  Several Transport Area representatives
made further significant inputs this discussion during IETF84,
including Lars Eggert, Michael Scharf, and David Black.  Andrew
McGregor, Eric Rescorla, Richard Kelsey, Ed Beroset, Jari Arkko, Zach
Shelby, Matthias Kovatsch and many others provided very useful
additions.

Authors from Universitat Politecnica de Catalunya have been supported
in part by the Spanish Government's Ministerio de Economía y
Competitividad through projects TEC2009-11453 and TEC2012-32531, and
FEDER.

Carles Gomez has been funded in part by the Spanish Government
(Ministerio de Educacion, Cultura y Deporte) through the Jose
Castillejo grant CAS15/00336.  His contribution to this work has
been carried out in part during his stay as a visiting scholar at the
Computer Laboratory of the University of Cambridge, in collaboration
with Prof. Jon Crowcroft.

--- fluff

<!--  LocalWords:  LWIG interoperable IP RESTful IPv interoperability
 -->
<!--  LocalWords:  implementers optimizations CoRE WPAN LoWPAN Lossy
 -->
<!--  LocalWords:  ICMPv MLD IGMP DNS DHCPv TLS DTLS IPsec RPL KiB IW
 -->
<!--  LocalWords:  Moore's mainloop lookup MTU IEEE datagram PANA API
 -->
<!--  LocalWords:  APIs scalability URIs retransmission confirmable
 -->
<!--  LocalWords:  retransmissions acknowledgement Acknowledgements
 -->
<!--  LocalWords:  EAP PaC PAA PaCs AVPs AVP liveness retransmit IETF
 -->
<!--  LocalWords:  TCP UDP checksums Checksum POSIX CoAP PDUs NoSec
 -->
<!--  LocalWords:  MIDs bitfield datagrams RTT eggert CoAP's ACK coap
 -->
<!--  LocalWords:  lossy multicast TCP's ACKs IANA TBD BCP Unicast WG
 -->
<!--  LocalWords:  latencies RTO backoff bormann NSTART confirmables
 -->
