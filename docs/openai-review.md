# Botdoor: OpenAI plugin review notes

## Tool annotations (served live in `tools/list`)

Definitions used: **readOnlyHint** = changes nothing. **destructiveHint** = deletes or overwrites something that already exists. **idempotentHint** = repeating the same call has no further effect. **openWorldHint** = reaches outside Botdoor (publishes to a social network or sends an email).

| Tool | readOnly | destructive | idempotent | openWorld | Why |
|---|---|---|---|---|---|
| list_accounts | true | false | true | false | Reads connected accounts |
| get_connect_url | true | false | true | false | Returns a link; the person connects in the browser |
| get_post | true | false | true | false | Reads one post |
| list_posts | true | false | true | false | Reads posts |
| upload_media | false | false | false | false | Stores a new file in Botdoor only |
| create_post | false | false | false | true | Creates a new post that can publish (asks-first keys wait for approval); nothing existing is changed |
| approve_post | false | false | false | true | Publishes; refuses API keys (people only) |
| new_claim_link | false | false | false | true | Emails the owner a new claim link; the old link stops working |
| retry_first_comment | false | false | true | true | Posts a failed first comment; never twice |
| reject_post | false | true | true | false | Permanently discards a waiting post |
| cancel_post | false | true | true | true | Cancels a scheduled post (also at the network) |
| reschedule_post | false | true | true | true | Overwrites the scheduled time |
| signup (keyless only) | false | false | false | true | Creates a workspace and emails a claim link |

## Blocker before submission: authentication

Botdoor authenticates with a static `Authorization: Bearer bd_...` key. `mcp.json` declares `extensions["com.openai"].auth = { type: "api_key", headerScheme: "bearer" }`, which the schema accepts, but the ChatGPT app submission portal only offers **No auth**, **OAuth** or **Mixed**. A server that needs a key must not be declared as "No auth", and fixed headers are rejected. To list in ChatGPT, Botdoor needs OAuth 2.1 (`/.well-known/oauth-protected-resource`, authorization server metadata, PKCE S256, dynamic client registration or CIMD). Botdoor does not offer OAuth yet.

## Also needed before submission

- An OpenAI platform organisation with business verification, and the domain challenge served at `https://botdoor.co/.well-known/openai-apps-challenge`.
- A reviewer test workspace with connected test accounts and no MFA.
- A demo video (set `review.demo_recording_url`, then rebuild the ZIP).
