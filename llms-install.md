# Installing the AdMakeAI MCP server

Instructions for AI agents (Cline and others) setting this server up for a user.

AdMakeAI's MCP server is hosted. There is nothing to clone, install, build or run: you add one remote server entry.

- **URL:** `https://admakeai.com/api/mcp`
- **Transport:** Streamable HTTP
- **Auth:** an AdMakeAI API key in a header, or OAuth 2.1 sign-in in the browser

## Before you start

The user needs an AdMakeAI account. Signup is free at https://admakeai.com/register (30 free credits, no card). Ask the user to sign up themselves; don't create an account for them.

## Option A: API key

1. Ask the user to create an API key at https://admakeai.com/dashboard/integrations/api and paste it to you. Keys start with `amai_live_`. Never guess or invent a key.
2. Add this to the MCP settings file (`cline_mcp_settings.json` in Cline), with the user's key:

```json
{
  "mcpServers": {
    "admakeai": {
      "type": "streamableHttp",
      "url": "https://admakeai.com/api/mcp",
      "headers": {
        "x-api-key": "amai_live_..."
      },
      "disabled": false,
      "autoApprove": []
    }
  }
}
```

`"Authorization": "Bearer amai_live_..."` works the same as `x-api-key`. An API key sees every tool.

## Option B: OAuth, no key

In a client that supports MCP OAuth for remote servers, leave the headers out:

```json
{
  "mcpServers": {
    "admakeai": {
      "type": "streamableHttp",
      "url": "https://admakeai.com/api/mcp",
      "disabled": false,
      "autoApprove": []
    }
  }
}
```

The first request answers `401` with a `WWW-Authenticate` challenge that points to `https://admakeai.com/.well-known/oauth-protected-resource/api/mcp`. The client then registers itself (RFC 7591 dynamic client registration; `http` redirect URIs on a loopback address are accepted), opens AdMakeAI's sign-in and consent page in the browser, and receives a scoped token (authorization code with PKCE S256). In Cline, run the server's OAuth authorization if it doesn't start on its own. If the client can't do OAuth, use Option A.

The consent page grants three scopes, and tools outside the granted scopes are left out of `tools/list`:

| Scope | Covers |
| --- | --- |
| `mcp:read` | Profile, credits, projects, generated ads, competitor research results, Meta ad performance |
| `mcp:write` | Image and video generation, ad copy, ad sets, uploads, competitor scrapes, campaign drafts (spends credits) |
| `mcp:publish` | Creating real ads and draft campaigns in the user's connected Meta ad account |

Claude and ChatGPT custom connectors use the same URL with OAuth.

## Leave autoApprove empty

Several tools spend credits, and three create ads or draft campaigns in the user's real Meta ad account. Tool descriptions start with `[write]` for tools that change data and `[DANGER: live action]` for those three; confirm with the user before calling them. There are no delete tools.

## Check that it works

Call `user__getCredits`. It returns the account's credit balance. A `401` means the key is missing, malformed or revoked, or the OAuth token expired.

## First use

- Start with `adResearch__listProjects`. Every generation belongs to a project (a brand). If the user has several, pick the one their request is about, or ask.
- If the list is empty, the account is new: ask the user for their brand's website (never guess it) and call `project__createFirst`. It's free and only works for the first project; later projects are made at https://admakeai.com/dashboard.
- Generations are queued. Poll `adGeneration__getById` or `videoGeneration__getStatus` until the status is `COMPLETED`.
- A standard image ad costs 2 credits (8 high quality, 20 ultra); a UGC video 5, 15 or 50 credits. Video ads and the Facebook tools need a paid plan (https://admakeai.com/pricing), and the Facebook tools need a Facebook account connected at https://admakeai.com/dashboard/integrations/facebook.

## Tools (60)

| Group | Tools |
| --- | --- |
| Account | `user__me`, `user__getCredits` |
| Projects | `adResearch__listProjects`, `project__createFirst`, `adResearch__getProject`, `adResearch__pollStatus`, `project__updateBrief`, `project__getAssets` |
| Uploads | `upload__uploadImageBase64`, `upload__getUserImages` |
| Ad images | `adGeneration__create` (a finished ad with headline, CTA and logo from the brand brief), `adGeneration__generateImage` (an image from a verbatim prompt), `adGeneration__edit`, `adGeneration__getById`, `adGeneration__list`, `adGeneration__getRecent`, `adGeneration__searchAll`, `adGeneration__listPending`, `adGeneration__toggleFavorite` |
| Ad copy | `adCopyGeneration__generate`, `adCopyGeneration__getMessagingStyles` |
| Ad sets | `adSet__list`, `adSet__get`, `adSet__create`, `adSet__update`, `adSet__getGenerations`, `adSet__generateBatch`, `adSet__suggestDirections` |
| UGC video ads | `videoGeneration__start`, `videoGeneration__getStatus`, `videoGeneration__list`, `videoGeneration__listPending` |
| Competitor research (Meta Ad Library) | `adResearch__searchCompanies`, `adResearch__getCompanyAds`, `adResearch__addCompetitors`, `adResearch__scrapeCompetitor`, `adResearch__getCompetitorPreview`, `adResearch__getScrapedAds`, `adResearch__getAdDetails`, `adResearch__saveInspiration`, `adResearch__getSavedInspirations` |
| Meta uploads and drafts | `facebookConnection__list`, `facebookUpload__getCampaigns`, `facebookUpload__getAdSets`, `facebookUpload__runPreflight`, `facebookUpload__request`, `facebookUpload__list`, `facebookUpload__get`, `facebookCampaignDraft__sourceAssets`, `facebookCampaignDraft__getAgentDraft`, `facebookCampaignDraft__pushAgentDraft`, `facebookCampaignDraft__pushDraftCampaign` |
| Meta ad performance | `zernioAnalytics__accountOverview`, `zernioAnalytics__tree`, `zernioAnalytics__timeline`, `zernioAnalytics__listCampaigns`, `zernioAnalytics__listAds`, `zernioAnalytics__getAd`, `zernioAnalytics__getAdAnalytics`, `zernioAnalytics__getAdComments` |

The live ones are `facebookUpload__request`, `facebookCampaignDraft__pushAgentDraft` and `facebookCampaignDraft__pushDraftCampaign`. Pausing, resuming and launching ads is done in Meta Ads Manager, not through this server.

## Links

- Agents, MCP and REST overview: https://admakeai.com/agents
- API docs: https://admakeai.com/api/docs
- Pricing: https://admakeai.com/pricing
- Support: support@admakeai.com
