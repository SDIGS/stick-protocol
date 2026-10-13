# Notice

This repository is a fork of [Stick Protocol](https://github.com/sticknet/stick-protocol), Copyright © 2018-2024 [Sticknet](https://www.sticknet.org), licensed under the GNU General Public License v3.0 (see [LICENSE](LICENSE)).

Everything listed under Modifications, Additions and Documentation below is by Diego Bertola, Copyright © 2026, added starting August 2026 as part of a bachelor's thesis on formal verification and security auditing of the Stick Protocol (University of Turin, supervisor Prof. Idilio Drago, co-supervisor Prof. Ugo de' Liguoro). Licensed under the same GNU General Public License v3.0 as the rest of the project.

## What the original project already provides

The formal verification in the original project is five Verifpal models under `FormalVerification/VerifpalModels`, added by the project author in July 2024. They cover the four phases of the protocol, pairwise session, sticky session, re-establishment and backward secrecy, plus one model of a malicious server. Between them they ask fourteen queries of two kinds, whether a message reaching a participant was authentic, and whether a given value stays confidential. Three of the five (`01_`, `02_` and `04_`) use Verifpal's `phase` and `leaks` constructs, so they hand long-term keys to the attacker after the fact and ask the confidentiality question again, which is the forward secrecy question. Four are expected to pass and `05_malicious.vp` is expected to fail.

Each model runs a single session, with one copy of each participant. Verifpal's `authentication?` query does not count how many times a message is accepted, so replay falls outside what these models can express. In `03_re-establish.vp` the user's password appears as `confidentiality? alicePass`, which asks whether the password leaks while treating it as a value with full entropy rather than one a person chose.

The work added here starts from that boundary.

## Modifications

`FormalVerification/VerifpalModels/05_malicious_no_explixit_server.vp` is a corrected rewrite of the original `05_malicious.vp`. In the original the attack is written by hand, inside a `Server` principal scripted to misbehave. It appends extra data to the ciphertext and forwards the result together with Alice's original signature. Bob then checks that signature against the modified ciphertext, and it fails, as it has to, since the signature was made over something else. The analysis stops right there, at the primitive check, and never gets far enough to say anything about the security properties.

That does not test the protocol, it tests what happens when a participant is programmed to break it, and the answer is always that it breaks. The rewrite drops the explicit server and leaves the Dolev-Yao attacker to act on its own, which is the point of having one, since it already controls the channel and there is nothing to script.

The original stays in place rather than being replaced, so the correction remains part of the record. The starting point is still reachable, and the step from one model to the other can be followed and questioned.

`FormalVerification/VerifpalModels/01_pairwise_session.vp` has one identifier corrected. Bob derives his root key and sending chain key from `bmaster`, while the value computed on the line above is `bMaster`. Verifpal identifiers are case-sensitive, so the two are different names.

`FormalVerification/VerifpalModels/03_re-establish.vp`, `04_backward_secrecy.vp` and `05_malicious.vp` have their indentation normalized to tabs, matching `01_` and `02_`. Nothing else in them is touched.

`README.md` gains four things: a heading over the Verifpal subsection, which previously sat under Verification Tests untitled; a note on the second `05_` model; a ProVerif Models subsection; and a line under License pointing to this file.

No other file from the original project has been changed.

## Additions

```
FormalVerification/ProVerifModels/
├── README.md          how to run the models, expected results,
│                      and which failures are intentional
├── *.pv               the models themselves
└── outputs/           the output of each model, one file per model
```

Ten models, a translation of the Verifpal set into ProVerif with stronger queries. Authentication becomes injective, so a message accepted twice is a failure rather than a pass. Passwords are declared `weaksecret`, so the question becomes whether they survive offline guessing rather than whether they leak. Five models have no Verifpal counterpart. One removes a feature of the protocol to measure how much depends on it, three run the same protocol in many concurrent sessions rather than one, and one isolates the symmetric ratchet over two sequential messages.

## Documentation

[`FormalVerification/ProVerifModels/README.md`](FormalVerification/ProVerifModels/README.md) is the place to start. Several models are expected to fail, for reasons that differ from one to the next, and reading a failure without that context is misleading. It also covers how to read ProVerif's output, why only the `01_` models need solver tuning, and why some models exist in more than one variant.

## Not yet included

The source-level audit comparing these models against the Android, iOS, server and client handler implementations is kept outside this repository while it goes through coordinated disclosure with the project's maintainers. It will be added here once that process concludes.
