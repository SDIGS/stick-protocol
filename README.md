<h1 align="center">Stick Protocol</h1>
<p align="center">Re-Establishable Group End-to-End Encryption with Post-Compromise Security</p>

## Motivation

End-to-end encryption has become a de facto standard in messengers, especially after the outbreak of the highly secure
messaging protocol – Signal. However, this high adoption of secure end-to-end communications has been limited to
messengers, and has not yet seen a noticeable trace in social network platforms, despite the increase in users’ privacy
violations. The Stick protocol is an end-to-end encryption protocol, based on the Signal protocol, specifically designed
for social networks. The Stick protocol supports re-establishable "many-to-many" encryption sessions in an asynchronous
and multi-device setting while preserving forward secrecy and introducing backward secrecy. Performance evaluation of
the Stick protocol shows that it causes no noticeable compromise on usability or performance. The Stick protocol's
scientific foundation is 📄 <a href="https://www.sticknet.org/stick-protocol.pdf">published in IEEE</a> for those
interested in the
project’s technical and research motivations.

Stick is the protocol used in <a href="https://www.sticknet.org">Sticknet</a> - a secure social storage platform.

## Security Features

The Stick protocol benefits from the security features of the <a href="https://signal.org/docs/">Signal protocol<a/>, and adds the following new features, or as summarized in <a href="https://omarbasem.com/PDFs/StickProtocolPoster.pdf">this diagram</a>:

1. <b>♻️ Re-establishable</b>: A user will be able to securely re-establish their encryption sessions when re-installing the application, or installing on a new device

2. <b>📱💻 Multi Device</b>: Stick protocol provides end-to-end encryption for all of a user's devices.

3. <b>🔐 Perfect forward & backward secrecy for sharing sender keys</b>: provided by multiple pairwise sessions

4. <b>🔗 Many-to-many backward secrecy</b>: sticky sessions provides backward secrecy for group communications every a maximum of N Encryptions.

5. <b>💖 Identity Keys self-healing</b>: A user's identity key refreshes every while to mitigate the effect of an identity key compromise

6. <b>🛡️ Double-Hashing</b>: for complete hiding of the user's password, even from the service provider at login

## Technical Documentation

<a href="https://www.sticknet.org/stick-protocol">Click here<a/> for technical documentation of the Stick protocol.

## Usage Documentation

<a href="https://www.sticknet.org/stick-protocol/usage-documentation">Click here<a/> for usage documentation of the
Stick
protocol.

## Installation

The Stick protocol was implemented to be a superset to the Signal protocol making the Stick protocol logic external to
the Signal protocol. This allows the Signal protocol to be used in parallel with the Stick protocol, from just the Stick
protocol library. The stick protocol was implemented to be a fully comprehensive Android and iOS library (rather than
just a Java and C library) which can be simply dropped into a social network application, and provide E2EE using
re-establishable "sticky sessions", with as low development overhead as possible. The Stick protocol implementation is
composed of 4 libraries:

- Android Library (Gradle Package)
- iOS Library (CocoaPod Framework)
- Server Library (PIP Package)
- Client Handlers Library (NPM Package)

The Android library and the iOS library are the 2 main libraries of the Stick protocol. They have most of the logic
needed on the client-side. There is also a server library for the Stick protocol in Python. In addition, there is a
client handlers library in JavaScript which contains common handler methods needed for the Stick protocol client-side.

### Android

Gradle:

```gradle
dependencies {
   implementation 'com.github.sticknet:stick-protocol:+'
}
```

The <a href="https://github.com/sticknet/stick-protocol/blob/main/android/app/src/main/java/com/stiiick/stickprotocol/main/StickProtocol.java">
main StickProtocol java class file</a> has usage documentation as well. It includes all the methods that you would need.

### iOS

Check iOS installation [guide](./ios_installation.md).

The <a href="https://github.com/sticknet/stick-protocol/blob/main/ios/StickProtocol/StickProtocol/Main/StickProtocol.swift">
main StickProtocol Swift class file</a> has usage documentation. It includes all the methods that you would
need.

