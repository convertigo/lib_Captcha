
# ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/core/images/project_color_16x16.png?raw=true "Project") lib_Captcha

# lib_Captcha

Self-hosted Convertigo library for creating and verifying **ALTCHA v2** challenges using the official Java implementation.

## Sequences

- `altcha_createChallenge`: creates a signed challenge for the ALTCHA widget.
- `altcha_verifyPayload`: verifies the payload, expected action, and replay status.

## Global symbols

| Symbol | Required | Default | Description |
| --- | --- | --- | --- |
| `lib_Captcha.altcha.hmac.secret` | Yes in production | empty | Secret used to sign challenges. Use a random value containing at least 16 characters. |
| `lib_Captcha.altcha.key.secret` | Yes in production | empty | Independent secret used to sign derived keys. Use a random value containing at least 16 characters. |
| `lib_Captcha.altcha.algorithm` | No | `PBKDF2/SHA-256` | ALTCHA algorithm used for the challenge. |
| `lib_Captcha.altcha.cost` | No | `5000` | PBKDF2 derivation cost. |
| `lib_Captcha.altcha.counter.min` | No | `5000` | Minimum value of the random counter. |
| `lib_Captcha.altcha.counter.max` | No | `10000` | Exclusive maximum value of the random counter. |
| `lib_Captcha.altcha.expires.seconds` | No | `300` | Challenge validity period in seconds. |

Symbols ending in `.secret` are masked by Convertigo. When the two secrets are not configured, the library generates temporary values suitable for development only. These values are lost when the engine restarts and are not shared between cluster nodes.

Symbols are resolved directly in the JavaScript steps using Convertigo's native symbol substitution syntax.

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
