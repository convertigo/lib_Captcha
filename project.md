
# ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/core/images/project_color_16x16.png?raw=true "Project") lib_Captcha

Self-hosted Convertigo library for creating and verifying **ALTCHA v2** proof-of-work challenges with the official Java implementation and reusable NGX widget.

## Why ALTCHA

ALTCHA is a strong privacy-friendly alternative to SaaS CAPTCHA services. Once this project is deployed, challenge creation, browser proof-of-work, payload verification, action binding, expiration checks, and replay detection are handled entirely by the Convertigo application and the browser.

No request is sent to an external CAPTCHA provider at runtime. There is no third-party CAPTCHA account, API key, remote scoring service, tracking cookie, or external availability dependency. The solution is therefore **100% autonomous and self-hosted at runtime**. The official ALTCHA client is bundled with the NGX application and the official Java library runs inside the Convertigo Engine.

## How the proof of work operates

1. `altcha_createChallenge` selects a cryptographically random counter in the configured range, embeds optional business data such as an action, adds an expiration time, and signs the challenge with HMAC.
2. The ALTCHA widget solves the challenge locally in the browser. It tries counters and derives a key for each candidate until it finds the expected signed key prefix.
3. The browser submits the resulting Base64 payload with the business form.
4. The business sequence calls the private `altcha_verifyPayload` sequence. The server verifies the challenge signature, solution, expiration, expected action, and replay status before accepting the operation.
5. With the independent key-signature secret used by this project, successful deterministic solutions can be verified efficiently without repeating the complete browser workload on the server.

The client-side “Verified” state is only a user-interface signal. A business operation must always send the payload to `altcha_verifyPayload` or to a server sequence such as `altcha_demoSubmit` that calls it internally.

## Cost and counter tuning

With the default `PBKDF2/SHA-256` algorithm, `lib_Captcha.altcha.cost` is the PBKDF2 iteration count used for **each candidate counter**. The default cost is `5000`.

The counter range controls how many candidate values the browser is likely to test. This project chooses a random target counter in the half-open interval:

```text
[counter.min, counter.max)
```

With the defaults `5000` and `10000`, the browser typically performs between approximately 5,000 and 10,000 candidate derivations. The effective client workload is therefore influenced by both:

- the KDF cost per candidate;
- the number of candidate counters that must be tested.

Increasing `cost`, `counter.min`, or `counter.max` increases resistance to bulk automated solving, but it also increases CPU usage, battery consumption, and verification latency on legitimate client devices. More Web Workers can reduce wall-clock time by distributing the search, but they do not remove the total computational work. Tune these values using representative mobile and desktop devices rather than increasing them blindly.

For PBKDF2, the rough work factor is proportional to:

```text
PBKDF2 iterations per candidate × number of attempted counters
```

The configured limits are validated when a challenge is created. The defaults provide a development-friendly baseline and should be performance-tested for each production audience.

## Limits of human verification

ALTCHA is a proof-of-work CAPTCHA, not a proof of human identity. A successful solution proves that a client received an authentic challenge, performed the required computation, and returned a valid result before expiration. It does **not** prove that the client is a human, that one solution corresponds to one physical person, or that the submitted business data are trustworthy.

Automated software can still:

- request challenges from the public challenge endpoint;
- run the same proof-of-work algorithm in a headless browser or custom client;
- use additional CPU resources or parallel workers;
- pay humans or external solving farms to complete challenges;
- create many independent valid challenges unless other controls limit the request rate.

The `cost` and counter range increase the economic and computational price of automation; they do not make automation impossible. Replay protection prevents reuse of the **same** accepted payload, but it does not prevent a bot from solving a fresh challenge for every request.

The checkbox and the client-side “Verified” state are user-experience elements, not a security boundary. Only server-side verification is authoritative. The self-hosted ALTCHA mode used by this library does not provide behavioral risk scoring, device reputation, identity verification, or a guarantee that a mouse click came from a human.

For sensitive operations, use ALTCHA as one layer in a broader abuse-prevention strategy:

