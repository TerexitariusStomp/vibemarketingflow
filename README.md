# Vibemarketingflow â€” n8n Workflows

This repo contains a set of n8n workflow exports that automate a marketing content pipeline: ingest knowledge about an entity, build messaging foundations, and generate/schedule content. Import these JSON files into your n8n instance, wire credentials, map Airtable tables/fields, and run.

## Files
- OverallFlow.json â€” Master orchestration that builds the entity profile endâ€‘toâ€‘end (Persona, Core Messages, Target Audiences, Communication Style, Adjectives, Example Messages) and writes results to Airtable using multiple â€œCreate a recordâ€ nodes. Uses OpenRouter LLM and Perplexity.
- KnowledgeSetting.json â€” Upstream knowledge ingestion. Scrapes a target website (ScrapeNinja), defines â€œWebsiteInfo,â€ then derives Persona, Core Messages, Audiences, Style, Adjectives, and Examples. Intended to populate/refresh the entity record in Airtable. Includes an Airtable Trigger.
- DailyFlow.json â€” Daily longâ€‘form content generation (e.g., blog). Composes a prompt from the entity profile plus many news fields stored in Airtable, then generates formatted content via OpenRouter (and consults Perplexity for factuality). Designed for recurring runs.
- ContentCalendar.json â€” Produces an 84â€‘day content calendar (Day1â€¦Day84) and writes the schedule to an Airtable â€œContentCalendarâ€ table.
- OneTimeFlowMedia.json â€” Oneâ€‘off batch generation of 84 social post ideas (tweetâ€‘style). Enforces a strict JSON output using a Structured Output Parser, then writes multiple Airtable records.
- CompetitorAnalysis.json â€” Focused competitive analysis. Builds competitor personas, core messages, audiences, and recommended communication style, then stores results in Airtable.

All flows are inactive by default in the exports; activate them in n8n as needed.

## Prerequisites
- n8n (selfâ€‘hosted or cloud)
- Credentials (create in n8n â†’ Credentials and map them to nodes after import):
  - Airtable (Token API): Personal Access Token with data.records:read/write for your bases.
  - OpenRouter API Key: Used by lmChatOpenRouter nodes (default models can be changed).
  - Perplexity API Key: Used by perplexityTool nodes for retrievalâ€‘augmented answers.
  - ScrapeNinja API Key: Used by the ScrapeNinja node in KnowledgeSetting.

## Airtable Setup (recommended schema)
You can adapt names/fields, but the exports reference these patterns:
- Base: Your marketing base (IDs/URLs in the JSON must be reâ€‘selected in n8n after import).
- Tables and fields:
  - Websites: fields Website (URL), Created (datetime) â€” used by KnowledgeSettingâ€™s trigger and scraper.
  - EntityDescrips: longâ€‘text fields for EntitySummary, Adjectives, Core Messages, Target Audience, CommunicationStyle, ExampleMessages â€” written by OverallFlow/KnowledgeSetting.
  - ContentCalendar: text fields Day1 â€¦ Day84 â€” written by ContentCalendar.json.
  - Optional content/output tables (e.g., Blogs, Tweets) depending on your mapping in the â€œCreate a recordâ€ nodes.

After import, open each Airtable node and reâ€‘select:
- Authentication (select your Airtable Token credential)
- Base and Table (IDs/URLs are specific to the source instance)
- Columns/field mappings (update to match your schema)

## Importing into n8n
1. In n8n, go to Workflows â†’ Import from File and select each *.json file.
2. For each workflow, fix credentials:
   - Open any node showing a red credential alert and map it to your credential.
3. For Airtable nodes, select your base and table, then confirm field mappings.
4. Save the workflow.

## Running the Flows
A typical sequence:
1. KnowledgeSetting â€” Add a row in Websites with the target URL and run. This produces/updates the entity profile fields in Airtable.
2. OverallFlow â€” Run to consolidate and write a full set of persona/core messaging artifacts.
3. ContentCalendar â€” Run once to generate the 84â€‘day schedule in the ContentCalendar table.
4. OneTimeFlowMedia â€” Run to generate 84 social post ideas for the entity.
5. DailyFlow â€” Activate or run daily to produce longâ€‘form content (uses entity fields + many news fields in Airtable).
6. CompetitorAnalysis â€” Run as needed to analyze competitors and store the results.

Notes
- Some prompts mention external tools (e.g., Brave/Twitter) as guidance; actual nodes in these exports rely on Airtable, Perplexity, ScrapeNinja, and OpenRouter. You can add search/social nodes if you have them available in your n8n instance.
- Models can be swapped in lmChatOpenRouter nodes. Ensure your OpenRouter account allows the chosen model.
- All â€œCreate a recordâ€ nodes must target existing tables/fields; mismatches will cause runtime errors.

## Troubleshooting
- Credentials missing: Map credentials in each node with a red warning.
- Airtable errors: Reâ€‘select base/table, verify field names, check token scopes.
- Long outputs: If hitting Airtable field limits, switch fields to long text or split outputs across fields/records.
- Rate limits: Add Wait nodes or batching where needed.

## License
MIT License. See LICENSE for full text.
