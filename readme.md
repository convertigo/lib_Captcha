


# lib_Captcha

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


For more technical informations : [documentation](./project.md)

- [Installation](#installation)
- [Mobile Library](#mobile-library)
    - [Shared Components](#shared-components)
        - [AltchaWidget](#altchawidget)


## Installation

1. In your Convertigo Studio click on ![](https://github.com/convertigo/convertigo/blob/develop/eclipse-plugin-studio/icons/studio/project_import.gif?raw=true "Import a project in treeview") to import a project in the treeview
2. In the import wizard

   ![](https://github.com/convertigo/convertigo/blob/develop/eclipse-plugin-studio/tomcat/webapps/convertigo/templates/ftl/project_import_wzd.png?raw=true "Import Project")
   
   paste the text below into the `Project remote URL` field:
   <table>
     <tr><td>Usage</td><td>Click the copy button at the end of the line</td></tr>
     <tr><td>To contribute</td><td>

     ```
     lib_Captcha=/Users/opic/runtime-New84/lib_Captcha/.git:branch=master
     ```
     </td></tr>
     <tr><td>To simply use</td><td>

     ```
     lib_Captcha=/Users/opic/runtime-New84/lib_Captcha//archive/master.zip
     ```
     </td></tr>
    </table>
3. Click the `Finish` button. This will automatically import the __lib_Captcha__ project


## Mobile Library

Describes the mobile application global properties

### Shared Components

#### AltchaWidget

Reusable ALTCHA client widget for Convertigo NGX applications. Wraps the official `altcha` Web Component and exposes configurable challenge, presentation, localization, automation, and runtime options. The `Verified`, `StateChanged`, and `Expired` outputs allow host applications to react to widget events.

**variables**

<table>
<tr>
<th>name</th><th>comment</th>
</tr>
<tr>
<td>action</td><td>Optional action bound to the challenge. The value is appended to the challenge URL and must match expectedAction during server-side verification.</td>
</tr>
<tr>
<td>auto</td><td>Automatic verification trigger: 'off', 'onfocus', 'onload', or 'onsubmit'.</td>
</tr>
<tr>
<td>challengeUrl</td><td>Optional absolute challenge URL. When empty, the component derives the portable lib_Captcha sequence URL from the public Convertigo SDK endpoint.</td>
</tr>
<tr>
<td>configuration</td><td>Additional ALTCHA configuration object. Explicit component variables override properties with the same name.</td>
</tr>
<tr>
<td>debug</td><td>Enables ALTCHA client-side debug logging when true.</td>
</tr>
<tr>
<td>display</td><td>Widget layout: 'standard', 'bar', 'floating', 'overlay', or 'invisible'.</td>
</tr>
<tr>
<td>hideFooter</td><td>Hides the ALTCHA attribution footer when true.</td>
</tr>
<tr>
<td>hideLogo</td><td>Hides the ALTCHA logo when true.</td>
</tr>
<tr>
<td>language</td><td>ISO language code used by the widget. English is bundled by default.</td>
</tr>
<tr>
<td>minDuration</td><td>Minimum verification duration in milliseconds.</td>
</tr>
<tr>
<td>name</td><td>Name of the hidden form field that receives the verified ALTCHA payload.</td>
</tr>
<tr>
<td>theme</td><td>Widget theme name, normally 'default' or 'dark'.</td>
</tr>
<tr>
<td>timeout</td><td>Maximum verification duration in milliseconds.</td>
</tr>
<tr>
<td>type</td><td>Interaction style: 'checkbox', 'switch', or 'native'.</td>
</tr>
<tr>
<td>validationMessage</td><td>HTML5 validation message shown while verification is incomplete.</td>
</tr>
<tr>
<td>workers</td><td>Number of Web Workers used for proof-of-work computation.</td>
</tr>
</table>

**events**

<table>
<tr>
<th>name</th><th>comment</th>
</tr>
<tr>
<td>Expired</td><td>Emits the ALTCHA expired event detail when the current challenge expires.</td>
</tr>
<tr>
<td>StateChanged</td><td>Emits the ALTCHA statechange event detail whenever the widget state changes.</td>
</tr>
<tr>
<td>Verified</td><td>Emits the ALTCHA verified event detail, including the generated payload.</td>
</tr>
</table>