- rate-limit challenge creation and protected business endpoints;
- bind every challenge to the intended business `action`;
- require authentication, email confirmation, or multi-factor authentication when appropriate;
- validate all business fields and apply domain-specific fraud rules;
- add privacy-respecting honeypots, velocity controls, quotas, or anomaly detection;
- monitor rejection, replay, and request-volume patterns;
- increase friction progressively instead of imposing an excessive proof-of-work cost on every legitimate user.

This limitation is not specific to ALTCHA: no checkbox CAPTCHA can mathematically prove humanity. ALTCHA is valuable because it provides a transparent, privacy-friendly and autonomous way to raise the cost of automated abuse without depending on a third-party SaaS service.

## Replay protection

A valid proof-of-work payload remains cryptographically valid until its challenge expires. Without replay protection, a bot could capture one valid payload and submit it repeatedly during that validity period.

`altcha_verifyPayload` enables replay protection by default through `preventReplay=true`. After all cryptographic, expiration, and action checks succeed, the verifier atomically records the signed challenge identifier as consumed. The first submission is accepted; later submissions using the same payload are rejected with:

```json
{
  "verified": false,
  "replayed": true,
  "error": "replayed"
}
```

Consumed identifiers are retained until the challenge expiration time and expired entries are cleaned from the registry. Recording happens only after successful verification, so malformed or invalid submissions cannot consume a legitimate challenge.

The built-in replay registry is stored in a thread-safe, node-local in-memory map and is bounded to protect the engine. This is fully autonomous for a single Convertigo Engine instance. In a multi-node cluster, use a shared atomic store such as Redis or a database, or enforce node affinity, so that the same payload cannot be accepted once by each node. Restarting an engine clears its in-memory replay registry, so short challenge expiration remains important.

Replay protection complements, but does not replace:

- short challenge expiration;
- binding challenges to an `action`;
- rate limiting on public challenge and business endpoints;
- server-side validation of the complete business request.

## Sequences

- `altcha_createChallenge`: public challenge endpoint used by the widget.
- `altcha_verifyPayload`: private server-side verifier for signature, solution, expiration, expected action, and replay status.
- `altcha_demoSubmit`: demonstration business endpoint that calls the private verifier and returns a stable business response.

## Global symbols

| Symbol | Required | Default | Description |
| --- | --- | --- | --- |
| `lib_Captcha.altcha.hmac.secret` | Yes in production | empty | Secret used to sign challenges. Use a random value containing at least 16 characters. |
| `lib_Captcha.altcha.key.secret` | Yes in production | empty | Independent secret used to sign derived keys and enable efficient deterministic verification. Use a random value containing at least 16 characters. |
| `lib_Captcha.altcha.algorithm` | No | `PBKDF2/SHA-256` | Key-derivation algorithm used by the challenge. |
| `lib_Captcha.altcha.cost` | No | `5000` | Algorithm-specific cost; for PBKDF2 this is the iteration count for every candidate counter. |
| `lib_Captcha.altcha.counter.min` | No | `5000` | Inclusive minimum random target counter. |
| `lib_Captcha.altcha.counter.max` | No | `10000` | Exclusive maximum random target counter. |
| `lib_Captcha.altcha.expires.seconds` | No | `300` | Challenge validity period in seconds. |

Symbols ending in `.secret` are masked by Convertigo. When the two secrets are not configured, the library generates temporary values suitable for development only. These values are lost when the engine restarts and are not shared between cluster nodes. Configure stable secrets in every production environment and use identical secrets on every node that must validate the same challenges.

Symbols are resolved directly in the JavaScript steps using Convertigo native symbol substitution syntax.


<details><summary><span style="color:DarkGoldenRod"><i>Connectors</i></span></summary><blockquote><p>


## ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/connectors/images/sqlconnector_color_16x16.png?raw=true "SqlConnector") void

Placeholder SQL connector for template projects

<details><summary><span style="color:DarkGoldenRod"><i>Transactions</i></span></summary><blockquote><p>


### ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/transactions/images/sqltransaction_color_16x16.png?raw=true "SqlTransaction") void

Placeholder transaction that intentionally cancels execution
</p></blockquote></details>
</p></blockquote></details>

