# RSS to Airtable with AI Processing - Setup Guide

## Overview

This workflow automatically processes RSS feed articles with these features:

- ✅ **Scheduled RSS feed monitoring** (every 30 minutes, configurable)
- ✅ **Duplicate detection** via Airtable URL matching
- ✅ **Multi-method web scraping** with automatic fallback:
  1. Apify API (best for complex sites)
  2. Firecrawl API (good for general content)
  3. Direct HTTP requests (works for simple sites)
  4. RSS content fallback (always works)
- ✅ **AI-powered summarization** using OpenAI GPT-4
- ✅ **AI content generation** creates new articles based on source
- ✅ **Airtable storage** with complete metadata

---

## Quick Start

### 1. Import the Workflow

1. Open n8n
2. Go to **Workflows** → **Import from File**
3. Select `RSS_to_Airtable_with_AI_Processing.json`
4. Click **Import**

### 2. Set Up Airtable

#### Create Your Airtable Base

1. Go to [Airtable](https://airtable.com)
2. Create a new base called **"Article Repository"** (or any name)
3. Create a table called **"Articles"**
4. Add these fields to your table:

| Field Name | Field Type | Description |
|------------|------------|-------------|
| **URL** | URL | The original article URL (used for deduplication) |
| **Title** | Single line text | Article title from RSS feed |
| **Published Date** | Date | When the article was published |
| **Full Content** | Long text | Complete scraped article content |
| **AI Summary** | Long text | AI-generated summary (150-200 words) |
| **Generated Content** | Long text | New AI-generated content (300-400 words) |
| **Scraping Method** | Single select | Which method successfully scraped content |
| **Processed Date** | Date | When the workflow processed this article |
| **RSS Source** | URL | The RSS feed URL |

#### Get Airtable Credentials

1. Go to [Airtable Account](https://airtable.com/account)
2. Click **Generate API key** (or use existing key)
3. Copy your API key
4. Get your **Base ID**:
   - Open your base in Airtable
   - Go to **Help** → **API documentation**
   - Your Base ID is shown at the top (looks like: `appXXXXXXXXXXXXXX`)

### 3. Set Up n8n Credentials

#### Airtable Credential

1. In n8n, go to **Credentials** → **New**
2. Search for **Airtable API**
3. Enter your **API Key**
4. Save as **"Airtable API"**

#### OpenAI Credential

1. Go to **Credentials** → **New**
2. Search for **OpenAI**
3. Enter your **OpenAI API Key** (get from [OpenAI Platform](https://platform.openai.com/api-keys))
4. Save as **"OpenAI API"**

#### Apify Credential (Optional but Recommended)

1. Create account at [Apify](https://apify.com)
2. Get your API token from **Settings** → **Integrations**
3. In n8n, go to **Credentials** → **New**
4. Search for **HTTP Header Auth**
5. Set:
   - **Name**: `apify_auth`
   - **Header Name**: `Authorization`
   - **Header Value**: `Bearer YOUR_APIFY_TOKEN`
6. Save

#### Firecrawl Credential (Optional but Recommended)

1. Create account at [Firecrawl](https://firecrawl.dev)
2. Get your API key from dashboard
3. In n8n, go to **Credentials** → **New**
4. Search for **HTTP Header Auth**
5. Set:
   - **Name**: `firecrawl_auth`
   - **Header Name**: `Authorization`
   - **Header Value**: `Bearer YOUR_FIRECRAWL_API_KEY`
6. Save

### 4. Configure Environment Variables

Set these environment variables in n8n (or in workflow nodes directly):

```bash
# Required
RSS_FEED_URL=https://your-blog.com/feed
AIRTABLE_BASE_ID=appXXXXXXXXXXXXXX
AIRTABLE_TABLE_NAME=Articles

# Optional (if not set, scraping will use fallback methods)
# These are handled via credentials configured above
```

#### Setting Environment Variables in n8n:

**Option A: In Docker/Docker Compose**
```yaml
environment:
  - RSS_FEED_URL=https://your-blog.com/feed
  - AIRTABLE_BASE_ID=appXXXXXXXXXXXXXX
  - AIRTABLE_TABLE_NAME=Articles
```

**Option B: In n8n Settings**
1. Go to **Settings** → **Environments**
2. Add variables:
   - `RSS_FEED_URL`
   - `AIRTABLE_BASE_ID`
   - `AIRTABLE_TABLE_NAME`

**Option C: Directly in Workflow Nodes**
If you don't want to use environment variables, you can edit these nodes:
- **RSS Feed Reader**: Replace `{{ $env.RSS_FEED_URL }}` with your actual feed URL
- **Get Existing Articles from Airtable**: Replace `{{ $env.AIRTABLE_BASE_ID }}` with your base ID
- **Add to Airtable**: Replace environment variables with actual values

### 5. Link Credentials to Workflow Nodes

Open the workflow and update these nodes with your credential names:

1. **Get Existing Articles from Airtable** node
   - Select your Airtable credential: `Airtable API`

2. **Add to Airtable** node
   - Select your Airtable credential: `Airtable API`

3. **Generate AI Summary** node
   - Select your OpenAI credential: `OpenAI API`

4. **Generate New Content** node
   - Select your OpenAI credential: `OpenAI API`

5. **Try Apify Scraper (Method 1)** node (optional)
   - Select your Apify credential: `apify_auth`

6. **Try Firecrawl API (Method 2)** node (optional)
   - Select your Firecrawl credential: `firecrawl_auth`

---

## Testing the Workflow

### Test with Manual Trigger

1. Click on the **"Manual Trigger (Testing)"** node
2. Click **"Execute Workflow"**
3. Watch the workflow process articles from your RSS feed
4. Check your Airtable to see results

### Test Individual Nodes

You can test scraping on a specific article:

1. Click on any node in the scraping section
2. Click **"Execute Node"**
3. Check the output

---

## Workflow Configuration

### Adjust Schedule

To change how often the workflow runs:

1. Click on **"Schedule Trigger"** node
2. Modify the **triggerTimes** parameter:
   - Every 15 minutes: `{"mode": "everyX", "value": 15, "unit": "minutes"}`
   - Every hour: `{"mode": "everyX", "value": 1, "unit": "hours"}`
   - Specific times: Use cron expression

### Customize AI Prompts

#### Summary Customization

Edit the **"Generate AI Summary"** node:
- Change system message to adjust summary style
- Adjust `maxTokens` for longer/shorter summaries
- Adjust `temperature` (0.1-1.0) for more/less creative summaries

#### Content Generation Customization

Edit the **"Generate New Content"** node:
- Modify system message to change content style
- Adjust `maxTokens` for longer/shorter content
- Adjust `temperature` for creativity level

### Adjust Scraping Methods

#### To Disable Optional Scrapers

If you don't want to use Apify or Firecrawl:

1. These will automatically fail and fall through to next method
2. You can disable the nodes entirely (won't affect workflow)

#### To Add Custom Scraping Logic

Add your own HTTP request node in the fallback chain:

1. Insert between any IF node's "false" output and the next scraper
2. Follow the same pattern: HTTP Request → IF Success → Set Content → Merge

---

## How the Workflow Works

### Flow Diagram

```
[RSS Feed]
    ↓
[Get Existing Airtable URLs]
    ↓
[Process Each Article] (Loop)
    ↓
[Check for Duplicate URL]
    ↓ (If New)
[Try Apify Scraper]
    ↓ (If fails)
[Try Firecrawl Scraper]
    ↓ (If fails)
[Try Direct HTTP Request]
    ↓ (If fails)
[Use RSS Content Fallback]
    ↓
[Merge All Scraping Paths]
    ↓
[Generate AI Summary]
    ↓
[Generate New Content]
    ↓
[Format Data for Airtable]
    ↓
[Save to Airtable]
    ↓
[Continue Loop] → Back to next article
```

### Key Features

1. **Deduplication**
   - Checks article URL against all existing Airtable records
   - Skips processing if URL already exists
   - Prevents duplicate entries

2. **Fallback Chain**
   - Each scraping method has `continueOnFail: true`
   - IF nodes check success status codes
   - Automatically falls through to next method on failure
   - Always succeeds with RSS content fallback

3. **Error Handling**
   - Workflow settings include `retryOnFail: true`
   - 3 automatic retries with 1-second delay
   - 1-hour execution timeout
   - Error handler node for critical failures

4. **Data Integrity**
   - All article metadata preserved
   - Tracks which scraping method succeeded
   - Records processing timestamp
   - Maintains RSS source reference

---

## Troubleshooting

### Issue: No Articles Being Processed

**Check:**
1. RSS feed URL is correct and accessible
2. RSS feed has new articles since last run
3. Articles aren't already in Airtable (check URLs)
4. Manual trigger works but cron doesn't → check workflow is activated

### Issue: All Scraping Methods Failing

**Check:**
1. Article URLs are valid and accessible
2. Try opening URLs in browser manually
3. Check if sites require authentication
4. Verify HTTP headers are correct
5. Check execution logs for specific errors

**Solution:** The RSS fallback will always work, but might have less content.

### Issue: Apify/Firecrawl Not Working

**Check:**
1. API credentials are correctly configured
2. API keys are valid and not expired
3. You have API credits/quota remaining
4. Node credentials are linked properly

**Solution:** Workflow will automatically fall back to other methods.

### Issue: OpenAI Errors

**Check:**
1. OpenAI API key is valid
2. You have sufficient API credits
3. Content isn't too long (check token limits)
4. Rate limits not exceeded

**Solution:**
- Increase retry delays
- Reduce `maxTokens` in OpenAI nodes
- Use `gpt-3.5-turbo` instead of `gpt-4o-mini` for lower costs

### Issue: Airtable Errors

**Check:**
1. Base ID is correct
2. Table name matches exactly (case-sensitive)
3. All field names in workflow match Airtable exactly
4. API key has write permissions
5. Not hitting Airtable rate limits

### Issue: Duplicate Articles Still Being Added

**Check:**
1. Deduplication function is working (test the Function node)
2. URL field in Airtable matches exactly
3. IF node condition is correct
4. Airtable List operation is returning data

---

## Advanced Configuration

### Process Multiple RSS Feeds

To monitor multiple feeds:

**Option 1: Multiple Workflow Instances**
- Duplicate the workflow
- Change RSS_FEED_URL for each instance
- Use different schedules to spread load

**Option 2: Modify Workflow**
1. Replace RSS Feed Reader with a manual list of URLs
2. Add a Split In Batches before RSS reader
3. Loop through each feed URL

### Filter Articles by Date

Add an IF node after RSS Feed Reader:
```javascript
// Only process articles from last 24 hours
const pubDate = new Date($json.pubDate);
const oneDayAgo = new Date(Date.now() - 24 * 60 * 60 * 1000);
return pubDate > oneDayAgo;
```

### Custom HTML Selectors

Edit **"Extract Article from HTML"** node:
- Add site-specific CSS selectors
- Use multiple selectors with fallbacks
- Extract additional metadata (author, images, etc.)

### Webhook Integration

Replace Cron trigger with Webhook:
1. Add Webhook node
2. Configure webhook URL
3. Call webhook from external service (Zapier, IFTTT, etc.)

---

## Cost Estimation

### OpenAI Costs (GPT-4o-mini)

Per article:
- Summary: ~300 tokens = $0.0001 (input) + ~200 tokens output = $0.0001
- Content: ~500 tokens = $0.0002 (input) + ~400 tokens output = $0.0002
- **Total per article: ~$0.0006**

Processing 100 articles: ~$0.06
Processing 1000 articles: ~$0.60

### Apify Costs

- Pay-as-you-go: ~$0.001-0.005 per scrape
- Free tier: Limited requests

### Firecrawl Costs

- Free tier: 500 requests/month
- Pro: $20/month for 10,000 requests

### Airtable Costs

- Free: 1,200 records per base
- Plus: $10/month for 5,000 records per base

---

## Best Practices

1. **Start Small**
   - Test with manual trigger first
   - Process a few articles before automating
   - Verify Airtable structure

2. **Monitor Execution**
   - Check n8n execution history
   - Review Airtable for data quality
   - Monitor API costs

3. **Optimize Performance**
   - Reduce RSS check frequency if feed updates slowly
   - Use more aggressive filtering (date, keywords)
   - Consider archiving old articles

4. **Security**
   - Never commit API keys to version control
   - Use environment variables for sensitive data
   - Regularly rotate API keys
   - Set up IP restrictions where possible

5. **Data Management**
   - Regularly backup Airtable base
   - Archive old articles
   - Monitor storage limits

---

## Support and Resources

### Documentation
- [n8n Documentation](https://docs.n8n.io/)
- [Airtable API Docs](https://airtable.com/developers/web/api/introduction)
- [OpenAI API Docs](https://platform.openai.com/docs)
- [Apify Documentation](https://docs.apify.com/)
- [Firecrawl Docs](https://docs.firecrawl.dev/)

### Community
- [n8n Community Forum](https://community.n8n.io/)
- [n8n Discord](https://discord.gg/n8n)

### This Repository
- Check other workflow examples in `/workflows`
- Review `CLAUDE.md` for repository structure
- See `README.md` for general information

---

## Changelog

### Version 1.0 (Initial Release)
- RSS feed monitoring with scheduling
- Airtable deduplication
- Multi-method scraping (Apify, Firecrawl, HTTP, RSS fallback)
- AI summarization and content generation
- Complete error handling and retries

---

## License

This workflow is part of the n8n-workflows repository and follows the same license as the parent repository.

---

## Contributing

If you improve this workflow:
1. Test thoroughly
2. Update this documentation
3. Add your changes to the CHANGELOG
4. Submit a pull request

---

**Happy Automating! 🚀**
