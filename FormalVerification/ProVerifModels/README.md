<h1 align="center">ProVerif Models</h1>
<p align="center">Expected results, and what each verdict means</p>

How to install ProVerif and run these models is in the [root README](../../README.md). This file is the reference to
keep open while reading `outputs/`: it says what each model is expected to answer, and why several of them are expected
to fail.

## Expected results

| Model | Secrecy | Authentication | Notes |
| --- | --- | --- | --- |
| `01_pairwise_session.pv` | holds | holds | The X3DH handshake as specified |
| `01_pairwise_session_no_opk.pv` | **fails** | holds | Same handshake, one-time prekey and DH4 removed |
| `02_sticky_session.pv` | holds | **fails** | Group messaging, no replay protection |
| `02_sticky_session_fixed.pv` | holds | *cannot be proved* | Replay protection via stored counters |
| `02_sticky_session_pass.pv` | holds | holds | Two messages unrolled, replay resistance from the ratchet alone |
| `03_re-establish.pv` | holds\* | **fails** | Backing up private keys to the server |
| `04_backward_secrecy.pv` | holds† | holds | Recovery after a chain key is compromised |
| `04_backward_secrecy_multi.pv` | holds† | **fails** | The same, concurrent sessions |
| `05_malicious_no_explixit_server.pv` | holds | holds | Signature checked against a known key |
| `05_malicious_no_explixit_server_multi.pv` | holds | **fails** | The same, concurrent sessions |

\* The private keys are never derivable, but the model also reports a successful guessing attack against the password.

† Refers to `postY`, sent after the compromise. Both `04_` models also query `postX`, which is meant to fail.

## Reading a verdict

| Output | Meaning |
| --- | --- |
| `not attacker(secret) is true` | The good outcome. The attacker never obtains the value |
| An authentication query `is false` | ProVerif found a concrete attack |
| `cannot be proved` | Settled neither way, which says something about the tool and nothing about the protocol |
| `Weak secret X is false` | The query asks whether the value resists offline guessing, so this verdict means guessing succeeded |

The files in `outputs/` are long, and most of the time only `Verification summary` at the end matters. When something
fails, the part worth reading is the `Derivation` just above it, where each numbered step says what the attacker learns
and from which earlier step.

Three details are easy to misread.

- `attacker_p1` means phase 1, so a value can be safe before a compromise and reachable after it, which is what the
  models with a leak are testing.
- A failure followed by `RESULT (but event(...) is true.)` means only the injective form broke, so the flaw is one of
  multiplicity rather than of value.
- In the replicated models `in copy a` labels which instance of a process acted, and the same goal reached in two
  different copies is the signature of a replay.

## Why the failures are not all alike

**The failure is the result being measured.** `01_pairwise_session_no_opk.pv` is identical to `01_pairwise_session.pv`
except that the one-time prekey, and the exchange depending on it, are removed. Secrecy then fails where it previously
held, which isolates the one-time prekey as the reason forward secrecy survives a later compromise of the long-term
keys. `postX` in the two `04_` models is the same kind of control: it has to fall, because if a leaked key opened
nothing the leak would be testing nothing.

**A replay left open on purpose.** `02_sticky_session.pv`, `04_backward_secrecy_multi.pv` and
`05_malicious_no_explixit_server_multi.pv` all fail the same way: one legitimate message delivered twice to two
instances of the recipient. Only the injective form breaks, so nothing was forged. Closing it needs the stateful
mechanism described below, and the replay is orthogonal to what each of those models was written to measure. The
implementation does track it, in [`GroupCipher.getSenderKey()`](../../android/app/src/main/java/com/stiiick/stickprotocol/internal/GroupCipher.java),
though not for messages marked sticky.

**A limit of the tool.** `02_sticky_session_fixed.pv` answers `cannot be proved` on both correspondence queries. That
is neither a failure nor a proof: saturation does not reach a conclusion, and it says nothing about the protocol in
either direction.

**A genuine weakness.** `03_re-establish.pv` is the one place where a verdict reports a real defect of the protocol,
in the mechanism that restores private keys onto a new device. Both forms of the correspondence break there, so the
flaw is one of value. The weak secret result on the same model compounds it.

## Notes on how they are built

**Only the `01_` models need tuning.** Both carry pruning directives and a `nounif` weighting that appear nowhere
else. Diffie-Hellman commutativity gives the solver an unbounded space to unify over, and without them the run
exhausts memory instead of terminating. The `nounif` only changes the order of exploration; the pruning is an
assumption, and ProVerif verifies it before relying on it, visible in the output as `secrecy assumption verified`.

How much that equation costs is visible in the two models themselves. They differ only in the fourth exchange, and
`01_pairwise_session_no_opk.pv` returns in about six seconds where `01_pairwise_session.pv` needs around eight
minutes. Every other model here finishes in under a second, because none of them declares the equation at all.

**There are three `02_` models because the faithful fix cannot be proved.** `02_sticky_session_fixed.pv` keeps a table
of counters guarded by a lock, which is an accurate picture of what the real sender key store does, and ProVerif cannot
settle it. `02_sticky_session_pass.pv` instead unrolls two messages and shows that the ratchet alone makes an old
message useless once the key has moved on. That is a weaker claim than the real system offers, but one the tool can
establish.

**The `_multi` variants exist to be compared against their single-session counterparts.** The comparison is informative
for `04_` and `05_`, where one version passes and the other does not. The `03_` family needs only the replicated model,
because there the non-injective query fails too, which means one session is enough to exhibit the attack.

## Why `05_malicious.vp` was not translated

That file writes one particular sequence of tampering into the model itself. The file does declare
`attacker[active]`, so the general Dolev-Yao attacker is present; what the script adds is a `Server` principal that
performs a fixed move, and the run that follows is the one the script describes rather than one the tool found.

What that costs is visible in the trace. That principal appends data to the ciphertext, and the first thing Bob does
with the result is `SIGNVERIF(aSigPub, malpost, postSigned)?`. The signature cannot match, which is the intended
point, but the trailing `?` makes it a guard, so the branch is discarded there.
Bob never reaches the decryption, and the confidentiality query ends up answered over an execution in which nothing of
substance took place.

`05_malicious_no_explixit_server.vp` reformulates the question without the scripted move, asking instead what happens
when Bob accepts the verification key from the network rather than from a locally established value. Its two ProVerif
counterparts are in the table above. See [NOTICE.md](../../NOTICE.md) for what this fork adds.