<details><summary><span style="color:DarkGoldenRod"><i>Sequences</i></span></summary><blockquote><p>


<details><summary><b>altcha_createChallenge</b> : Creates a fresh signed ALTCHA v2 challenge with the official Java library called directly from the sequence</summary><blockquote><p>


## ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/sequences/images/genericsequence_color_16x16.png?raw=true "GenericSequence") altcha_createChallenge

Creates a fresh signed ALTCHA v2 challenge with the official Java library called directly from the sequence.

<span style="color:DarkGoldenRod">Variables</span>

<table>
<tr>
<th>
name
</th>
<th>
comment
</th>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;action
</td>
<td>
Optional business action bound to the signed challenge, for example `signup` or `password-reset`. Pass the same value as `expectedAction` during verification. Maximum length: 128 characters.
</td>
</tr>
</table>

</p></blockquote></details>

<details><summary><b>altcha_demoSubmit</b> : Demo form submission endpoint</summary><blockquote><p>


## ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/sequences/images/genericsequence_color_16x16.png?raw=true "GenericSequence") altcha_demoSubmit

Demo form submission endpoint. Receives business form data and delegates CAPTCHA validation to the private altcha_verifyPayload sequence. The submission is accepted only for the captcha-demo action with replay prevention enabled.

<span style="color:DarkGoldenRod">Variables</span>

<table>
<tr>
<th>
name
</th>
<th>
comment
</th>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;email
</td>
<td>
Required email address submitted by the demonstration form.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;fullName
</td>
<td>
Required display name submitted by the demonstration form.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;payload
</td>
<td>
Verified ALTCHA payload emitted by the client widget and revalidated on the server.
</td>
</tr>
</table>

</p></blockquote></details>

<details><summary><b>altcha_verifyPayload</b> : Privately verifies ALTCHA v2 payloads, expected business action, and replay</summary><blockquote><p>


## ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/sequences/images/genericsequence_color_16x16.png?raw=true "GenericSequence") altcha_verifyPayload

Privately verifies ALTCHA v2 payloads, expected business action, and replay.

<span style="color:DarkGoldenRod">Variables</span>

<table>
<tr>
<th>
name
</th>
<th>
comment
</th>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;expectedAction
</td>
<td>
Optional business action expected in the signed challenge data. When provided, verification fails with `action_mismatch` unless it exactly matches the `action` used to create the challenge.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;payload
</td>
<td>
Required Base64-encoded ALTCHA payload returned by the client widget after solving the challenge. The maximum accepted length is 65,536 characters.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;preventReplay
</td>
<td>
Controls single-use protection for valid challenges. Defaults to `true`; when enabled, a second verification of the same challenge is rejected with the `replayed` error.
</td>
</tr>
</table>

</p></blockquote></details>
</p></blockquote></details>

<details><summary><span style="color:DarkGoldenRod"><i>Mobile Application</i></span></summary><blockquote><p>


## ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/core/images/mobileapplication_color_16x16.png?raw=true "MobileApplication") Application

Describes the mobile application global properties

<details><summary><span style="color:DarkGoldenRod"><i>Pages</i></span></summary><blockquote><p>


### ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/ngx/components/images/pagecomponent_color_16x16.png?raw=true "PageComponent") Captcha

Palette-first demonstration page for the reusable AltchaWidget SharedComponent.
</p></blockquote></details>

<details><summary><span style="color:DarkGoldenRod"><i>Shared Components</i></span></summary><blockquote><p>


### ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/ngx/components/images/uisharedcomponent_16x16.png?raw=true "UISharedRegularComponent") AltchaWidget

Reusable ALTCHA client widget for Convertigo NGX applications. Wraps the official `altcha` Web Component and exposes configurable challenge, presentation, localization, automation, and runtime options. The `Verified`, `StateChanged`, and `Expired` outputs allow host applications to react to widget events.

<span style="color:DarkGoldenRod">Variables</span>

