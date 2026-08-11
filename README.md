# AdMakeAI MCP Server

Remote MCP server for [AdMakeAI](https://admakeai.com), an AI ad creative platform for Meta (Facebook and Instagram) advertisers. Connect it once and your assistant can generate ad images and video, write ad copy, pull a competitor's live Meta ads, and read or pause real campaigns.

**Endpoint:** `https://admakeai.com/api/mcp` (Streamable HTTP, OAuth 2.1)

No API key to paste. The client registers itself, you approve a consent screen in the browser, and it receives a scoped token.

## What you can do with it

**Generate ad creatives.** Create ad images from a brief, edit an existing creative with an instruction, or batch a whole ad set across several angles at once. Generation runs on AdMakeAI's image pipeline and the results land in your project library, so anything the assistant makes is also available in the web app.

**Write ad copy.** Headlines, primary text, and CTA variations in a chosen messaging style. The generator reads the project's brand brief, so it stays on voice instead of inventing a new one per request.

**Research competitors.** Search for a company, scrape its live Meta ads, page through the results, open a single ad's detail, and save the good ones as inspiration on a project. This is the same corpus the AdMakeAI [competitor research tool](https://admakeai.com/competitor-research) runs on.

**Ship to Meta.** Upload a generated creative to a connected ad account, build a campaign draft, run a preflight check on it, and push it live. Preflight catches the usual rejections (aspect ratio, missing pixel, ad set targeting conflicts) before Meta does.

**Read performance.** Account overview, campaign and ad set and ad tree, spend and results timelines, per ad analytics, and ad comments. Pausing and resuming live campaigns is available too, but only if you explicitly grant the manage permission at connect time.

**Video ads.** Start a UGC-style video generation and poll it to completion.

## Connecting

Add `https://admakeai.com/api/mcp` as a remote MCP server in any client that supports Streamable HTTP with OAuth. Step by step guides for each client are at [admakeai.com/connect](https://admakeai.com/connect), and the agent platform overview is at [admakeai.com/agents](https://admakeai.com/agents).

On first call the server answers `401` with a `WWW-Authenticate` challenge, the client discovers the authorization server through RFC 9728 and RFC 8414 metadata, registers through RFC 7591 dynamic client registration or a client ID metadata document, and runs an authorization code flow with PKCE S256. Tokens are audience bound, short lived, and refreshed with rotation.

## Permissions

Three scopes map onto what the tools actually do. The consent screen lets you pick.

| Scope | Consent label | Covers |
| --- | --- | --- |
| `mcp:read` | Read your account | Projects, generations, research results, analytics |
| `mcp:write` | Create ads and drafts | Image and video generation, ad copy, ad sets, campaign drafts |
| `mcp:publish` | Manage live Facebook ads | Pause and resume running campaigns, ad sets, and ads |

Scope filtering is enforced at `tools/list`, not just at call time. A connection approved without `mcp:publish` never sees the pause and resume tools at all. Grants are listed and revocable from the Integrations section of your AdMakeAI dashboard.

Project creation is deliberately not exposed. An assistant that can mint brand projects tends to create a near duplicate rather than ask, then generate into the wrong brand and spend your credits doing it. List the projects, pick one, and ask when it is ambiguous.

## Tool surface

65 tools, grouped:

| Group | Tools |
| --- | --- |
| Account | `user__me`, `user__getCredits` |
| Projects | `adResearch__listProjects`, `adResearch__getProject`, `adResearch__pollStatus`, `project__updateBrief`, `project__getAssets` |
| Uploads | `upload__uploadImageBase64`, `upload__getUserImages` |
| Ad images | `adGeneration__create`, `adGeneration__generateImage`, `adGeneration__edit`, `adGeneration__getById`, `adGeneration__list`, `adGeneration__getRecent`, `adGeneration__searchAll`, `adGeneration__listPending`, `adGeneration__toggleFavorite` |
| Ad copy | `adCopyGeneration__generate`, `adCopyGeneration__getMessagingStyles` |
| Ad sets | `adSet__list`, `adSet__get`, `adSet__create`, `adSet__update`, `adSet__getGenerations`, `adSet__generateBatch`, `adSet__suggestDirections` |
| Video | `videoGeneration__start`, `videoGeneration__getStatus`, `videoGeneration__list`, `videoGeneration__listPending` |
| Competitor research | `adResearch__searchCompanies`, `adResearch__getCompanyAds`, `adResearch__addCompetitors`, `adResearch__scrapeCompetitor`, `adResearch__getCompetitorPreview`, `adResearch__getScrapedAds`, `adResearch__getAdDetails`, `adResearch__saveInspiration`, `adResearch__getSavedInspirations` |
| Meta publishing | `facebookConnection__list`, `facebookUpload__getCampaigns`, `facebookUpload__getAdSets`, `facebookUpload__runPreflight`, `facebookUpload__request`, `facebookUpload__list`, `facebookUpload__get`, `facebookCampaignDraft__sourceAssets`, `facebookCampaignDraft__getAgentDraft`, `facebookCampaignDraft__pushAgentDraft`, `facebookCampaignDraft__pushDraftCampaign` |
| Meta analytics | `zernioAnalytics__accountOverview`, `zernioAnalytics__tree`, `zernioAnalytics__timeline`, `zernioAnalytics__listCampaigns`, `zernioAnalytics__listAds`, `zernioAnalytics__getAd`, `zernioAnalytics__getAdAnalytics`, `zernioAnalytics__getAdComments`, `zernioAnalytics__pauseAd`, `zernioAnalytics__resumeAd`, `zernioAnalytics__pauseCampaign`, `zernioAnalytics__resumeCampaign`, `zernioAnalytics__pauseAdSet`, `zernioAnalytics__resumeAdSet` |

Every tool here is also reachable over plain REST at `https://admakeai.com/api/v1/<tool>` with an API key, for clients that do not speak MCP. Machine readable descriptions live at [admakeai.com/api/docs](https://admakeai.com/api/docs).

## Example prompts

- "List my AdMakeAI projects, then show the 5 most recent generations for the skincare one."
- "Scrape huel.com's live Meta ads and summarize the three angles they run most."
- "Generate 3 image variations for my skincare project: bright studio background, product front and center, headline about 24 hour hydration."
- "Which campaigns spent more than $20 last week with zero results?"
- "Pause every ad in the Retargeting campaign that has a CPA above $40." (needs the manage permission)

## Requirements

An [AdMakeAI account](https://admakeai.com/register). Read and research tools work on any account. Image, video, and copy generation consume plan credits; see [pricing](https://admakeai.com/pricing).

## Free tools

These run in the browser with no account and no MCP connection:

- [AI Ad Copy Generator](https://admakeai.com/free-tools/ad-copy-generator)
- [Video Ad Script Generator](https://admakeai.com/free-tools/video-ad-copy-generator)
- [Meta Ad Resizer](https://admakeai.com/free-tools/meta-ad-resizer)
- [AI Ad Translator](https://admakeai.com/free-tools/ad-translator)

## Links

- Website: [admakeai.com](https://admakeai.com)
- Connect guides: [admakeai.com/connect](https://admakeai.com/connect)
- Agents and API: [admakeai.com/agents](https://admakeai.com/agents)
- Blog: [admakeai.com/blog](https://admakeai.com/blog)
- Support: support@admakeai.com

## License

MIT
