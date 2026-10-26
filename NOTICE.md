# Notice

This repository is a fork of [Stick Protocol](https://github.com/sticknet/stick-protocol), Copyright © 2018-2024 [Sticknet](https://www.sticknet.org), licensed under the GNU General Public License v3.0 (see [LICENSE](LICENSE)).

Everything listed under Modifications, Additions and Documentation below is by Diego Bertola, Copyright © 2026, added starting August 2026 as part of a bachelor's thesis on formal verification and security auditing of the Stick Protocol (University of Turin, supervisor Prof. Idilio Drago, co-supervisor Prof. Ugo De' Liguoro). Licensed under the same GNU General Public License v3.0 as the rest of the project.

## What the original project provided

The formal verification in the original project was five Verifpal models under `FormalVerification/VerifpalModels`, added by the project author in July 2024 and written for Verifpal 0.52.0, the current release until August 2026. Everything said about them in this section describes that version and those files as published. They covered the four phases of the protocol, pairwise session, sticky session, re-establishment and backward secrecy, plus one model of a malicious server. Between them they asked fourteen queries of two kinds, whether a message reaching a participant was authentic, and whether a given value stays confidential. Three of the five (`01_`, `02_` and `04_`) used Verifpal's `phase` and `leaks` constructs, so they handed long-term keys to the attacker after the fact and asked the confidentiality question again, which is the forward secrecy question. Four were expected to pass and `05_malicious.vp` to fail.

Each model ran a single session, with one copy of each participant. In 0.52.0 the `authentication?` query did not count how many times a message was accepted, so replay fell outside what those models could express; Verifpal 1.3 of 26 August 2026 changed that, and the Modifications section records what followed. In `03_re-establish.vp` the user's password appeared as `confidentiality? alicePass`, which asks whether the password leaks while treating it as a value with full entropy rather than one a person chose.

The work added here starts from that boundary.

## Modifications

`FormalVerification/VerifpalModels/05_malicious_no_explixit_server.vp` reformulates the security question the original `05_malicious.vp` was meant to ask, with the network attacker left to act on its own and the verification key anchored to a locally established value. In the original a `Server` principal performs one fixed tampering move. It appends extra data to the ciphertext and forwards the result together with Alice's original signature. Bob then checks that signature against the modified ciphertext, and it fails, as it has to, since the signature was made over something else. The analysis stops right there, at the primitive check, and never gets far enough to say anything about the security properties.

What that execution establishes is narrow. A signature does not verify over a message it was not made over, which holds for any signature scheme and says nothing about Stick. The rewrite drops the scripted server and leaves the Dolev-Yao attacker, which the file already declares, to act on its own.

The original stays in place rather than being replaced, so the correction remains part of the record. The starting point is still reachable, and the step from one model to the other can be followed and questioned.

`FormalVerification/VerifpalModels/01_pairwise_session.vp` has one identifier corrected. Bob derives his root key and sending chain key from `bmaster`, while the value computed on the line above is `bMaster`. Verifpal identifiers are case-sensitive, so the two are different names.

`FormalVerification/VerifpalModels/03_re-establish.vp`, `04_backward_secrecy.vp` and `05_malicious.vp` have their indentation normalized to tabs, matching `01_` and `02_`.

Five of the six models are migrated to the syntax Verifpal 1.x requires and parse under it, since Verifpal 1.0 of 15 August 2026 rewrote the modelling language and the files as published no longer parse. The sixth, `03_re-establish.vp`, carries the same syntax migration and still does not parse, for the reason below. The migration is mechanical. `G^x` becomes `PUBKEY(x)`, `gx^y` becomes `DH_KEX(gx, y)`, and `AEAD_ENC`/`AEAD_DEC` take a nonce as their second argument, for which the `nil` constant already present in the models is used, since they model no nonce and inventing a fresh one would change what they state. Substituting a freshly generated nonce changes no verdict, which was checked. The migration is commit `985fee2`, and `d673146` is the last commit in which these files carry the syntax they were published with.

`03_re-establish.vp` is migrated as far as that goes and remains unrunnable under 1.x. It uses `knows password` and `PW_HASH`, and 1.x removed both. Low-entropy secrets are now expressed by annotating the primitive that consumes them, which is a different modelling decision rather than a different spelling, so rewriting the model to suit it would change what it asserts. The file is left at that point rather than reinterpreted.

The output of each run is in `FormalVerification/VerifpalModels/outputs/`, one file per model, produced with Verifpal 1.6.7. Under that version the migrated models report replay on `02_`, `04_` and `05_malicious_no_explixit_server.vp`, which are the three the ProVerif models in this repository already report through injective correspondences, and they reject `05_malicious.vp` before any query is reached, for the reason given below. `01_pairwise_session.vp` fails there and its ProVerif counterpart does not; the two are different models and the difference is recorded rather than resolved.

`README.md` gains four things: a heading over the Verifpal subsection, which previously sat under Verification Tests untitled; a note on the second `05_` model; a ProVerif Models subsection; and a line under License pointing to this file.

No other file from the original project has been changed.

## Additions

```
FormalVerification/ProVerifModels/
├── README.md          expected results, relevant explanations
│                      and which failures are intentional
├── *.pv               the models themselves
└── outputs/           the output of each model, one file per model
```

Ten models, a translation of the Verifpal set into ProVerif with stronger queries. Authentication becomes injective, so a message accepted twice is a failure rather than a pass. Passwords are declared `weaksecret`, so the question becomes whether they survive offline guessing rather than whether they leak. Five models have no Verifpal counterpart. One removes a feature of the protocol to measure how much depends on it, three run the same protocol in many concurrent sessions rather than one, and one isolates the symmetric ratchet over two sequential messages.

## Documentation

[`FormalVerification/ProVerifModels/README.md`](FormalVerification/ProVerifModels/README.md) is the place to start. Several models are expected to fail, for reasons that differ from one to the next, and reading a failure without that context is misleading. It also covers how to read ProVerif's output, why only the `01_` models need solver tuning, and why some models exist in more than one variant.

## Not yet included

The source-level audit comparing these models against the Android, iOS, server and client handler implementations is kept outside this repository while it goes through coordinated disclosure with the project's maintainers. It will be added here once that process concludes.