<table>
<tr>
<th>
name
</th>
<th>
comment
</th>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/ngx/components/images/uicompvariable_16x16.png?raw=true "  alt="UICompVariable" >&nbsp;action
</td>
<td>
Optional action bound to the challenge. The value is appended to the challenge URL and must match expectedAction during server-side verification.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/ngx/components/images/uicompvariable_16x16.png?raw=true "  alt="UICompVariable" >&nbsp;auto
</td>
<td>
Automatic verification trigger: 'off', 'onfocus', 'onload', or 'onsubmit'.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/ngx/components/images/uicompvariable_16x16.png?raw=true "  alt="UICompVariable" >&nbsp;challengeUrl
</td>
<td>
Optional absolute challenge URL. When empty, the component derives the portable lib_Captcha sequence URL from the public Convertigo SDK endpoint.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/ngx/components/images/uicompvariable_16x16.png?raw=true "  alt="UICompVariable" >&nbsp;configuration
</td>
<td>
Additional ALTCHA configuration object. Explicit component variables override properties with the same name.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/ngx/components/images/uicompvariable_16x16.png?raw=true "  alt="UICompVariable" >&nbsp;debug
</td>
<td>
Enables ALTCHA client-side debug logging when true.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/ngx/components/images/uicompvariable_16x16.png?raw=true "  alt="UICompVariable" >&nbsp;display
</td>
<td>
Widget layout: 'standard', 'bar', 'floating', 'overlay', or 'invisible'.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/ngx/components/images/uicompvariable_16x16.png?raw=true "  alt="UICompVariable" >&nbsp;hideFooter
</td>
<td>
Hides the ALTCHA attribution footer when true.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/ngx/components/images/uicompvariable_16x16.png?raw=true "  alt="UICompVariable" >&nbsp;hideLogo
</td>
<td>
Hides the ALTCHA logo when true.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/ngx/components/images/uicompvariable_16x16.png?raw=true "  alt="UICompVariable" >&nbsp;language
</td>
<td>
ISO language code used by the widget. English is bundled by default.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/ngx/components/images/uicompvariable_16x16.png?raw=true "  alt="UICompVariable" >&nbsp;minDuration
</td>
<td>
Minimum verification duration in milliseconds.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/ngx/components/images/uicompvariable_16x16.png?raw=true "  alt="UICompVariable" >&nbsp;name
</td>
<td>
Name of the hidden form field that receives the verified ALTCHA payload.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/ngx/components/images/uicompvariable_16x16.png?raw=true "  alt="UICompVariable" >&nbsp;theme
</td>
<td>
Widget theme name, normally 'default' or 'dark'.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/ngx/components/images/uicompvariable_16x16.png?raw=true "  alt="UICompVariable" >&nbsp;timeout
</td>
<td>
Maximum verification duration in milliseconds.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/ngx/components/images/uicompvariable_16x16.png?raw=true "  alt="UICompVariable" >&nbsp;type
</td>
<td>
Interaction style: 'checkbox', 'switch', or 'native'.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/ngx/components/images/uicompvariable_16x16.png?raw=true "  alt="UICompVariable" >&nbsp;validationMessage
</td>
<td>
HTML5 validation message shown while verification is incomplete.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/ngx/components/images/uicompvariable_16x16.png?raw=true "  alt="UICompVariable" >&nbsp;workers
</td>
<td>
Number of Web Workers used for proof-of-work computation.
</td>
</tr>
</table>


<span style="color:DarkGoldenRod">Events</span>

<table>
<tr>
<th>
name
</th>
<th>
comment
</th>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/ngx/components/images/uicompevent_16x16.png?raw=true "  alt="UICompEvent" >&nbsp;Expired
</td>
<td>
Emits the ALTCHA expired event detail when the current challenge expires.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/ngx/components/images/uicompevent_16x16.png?raw=true "  alt="UICompEvent" >&nbsp;StateChanged
</td>
<td>
Emits the ALTCHA statechange event detail whenever the widget state changes.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/ngx/components/images/uicompevent_16x16.png?raw=true "  alt="UICompEvent" >&nbsp;Verified
</td>
<td>
Emits the ALTCHA verified event detail, including the generated payload.
</td>
</tr>
</table>

</p></blockquote></details>
</p></blockquote></details>
