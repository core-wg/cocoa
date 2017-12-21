- IETF 84: Before CoCoA was created, CoRE WG had a special session on
  congestion control in CoAP, with input from transport area people,
  as collected in the acknowledgments section of the draft.  The
  initial version of CoCoA was generated a few days after that IETF
  meeting.

- IETF 87: a short "external report" given to CoRE by August Betzler
  on the first simulations the authors ran on
  draft-bormann-core-cocoa-00. There were some comments by Zach Shelby
  and Akbar Rahman (CoRE regulars).

- IETF 89: presented draft-bormann-core-cocoa-01 to CoRE (improvements
  where the initial version led to underperformance). Some feedback
  from Zach (suggesting other evaluation scenarios). Matthias Kovatsch
  explained there were plans to incorporate CoCoA in Cf (leading Java
  implementation of CoAP).

- IETF 90: presented draft-bormann-core-cocoa-02, after having invited
  several congestion control people to that agenda item:

    *  Lars Eggert and Richard Scheffenegger had questions on whether
       the strong and the weak estimators would be too complex for
       constrained devices.
    *  Zach Shelby and Lars had comments on scaling up the number of
       nodes considered in the evaluation.
    *  Jörg Ott asked about the topologies considered and stability of
       the mechanism (we had already done a first fine-tuning).
    *  Lars had more input on further metrics related with safety of
       the mechanism
    *  Robby Simpson asked whether CoCoA is TCP friendly (it is).
    *  Lars suggested testing strong-only and also weak-only
    *  Richard: also RTO over time
    *  Richard on being aggressive vs protecting the network
    *  Robby, on losses due to wireless, not to congestion (weak
       estimator for that!)
    *  Ari Keränen, suggesting using a subset of CoCoA for more
       constrained devices
    *  Pascal Thubert and Lars on using ECN
    *  Richard on considering alternative RTO algorithms with other
       properties
    *  Lars saying this can be a starting point, but work will take
       years (he was right :))

- IETF 91: With the draft not updated, authors presented results to
  CoRE (of several RTO algorithms including CoCoA) over GPRS. Zach
  Shelby and Ari Keränen supportive, Ari asking about memory footprint
  and about scenarios combining default CoAP nodes and CoCoA nodes,
  etc.

- IETF 92: Presentation at ICCRG, including results over 15.4 mesh
  testbed.  Richard Scheffenegger thanked the presenter for addressing
  all his concerns as raised in CoRE. Some clarifying questions by
  Michael Welzl.

- IETF 94: presented draft-bormann-core-cocoa-03 to CpRE, now
  including results on NONs. Some discussion with Markku Kojo/Ilpo
  Järvinen, who also experimented with CoCoA and with alternatively
  porting over Linux TCP CC algorithms to CoAP.

- IETF 96: No draft update; presentation of new evaluation results to
  CoRE. Zach Shelby and Zhen Cao supportive of WG adoption. Support in
  the room. Some questions from Hannes Tschofenig. Gabriel Montenegro
  asking about review from transport area.

- IETF 97: there were some slides in the CoRE meeting deck (results on
  Aggregate Congestion Control), but a presentation was not actually
  given due to lack of agenda time.

- IETF 98: presented draft-ietf-core-cocoa-01 to CoRE; asked for WGLC.

- IETF 99: presented authors' position to CoRE regarding comments by
  Michael Scharf and Ingemar Johansson. Plans for WGLC after
  addressing the comments. No comments in the room.

- IETF 100: presented draft-ietf-core-cocoa-02 to CoRE, intended for
  WGLC.  No further comments in the room.
