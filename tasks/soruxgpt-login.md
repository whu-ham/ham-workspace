# SoruxGPT login

Add SoruxGPT as a third-party OAuth2 login provider, alongside GitHub and ZiQiang, on every
client and on the backend.

Workspace issue: whu-ham/ham-workspace#16.

## Flow

The client opens the SoruxGPT authorize URL. The provider redirects back with a `code`. The
client sends that code to the backend, which exchanges it for a profile and issues a Ham
session. Users can also bind and unbind SoruxGPT from the social-account screen, exactly as
they bind GitHub today.

```
client ──open authorize URL──▶ SoruxGPT
       ◀──redirect ham://…?code=…──┘
       ──LoginRequest{login_type=SORUXGPT, data=LoginSoruxGPTData{code}}──▶ backend
       ◀──token + user────────────────────────────────────────────────────┘
```

## Contract

`ham-proto` gains:

```proto
enum LoginOpenType {
  … LOGIN_OPEN_TYPE_ZIQIANG = 5;
  LOGIN_OPEN_TYPE_SORUXGPT = 6;
  LOGIN_OPEN_TYPE_PASSKEY = 300;
}

message LoginSoruxGPTData {
  string code = 1;
}
```

Additive only — no field renumbering, no removals. Consumers pin the proto by `PROTO_VERSION`,
so the contract lands first and each repo then bumps its pin.

## Icon

Black background with a white "S". Inverted in dark mode: white background, black "S". No
gradient, no brand colour — the mark is a single letter on a flat fill.

## Credentials

A public native-app client id with no secret. Clients that ship a binary may embed it; clients
that cannot keep it private (the web client, which runs in a browser and on Cloudflare) read it
from config instead.

## Order of work

1. `ham-proto` — contract. Auto-tags on merge; everything below waits for the tag.
2. `ham-backend-go` — code exchange, login case, social-account binding, and the web callback
   `POST /web/auth/oauth/soruxgpt/callback`.
3. Clients — independent of each other once the tag exists.

## Per-repo work

| Submodule | Work | Issue |
| --- | --- | --- |
| `repos/ham-proto` | Enum value, message, `Any`-payload comments | whu-ham/ham-proto#9 |
| `repos/ham-backend-go` | `SoruxGPTClient`, `LoginSoruxGPT`, `InsertSoruxGPT`, both handler switches, config block, web callback endpoint | whu-ham/ham-backend-go#94 |
| `repos/ham-android` | Login sheet button, social-account row, `PROTO_VERSION` bump | whu-ham/ham-android#127 |
| `repos/ham-ios` | Native SwiftUI sheet + row, CCKV URL key, imageset, `project.pbxproj`, `PROTO_VERSION` bump | whu-ham/ham-ios#148 |
| `repos/ham-web` | Provider registry entry, icon assets, i18n labels, `wrangler.jsonc` var, grid width | whu-ham/ham-web#104 |

## Visibility

Android and iOS gate providers on the `ham_validLoginType` cloud-config list; the web client
hardcodes its registry. The buttons stay hidden until `"soruxgpt"` is added to that list, so
shipping the client code is safe on its own.

## Out of scope

- SoruxGPT as a *course* or *schedule* data source — login only.
- Unbinding UI in the web client, which has no social-account screen today.
