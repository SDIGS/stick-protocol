# Notice

This repository is a fork of [Stick Protocol](https://github.com/sticknet/stick-protocol),
Copyright © 2018-2024 [Sticknet](https://www.sticknet.org), licensed under the
GNU General Public License v3.0 (see [LICENSE](LICENSE)).

## Additions in this fork

The following are original contributions by Diego Bertola, Copyright © 2026,
developed as part of a bachelor's thesis on formal verification and security
auditing of the Stick Protocol (University of Turin, supervisor: Prof. Idilio Drago).
Licensed under the same GNU General Public License v3.0 as the rest of the project.

- `FormalVerification/ProVerifModels/` — ProVerif models translating and extending
  the original Verifpal models, with strengthened queries (injective authentication,
  differential/ablation analysis, weak-secret guessing attacks) and, where noted in
  the model comments, corrections to the modeled protocol logic.
- `FormalVerification/VerifpalModels/05_malicious_no_explixit_server_pass.vp` — new
  Verifpal model for the malicious-server / key-substitution scenario.

Further audit documentation (correctness notes, cross-checks against the real
Java/Android implementation) is maintained separately as part of the thesis and is
not included in this repository.
