<h1 align="center">ProVerif Models</h1>
<p align="center">A stricter re-verification of the Stick protocol, translated from the Verifpal models</p>

Eleven models. Most of the properties they state are proved, the `03_` pair exposes a genuine weakness of the protocol, and every other verdict that is not `is true` is either the result the experiment was built to measure, a replay left open on purpose, or a limit of what the tool can decide. The table under [Expected results](#expected-results) says which is which, and no verdict in `outputs/` should be read without it.

## Overview

This directory contains a translation of the Verifpal models in [`../VerifpalModels`](../VerifpalModels), in a language more centered on mathematics and formal verification. Where the originals ask whether a message was authentic, these also analyze whether it could have been replayed, and whether it is bound to the identity of whoever actually sent it, through stronger queries.

A few models have no Verifpal counterpart. Those are either variants that remove one feature from the protocol to measure how much it depends on it, or variants that run the same protocol in many concurrent sessions rather than one. It goes the other way as well, and the last section explains which Verifpal model was left untranslated.

## Running them

The models were written and checked against **ProVerif 2.05**, which on most systems comes from opam:

```
opam install proverif
```

The [official install guide](https://bblanche.gitlabpages.inria.fr/proverif/install.html) covers the other ways.

A single model runs with:

```
proverif file_name.pv
```

The output of every model is kept in [`outputs/`](outputs), produced with:

```
proverif file_name.pv > outputs/file_name.txt
```

To regenerate all of them at once:

```
for f in *.pv; do proverif "$f" > "outputs/${f%.pv}.txt"; done
```

### Why the first model takes minutes

Everything finishes in about one second except `01_pairwise_session.pv`, which takes six to seven minutes on an Intel Core i5-1235U with 16 GB of RAM. What costs is the equational theory. The two `01_` models are the only ones that declare Diffie-Hellman commutativity, so the solver has to accept that the same key can be written in more than one way, and each rewriting opens further ones. The base model combines four exchanges into a single secret and the space grows with them.

The asymmetry between the two is also in what each verdict demands. `01_pairwise_session_no_opk.pv` fails early, since an attack is a single derivation and the solver has its answer as soon as it finds one. `01_pairwise_session.pv` holds instead, and holding is a claim about everything that did not happen, so the whole space has to be explored and closed before an answer can be given. It is not stuck, and a long run does not mean it found something; it means it is still ruling things out.

## Reading the results

| Output | Meaning |
| --- | --- |
| `not attacker(secret) is true` | The good outcome. The attacker never obtains the value |
| An authentication query `is false` | ProVerif found a concrete attack |
| `cannot be proved` | Settled neither way, which says something about the tool and nothing about the protocol |
| `Weak secret X is false` | The query asks whether the value resists offline guessing, so this verdict means the guessing attack succeeded |

The files in `outputs/` are long, but most of the time only the end matters, since `Verification summary` collects every verdict in one place. When something fails, the part worth reading is the `Derivation` just above it, where each numbered step says what the attacker learns and from which earlier step, followed by a concrete trace of the attack.

Three details are easy to misread.

- `attacker_p1` means phase 1, so a value can be safe before a compromise and reachable after it, which is what the models with a leak are testing.

- A failure followed by `RESULT (but event(...) is true.)` means only the injective form broke, so the flaw is one of multiplicity rather than of value.

- In the replicated models `in copy a` labels which instance of a process acted, and the same goal reached in two different copies is the signature of a replay.

## Expected results

| Model | Secrecy | Authentication | Notes |
| --- | --- | --- | --- |
| `01_pairwise_session.pv` | holds | holds | The X3DH handshake as specified |
| `01_pairwise_session_no_opk.pv` | **fails** | holds | Same handshake, one-time prekey and DH4 removed |
| `02_sticky_session.pv` | holds | **fails** | Group messaging, no replay protection |
| `02_sticky_session_fixed.pv` | holds | *cannot be proved* | Replay protection via stored counters |
| `02_sticky_session_pass.pv` | holds | holds | Replay resistance from the ratchet alone |
| `03_re-establish.pv` | holds\* | **fails** | Backing up private keys to the server |
| `03_re-establish_multi.pv` | holds\* | **fails** | The same, concurrent sessions |
| `04_backward_secrecy.pv` | holds† | holds | Recovery after a chain key is compromised |
| `04_backward_secrecy_multi.pv` | holds† | **fails** | The same, concurrent sessions |
| `05_malicious_no_explixit_server.pv` | holds | holds | Signature checked against a known key |
| `05_malicious_no_explixit_server_multi.pv` | holds | **fails** | The same, concurrent sessions |

\* The private keys are never leaked, but both `03_` models report a successful guessing attack against the password itself.

† Refers to `postY`, sent after the compromise. Both `04_` models also query `postX`, which is meant to fail. See below.

### What secrecy means in each row

Secrecy is not the same claim everywhere, because most of these models hand the attacker a key partway through and then ask what is still out of reach. Which side of the compromise the protected message sits on is what decides the property being measured, and the verdicts that read `attacker_p1` are the ones where a leak took place.

**Forward secrecy, in `01_` and `02_`.** These put the message before the leak. Alice sends, and only afterwards are the long-term keys in one case, and the chain key in the other, handed over. What they establish is that a compromise does not open what was already sent, and the two get it from different places. In `01_` it rests on a secret that no longer exists, the one-time prekey having been discarded once the handshake is over, and the pair of models is there to measure how much of the property depends on that. In `02_` it rests on the chain key moving on through a function that cannot be run backwards, so the key that encrypted an earlier message cannot be worked out from the one currently held.

**Backward secrecy, in `04_`.** These ask the opposite question. The chain key of a session is leaked, and what is queried is a message sent afterwards, in a session established anew. Holding there means the protocol recovers, that a compromise comes to an end instead of lasting forever, which is also called post-compromise security or healing. Recovery does not come from the ratchet itself, since anyone holding a chain key can keep deriving forward from it indefinitely. It comes from the session being replaced by a fresh one, and that is exactly the boundary these models describe. The companion query `postX`, which sits before the leak, is meant to fall, and why it has to is in the next section.

**No leak, in `03_` and `05_`.** Secrecy there carries its ordinary meaning.

## The verdicts that are not `is true`

Every verdict that is not `is true` falls into one of four categories, and reading them as a single group would be misleading. The categories sort verdicts and not models, so one model can contribute to more than one.

### 1. The failure is the result being measured

Two kinds of verdict are here, and in both the experiment was built around something falling.

**Secrecy in `01_pairwise_session_no_opk.pv`.** The file is identical to `01_pairwise_session.pv` except that the one-time prekey, and the key exchange depending on it, are removed. Everything else is untouched, including the solver directives. Secrecy then fails where it previously held, and since only one variable changed, this isolates the one-time prekey as the reason forward secrecy survives a later compromise of the long-term keys.

**`postX` in the two `04_` models.** Alice sends a message in the old session, that session's chain key is leaked, then she sends another one in a new session. `postX` has to fall, because if a leaked key opened nothing the leak would be testing nothing, and the result on `postY` would say very little. It is the pair that carries the meaning, `postX` falling and `postY` holding.

### 2. A replay left open on purpose

The same failure appears in `02_sticky_session.pv`, `04_backward_secrecy_multi.pv` and `05_malicious_no_explixit_server_multi.pv`. An attacker captures one legitimate message and delivers it twice, to two instances of the recipient, and nothing in those models tracks what has already arrived. Only the injective form breaks, so the message is genuine and nothing was forged.

In `02_sticky_session.pv` the replay is the baseline the other two `02_` models were written against, one faithful and undecidable, one weaker and provable. In the two `_multi` files it was left open, because closing it needs the same stateful mechanism whose modelling limits are described under [Notes on how they are built](#notes-on-how-they-are-built), and the replay is orthogonal to the property each of those two was written to measure.

The implementation does track it, though not everywhere. In [`GroupCipher.getSenderKey()`](../../android/app/src/main/java/com/stiiick/stickprotocol/internal/GroupCipher.java) a message key is normally used at most once, dropped from the store as soon as it is consumed, and a message whose counter has already been seen is refused, so a replay of this shape does not get through. All of it is conditional on the message not being marked sticky. For sticky content the chain key is left where it is and the message key kept, by design, so that the content can be decrypted again later. The receiver's state never moves past that message, and the check that rejects an old counter is never reached. These models say nothing about either case, because they were written to test a different property.

### 3. A genuine weakness of the protocol

Both `03_` models are here, and they are the only place where a verdict reports a real defect. Private keys are backed up encrypted under a key derived from the user's password, but nothing binds each encrypted key to the slot it belongs in, and nothing protects the backup as a whole. An attacker able to tamper with the server response can swap the encrypted keys between slots, and the receiving device accepts them without noticing it has installed the wrong key in the wrong place. Here both forms of the correspondence break, so the flaw is one of value rather than of multiplicity.

The weak secret result compounds it, since the key protecting the backup comes from a password, and the backup itself gives whoever holds it a way to tell a correct guess from a wrong one without ever contacting the server.

### 4. A limit of the tool

`02_sticky_session_fixed.pv` answers `cannot be proved` on both of its correspondence queries. That is neither a failure nor a proof; the saturation simply never reaches a conclusion, and it says nothing about the protocol in either direction. The reason is in the notes below.

## Notes on how they are built

**Only the `01_` models need tuning.** Both carry pruning directives and a `nounif` weighting that appear nowhere else, identical in the two files. Diffie-Hellman commutativity gives the solver an unbounded space to unify over, and without them the run exhausts memory instead of terminating. The `nounif` only changes the order in which the solver explores; the pruning is an assumption, and ProVerif verifies it before relying on it, visible in the output as `secrecy assumption verified`. No other model declares an equational theory, and signatures, MACs and key derivation, which the later models do use, are plain constructors with no equations attached, so nothing there has to be unified.

**There are three `02_` models because the faithful fix cannot be proved.** `02_sticky_session_fixed.pv` keeps a table of counters guarded by a lock, which is an accurate picture of what the real sender key store does. ProVerif cannot settle it, because its reasoning loses precision on table lookups inside replicated processes, and the answer comes back as `cannot be proved` rather than an attack. `02_sticky_session_pass.pv` instead unrolls two messages, showing that the ratchet alone makes an old message useless once the key has moved on, without tracking anything. That is a weaker claim than the real system offers, but one the tool can establish.

**The `_multi` variants exist to be compared against their single-session counterparts.** Read in isolation they are misleading, for the reason given under category 2. The comparison is informative for `04_` and `05_`, where one version passes and the other does not, and the difference is the whole point. For `03_` there is nothing to compare. Both versions fail, the derivations coincide, and the non injective query fails as well, which tells us the attack is about a wrong value and not a repeated one. A single session is enough to exhibit it.

## Why `05_malicious.vp` was not translated

That file writes the attacker's moves into the model itself. Both tools assume a Dolev Yao attacker, one that already controls the network and can read, drop, reorder and forge anything that travels on it, so the tool is meant to find an attack on its own and the model never needs to hand it one. Writing a particular sequence of moves does not add an attacker; it replaces the general one with a scripted and much weaker one, and the run then says nothing about everything that was left unwritten.

Here the consequence is more concrete than a loss of generality. The tampering is performed by a `Server` principal that appends data to the ciphertext, and the first thing Bob does with the result is `SIGNVERIF(aSigPub, malpost, postSigned)?`. The signature cannot match, which is the intended point, but the trailing `?` makes it a guard, so the branch is discarded right there. Bob never reaches the decryption, and the confidentiality query ends up answered over an execution in which nothing of substance took place. The verification never really runs; it stops on a failing primitive, and the result it prints carries no information about the protocol.

`05_malicious_no_explixit_server.vp` was added to cover the same scenario without the scripted server, and its two ProVerif counterparts are in the table above. See [NOTICE.md](../../NOTICE.md) for what this fork adds to the original work.
