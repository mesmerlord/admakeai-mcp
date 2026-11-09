# AdMakeAI MCP Server

[AdMakeAI](https://admakeai.com) designs on-brand ad images, UGC-style video ads and ad copy, and looks up the ads any brand runs in the Meta Ad Library. This is its hosted MCP server.

- **URL:** `https://admakeai.com/api/mcp`
- **Transport:** Streamable HTTP
- **Auth:** an AdMakeAI API key, or OAuth 2.1 sign-in in the browser

The server is hosted, so there's nothing to clone, install or build. Agents setting it up for a user can follow [llms-install.md](llms-install.md).

## Authentication

**API key.** Create one in the AdMakeAI dashboard under **API & Agents → API keys** ([admakeai.com/dashboard/integrations/api](https://admakeai.com/dashboard/integrations/api)). Keys start with `amai_live_`. Send it as `x-api-key: <key>` or `Authorization: Bearer <key>`. A key sees all 60 tools.

**OAuth, no key.** Clients that support MCP OAuth (Claude, ChatGPT, Claude Code, Cursor, Codex) can connect with the URL alone. The first request answers `401` with a `WWW-Authenticate` challenge pointing to `https://admakeai.com/.well-known/oauth-protected-resource/api/mcp`. The client registers itself (RFC 7591 dynamic client registration, or a client ID metadata document), the user signs in and approves on admakeai.com, and the client gets a token through the authorization code flow with PKCE S256. Access tokens last an hour and refresh with rotation. Connected apps are listed and revocable on the same dashboard page.

The consent screen grants three scopes, and tools outside the granted scopes are left out of `tools/list`:

| Scope | Consent label | Covers |
| --- | --- | --- |
| `mcp:read` | Read your account | Profile, credits, projects, generated ads, competitor research, Meta ad performance |
| `mcp:write` | Create ads and drafts | Image and video generation, ad copy, ad sets, uploads, competitor tracking (spends credits) |
| `mcp:publish` | Upload ads and campaign drafts to Meta | The three tools that create ads or draft campaigns in the user's connected Meta ad account |

## Install

**Claude Code.** The [plugin](https://github.com/mesmerlord/admakeai-claude-plugin) adds the server and a skill that knows the tools:

```bash
claude plugin marketplace add mesmerlord/admakeai-claude-plugin && claude plugin install admakeai@admakeai
```

Or add the server alone, then run `/mcp`, pick `admakeai` and choose **Authenticate**:

```bash
claude mcp add --transport http admakeai https://admakeai.com/api/mcp
```

With a key instead: append `--header "x-api-key: amai_live_..."`. Before sign-in, Claude Code can already call `adResearch__searchCompanies` and `adResearch__getCompanyAds`; every other tool answers with how to sign in.

**Claude Desktop and claude.ai.** Settings → Connectors → **Add custom connector**, paste `https://admakeai.com/api/mcp`, and sign in when asked. No client ID or secret. ChatGPT custom connectors take the same URL.

**Cursor.** In `~/.cursor/mcp.json` (or Settings → Tools & MCP), then sign in when Cursor asks:

```json
{ "mcpServers": { "admakeai": { "url": "https://admakeai.com/api/mcp" } } }
```

With a key instead, add `"headers": { "x-api-key": "amai_live_..." }` next to `url`.

**Cline.** See [llms-install.md](llms-install.md). In `cline_mcp_settings.json`:

```json
{
  "mcpServers": {
    "admakeai": {
      "type": "streamableHttp",
      "url": "https://admakeai.com/api/mcp",
      "headers": { "x-api-key": "amai_live_..." },
      "disabled": false,
      "autoApprove": []
    }
  }
}
```

**Codex.**

```bash
codex mcp add admakeai --url https://admakeai.com/api/mcp && codex mcp login admakeai
```

## Tools (60)

Read tools are safe to call freely. Write tools change data in the user's AdMakeAI account, and some spend credits; their descriptions start with `[write]`. The three live tools create ads or draft campaigns in the user's real Meta ad account; theirs start with `[DANGER: live action]`, so confirm before calling them. There are no delete tools. Uploads and drafts land paused, and launching, pausing and resuming ads stay in Meta Ads Manager.

**Account (2)**

| Tool | Kind | What it does |
| --- | --- | --- |
| `user__me` | read | Profile, plan and credit balance |
| `user__getCredits` | read | Credit balance only |

**Projects (6).** Every generation belongs to a project: a brand, with its brief and logo.

| Tool | Kind | What it does |
| --- | --- | --- |
| `adResearch__listProjects` | read | The user's projects. Call it first |
| `project__createFirst` | write | Create the account's first project from the brand's website (brief and logo in about 10 to 15 s). Free, and only when there are no projects; later ones are made in the dashboard |
| `adResearch__getProject` | read | One project with its brand brief, competitors, research jobs and scraped-ad counts |
| `adResearch__pollStatus` | read | A project's research status: pipeline stage, running scrape jobs, competitor and saved-ad counts |
| `project__updateBrief` | write | Update the brand brief (product summary, audience, value proposition, industry, keywords) |
| `project__getAssets` | read | The project's stored product photos and logos, with URLs, for reference images |

**Uploads (2)**

| Tool | Kind | What it does |
| --- | --- | --- |
| `upload__uploadImageBase64` | write | Upload a JPEG, PNG or WebP image as base64; returns an image ID and public URL |
| `upload__getUserImages` | read | Previously uploaded images, newest first |

**Ad images (9)**

| Tool | Kind | What it does |
| --- | --- | --- |
| `adGeneration__create` | write | Queue a finished ad from a creative brief: headline, subline, CTA and logo from the brand brief. Costs credits |
| `adGeneration__generateImage` | write | Queue an image from a verbatim prompt, with no added text or logo. Costs credits |
| `adGeneration__edit` | write | Queue an image-to-image edit of one of the user's ads or any image URL (background, restyle, aspect ratio, remix). Costs credits |
| `adGeneration__getById` | read | Status, prompt and image URLs of one generation; poll until `COMPLETED` |
| `adGeneration__list` | read | The user's ad generations, newest first, paginated |
| `adGeneration__getRecent` | read | The most recent ad generations |
| `adGeneration__searchAll` | read | Full-text search over the user's generations by prompt |
| `adGeneration__listPending` | read | Generations still in progress |
| `adGeneration__toggleFavorite` | write | Star or unstar a generation. Free |

**Ad copy (2)**

| Tool | Kind | What it does |
| --- | --- | --- |
| `adCopyGeneration__generate` | write | Facebook and Instagram ad copy for an ad image (headline, primary text, description, CTA, hook angle), up to 5 variations, on the brand brief when given a project |
| `adCopyGeneration__getMessagingStyles` | read | The messaging styles `adCopyGeneration__generate` accepts |

**Ad sets (7).** One shared brief, ad type, quality and aspect ratio, many variations.

| Tool | Kind | What it does |
| --- | --- | --- |
| `adSet__list` | read | The user's ad sets, paginated |
| `adSet__get` | read | One ad set with its instructions |
| `adSet__create` | write | Create an ad set. Free; the batch spends the credits |
| `adSet__update` | write | Change an ad set's brief, ad type, quality, aspect ratio or reference images |
| `adSet__getGenerations` | read | Every generation in an ad set |
| `adSet__generateBatch` | write | Queue a batch of variations from an ad set and per-variation instructions. Costs credits |
| `adSet__suggestDirections` | write | Suggest distinct creative directions (angles, hooks) for an ad set |

**UGC video ads (4).** Need a paid plan.

| Tool | Kind | What it does |
| --- | --- | --- |
| `videoGeneration__start` | write | Queue a UGC-style video ad for a project: script, voice and render. Costs credits by quality |
| `videoGeneration__getStatus` | read | Status and output URL of one video; poll until `COMPLETED` |
| `videoGeneration__list` | read | Completed video ads, paginated |
| `videoGeneration__listPending` | read | Videos still rendering |

**Competitor research, from the Meta Ad Library (9)**

| Tool | Kind | What it does |
| --- | --- | --- |
| `adResearch__searchCompanies` | read | Resolve a brand name to its Facebook advertiser Page (id, name, category, followers) |
| `adResearch__getCompanyAds` | read | The ads any Facebook Page is running now |
| `adResearch__addCompetitors` | write | Add competitors to a project by name (optionally with their website) and start scraping their ads |
| `adResearch__scrapeCompetitor` | write | Re-scrape one competitor's live ads into the project; returns a research job |
| `adResearch__getCompetitorPreview` | read | The ads already scraped for one competitor |
| `adResearch__getScrapedAds` | read | Browse a project's scraped ads, sorted by relevance, longest running or newest, filtered by media type or keyword |
| `adResearch__getAdDetails` | read | One scraped ad in full: creatives, text, headline, run dates, placements |
| `adResearch__saveInspiration` | write | Save a scraped ad to a project as inspiration, to remix with `adGeneration__create` |
| `adResearch__getSavedInspirations` | read | A project's saved inspirations with their IDs |

**Meta uploads and campaign drafts (11).** Need a paid plan and a Facebook ad account connected in the dashboard.

| Tool | Kind | What it does |
| --- | --- | --- |
| `facebookConnection__list` | read | The user's connected Facebook ad accounts and Pages |
| `facebookUpload__getCampaigns` | read | Campaigns in a connected ad account |
| `facebookUpload__getAdSets` | read | Ad sets under a campaign |
| `facebookUpload__runPreflight` | read | Check an ad against an ad account for upload (image dimensions, copy length) without uploading |
| `facebookUpload__request` | live | Upload a generated ad to a Meta ad account, campaign and ad set, as a real (paused) ad |
| `facebookUpload__list` | read | Uploaded ads, filterable by ad account and state |
| `facebookUpload__get` | read | Upload state and Meta IDs of one uploaded ad |
| `facebookCampaignDraft__sourceAssets` | read | Turn previously uploaded ads into reusable creatives for a campaign draft |
| `facebookCampaignDraft__getAgentDraft` | read | A saved campaign draft, by ID, to review before pushing |
| `facebookCampaignDraft__pushAgentDraft` | live | Push a saved campaign draft to Meta as paused campaigns, ad sets and ads |
| `facebookCampaignDraft__pushDraftCampaign` | live | Create paused Meta campaigns, ad sets and ads from campaign data, ad copy and image URLs |

**Meta ad performance (8).** Same requirement.

| Tool | Kind | What it does |
| --- | --- | --- |
| `zernioAnalytics__accountOverview` | read | Spend, impressions, clicks, CTR, CPC, conversions and ROAS for a date range, against the previous period |
| `zernioAnalytics__tree` | read | An ad account's campaign, ad set and ad hierarchy |
| `zernioAnalytics__timeline` | read | An ad account's metrics over time |
| `zernioAnalytics__listCampaigns` | read | Campaigns with rolled-up metrics |
| `zernioAnalytics__listAds` | read | Ads under a campaign or ad set, with metrics |
| `zernioAnalytics__getAd` | read | One ad's metadata and creative |
| `zernioAnalytics__getAdAnalytics` | read | One ad's spend, CTR, CPC and conversions by day |
| `zernioAnalytics__getAdComments` | read | Public comments on the Facebook or Instagram post behind an ad |

Every tool is also a REST endpoint at `https://admakeai.com/api/v1/<tool>` (for example `adResearch.listProjects`) with an API key, documented at [admakeai.com/api/docs](https://admakeai.com/api/docs).

## Pricing

30 free credits on signup (8 in some countries), no card; plans from $39/mo. See [admakeai.com/pricing](https://admakeai.com/pricing).

## Links

- Website: [admakeai.com](https://admakeai.com)
- Agents, MCP and REST overview: [admakeai.com/agents](https://admakeai.com/agents)
- Claude Code plugin (also Codex, Cursor, Gemini CLI): [mesmerlord/admakeai-claude-plugin](https://github.com/mesmerlord/admakeai-claude-plugin)
- Agent skills: [mesmerlord/admakeai-agent-skills](https://github.com/mesmerlord/admakeai-agent-skills)
- Support: support@admakeai.com

## License

MIT
