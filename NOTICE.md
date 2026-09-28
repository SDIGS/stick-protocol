# Notice

This repository is a fork of [Stick Protocol](https://github.com/sticknet/stick-protocol), Copyright © 2018-2024 [Sticknet](https://www.sticknet.org), licensed under the GNU General Public License v3.0 (see [LICENSE](LICENSE)).

Everything below is by Diego Bertola, Copyright © 2026, added starting September 2026 as part of a bachelor's thesis on formal verification and security auditing of the Stick Protocol (University of Turin, supervisor Prof. Idilio Drago, co-supervisor Prof. Ugo de' Liguoro). Licensed under the same GNU General Public License v3.0 as the rest of the project.

## Modifications

`FormalVerification/VerifpalModels/05_malicious_no_explixit_server.vp` is a corrected rewrite of the original `05_malicious.vp`. In the original the attack is written by hand, inside a `Server` principal scripted to misbehave: it appends extra data to the ciphertext and forwards the result together with Alice's original signature. Bob then checks that signature against the modified ciphertext, and it fails, as it has to, since the signature was made over something else. The analysis stops right there, at the primitive check, and never gets far enough to say anything about the security properties.

That does not test the protocol, it tests what happens when a participant is programmed to break it, and the answer is always that it breaks. The rewrite drops the explicit server and leaves the Dolev-Yao attacker to act on its own, which is the point of having one: it already controls the channel, so there is nothing to script.

The original stays in place rather than being replaced, so the correction remains part of the record: the starting point is still reachable, and the step from one model to the other can be followed and questioned.

`README.md` is modified in two places: a line under the License section points to this file, and the Verification Tests section is updated, since there are now two `05_` models with opposite outcomes.

No other file from the original project has been changed.

## Additions

```
FormalVerification/ProVerifModels/
├── README.md          how to run the models, expected results,
│                      and which failures are intentional
├── *.pv               the models themselves
└── outputs/           the output of each model, one file per model
```

The `.pv` files are a translation of the Verifpal models into ProVerif, with stronger queries: injective authentication, ablation variants that remove one feature to measure how much depends on it, concurrent-session variants, and weak secret queries for values a person chooses rather than a machine.

## Documentation

[`FormalVerification/ProVerifModels/README.md`](FormalVerification/ProVerifModels/README.md) is the place to start. Several models are expected to fail, for reasons that differ from one to the next, and reading a failure without that context is misleading. It also covers how to read ProVerif's output, why only the first model needs solver tuning, and why some models exist in more than one variant.

## Not yet included

The audit work comparing these models against the Java and Android implementation is currently kept outside this repository, as part of the thesis workflow. It will be integrated once the thesis write-up consolidates.