### Server

This is a server library for the Stick protocol in Python for Django. If you have a Django server you can use this
library. If not, you can easily implement your own.
The <a href="https://github.com/stickapp/stick-protocol/blob/main/server/stick_protocol/stick_protocol.py">main
StickProtocol python class<a/> includes full usage documentation needed on the server.

```
pip3 install stick-protocol-server
```

### Client Handlers

The Stick protocol implementation features a client handlers library in TypeScript. It contains common handler methods
needed for the Stick protocol client-side. These handlers may differ from one application to another. A developer is
free to write their own handlers. They can be implemented in any programming language.
The <a href="https://github.com/sticknet/stick-protocol/blob/main/client-handlers/src/StickProtocolHandlers.ts">main
TypeScript class</a> contains full usage documentation.

```
yarn add stick-protocol-handlers@1.2.2
```

## Verification Tests

The protocol's core properties are checked by two generations of formal models: a legacy set in
<a href="https://verifpal.com/">Verifpal</a>, kept for reference, and a stricter translation into
<a href="https://bblanche.gitlabpages.inria.fr/proverif/">ProVerif</a>, which is the one to trust going forward.

### Verifpal Models

Located under `FormalVerification/VerifpalModels` and run with `verifpal verify test_file.vp`. They were written for
Verifpal 0.52.0, under which tests `01`-`04` pass and `05_malicious.vp` fails. Verifpal 1.0 rewrote the modelling
language; on the migrated syntax (1.6.7) the verdicts shift — `01_` now fails both queries, `02_` and `04_` fail
authentication but pass confidentiality, and `03_re-establish.vp` only runs after a modelling decision forced by 1.x
dropping the constructs the original password model used, so its verdict reflects that decision rather than the
original model. `05_malicious_no_explicit_server.vp` passes under 0.52.0 and keeps its confidentiality result under
1.6.7, but now fails authentication on a replay that version can see; it covers the same scenario as `05_malicious.vp`
without a scripted server principal, which in the original leaves the test unable to say anything about the protocol
once the signature check fails. Outputs are in `outputs/`; what the migration changed and what each verdict means is
in [NOTICE.md](./NOTICE.md).

### ProVerif Models

Located under `FormalVerification/ProVerifModels`: a translation of the same models with stricter queries. Where
Verifpal asks only whether a message is authentic, these also ask whether it could have been replayed, whether it
binds to the sender's identity, and — where a password stands in for a secret — whether it survives offline guessing.
Six of the eleven have no Verifpal counterpart: one ablates a protocol feature to isolate how much depends on it,
three run the same protocol under concurrent sessions, one isolates the symmetric ratchet over two sequential
messages, and one is a historical pre-fix variant kept only for comparison against its fixed counterpart.

Written and checked against ProVerif 2.05 (`opam install proverif`; see the
<a href="https://bblanche.gitlabpages.inria.fr/proverif/install.html">official guide</a> for other ways). Run and
archive a model with:
```
proverif file_name.pv > outputs/file_name.txt
```
or regenerate every output at once:
```
for f in *.pv; do proverif "$f" > "outputs/${f%.pv}.txt"; done
```
Nine of the eleven finish in under a second; the two that declare Diffie-Hellman commutativity are slower, and by very
different margins — about six seconds for `01_pairwise_session_no_opk.pv` against roughly eight minutes for
`01_pairwise_session.pv`, which differ only in the fourth Diffie-Hellman exchange.

Not every model is expected to pass, and the failures don't all mean the same thing: some are the result being
measured, one marks a limit of the tool, others are replays left open on purpose. See
[FormalVerification/ProVerifModels/README.md](./FormalVerification/ProVerifModels/README.md) for the expected result of
each model and what its verdict means.

## License

Copyright © 2018-2024 <a href="https://www.sticknet.org">Sticknet</a>

Licensed under the GPLv3: http://www.gnu.org/licenses/gpl-3.0.html

See [NOTICE.md](./NOTICE.md) for attribution of contributed work.
