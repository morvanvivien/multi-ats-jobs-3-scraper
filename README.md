# How to Scrape Breezy HR, Pinpoint and JazzHR Job Postings

> **Run it now, no code: [multi-ats-jobs-3-scraper on the Apify Store](https://apify.com/bibian/multi-ats-jobs-3-scraper)** &nbsp;|&nbsp; $1.20 per 1,000 postings &nbsp;|&nbsp; no login

Breezy HR, Pinpoint and JazzHR are three of the applicant tracking systems (ATS) that companies use to run their career pages. Each one has its own public feed or public page, but reading all three normally means building three separate scrapers. This guide shows how to get job postings from all three ATS platforms in one run — title, department, location, remote policy, salary, employment type and full description — with a free Apify Actor, from the console or from code.

Searches this page answers: breezy hr scraper, pinpoint ats scraper, jazzhr scraper, ATS jobs scraper.

## What you get from the Breezy HR, Pinpoint & JazzHR scraper

- Job postings from company career pages hosted on any of the three platforms, with the same output fields whatever the platform.
- Department, location and remote policy for every posting.
- Salary string or range, when the employer publishes it (common on Breezy HR and Pinpoint boards).
- Employment type and the full description as plain text when enabled.

Export the dataset as JSON, CSV or Excel, or read it through the Apify API. A real sample (five results, personal data removed, long texts truncated) is in [`sample-output.json`](sample-output.json).

## Output fields

| Field | What it is |
|---|---|
| `id`, `url` | Unique posting id (`platform:company:postingId`) and public link |
| `platform` | `breezyhr`, `pinpoint` or `jazzhr` |
| `company`, `title`, `department` | Hiring company, job title and department |
| `location`, `remote` | Where the job is based, and whether it is remote |
| `employmentType` | For example `Full-Time` |
| `salary` | Published salary string or range, when shown |
| `publishedAt` | Posting date, when the platform publishes it |
| `description` | Plain text, only when `includeDescriptions` is on |

## Use cases

- **Cover three ATS platforms from one Actor.** Give it company subdomains for any mix of Breezy HR, Pinpoint and JazzHR and get one combined, deduplicated dataset instead of running three separate tools.
- **Monitor specific employers.** Point it at the companies you care about and schedule a daily run to see new postings first.
- **Research salary ranges.** Breezy HR and Pinpoint often publish a salary string or range; filter the saved dataset on `salary` to see what employers disclose.
- **Feed a job board or alert.** None of these three platforms ships a public export for third parties; this Actor reads the same postings each company's own career page shows.

## Input

| Field | What it does |
|---|---|
| `breezyhrBoards` | Breezy HR company subdomains, taken from a company's career page URL, for example `seasats` from seasats.breezy.hr. |
| `pinpointBoards` | Pinpoint company subdomains, taken from a company's career page URL, for example `skims` from skims.pinpointhq.com. |
| `jazzhrBoards` | JazzHR company subdomains, taken from a company's career page URL, for example `storyteller` from storyteller.applytojob.com. |
| `maxItems` | The run stops once this many postings are saved, across all boards combined. |
| `includeDescriptions` | Add the full job description as plain text. Free for Pinpoint, which already publishes it in the list; one extra request per posting for Breezy HR and JazzHR. |

Leave every board list empty for a productive default run across all three platforms.

Example input:

```json
{
  "breezyhrBoards": ["seasats"],
  "pinpointBoards": ["skims"],
  "jazzhrBoards": ["storyteller"],
  "maxItems": 150,
  "includeDescriptions": true
}
```

## Three ways to use it

### 1. Apify console, no code

1. Open [the Actor page](https://apify.com/bibian/multi-ats-jobs-3-scraper) and click **Try for free** (a free Apify account is enough).
2. Fill in the boards you want (or leave them empty for the default set) and click **Start**.
3. When the run finishes, open the **Output** tab, check the table, then **Export** as CSV, Excel or JSON.
4. For a daily alert, open **Schedules**, create a daily schedule for the Actor and, optionally, add an integration that sends new results to Slack, e-mail or Google Sheets.

### 2. Apify API with JavaScript

```bash
npm install apify-client
export APIFY_TOKEN="your token from Apify console, Settings, Integrations"
node examples/run.js
```

```js
import { ApifyClient } from 'apify-client';

const client = new ApifyClient({ token: process.env.APIFY_TOKEN });

const run = await client.actor('bibian/multi-ats-jobs-3-scraper').call({
  breezyhrBoards: ['seasats'],
  maxItems: 150,
  includeDescriptions: true,
});

const { items } = await client.dataset(run.defaultDatasetId).listItems();
```

The complete file is [`examples/run.js`](examples/run.js).

### 3. Apify API with Python

```bash
pip install apify-client
export APIFY_TOKEN="your token from Apify console, Settings, Integrations"
python examples/run.py
```

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])

run = client.actor("bibian/multi-ats-jobs-3-scraper").call(run_input={
    "breezyhrBoards": ["seasats"],
    "maxItems": 150,
    "includeDescriptions": True,
})

for item in client.dataset(run.default_dataset_id).iterate_items():
    print(f"{item['title']} | {item.get('company')} | {item.get('platform')} | {item.get('location')}")
```

The complete file is [`examples/run.py`](examples/run.py). Never put your token in the code: read it from the `APIFY_TOKEN` environment variable as above.

## Pricing

Pay per result: **$1.20 per 1,000 postings**, plus a tiny start fee of $0.00005 per run. You pay only for the postings saved to the dataset. Apify's free plan includes monthly credits, enough to try it on a few hundred results. Prices are those shown on the [Store page](https://apify.com/bibian/multi-ats-jobs-3-scraper) when this guide was written; the Store page is the reference.

## Why this one

- **Three ATS platforms, one normalized schema.** Most competing Actors cover a single platform; this one lets a single run and a single schema span Breezy HR, Pinpoint and JazzHR together.
- **No browser, no login.** Every platform is read through a public feed or its own public career page; runs are fast and cheap.
- **Handles each platform's own quirk for a missing company.** Breezy HR and Pinpoint answer a wrong subdomain with a plain 404; JazzHR redirects it to its own marketing site instead. Both are detected and skipped with a clear warning rather than breaking the run or silently returning nothing.

## Limits and honest notes

- `salary`, when present, is the raw string or range the employer itself published; its format (currency, period) is not standardized across companies or platforms.
- `location` and `remote` are resolved from each platform's own structured fields; a posting with no place at all and a remote flag is reported as `"Remote"`.
- Each board's own `robots.txt` is checked before every run; a board whose rules disallow the relevant path is skipped with a warning rather than failing the whole run.
- Lines that carry a recruiting mailbox are removed from descriptions; no individual recruiter's name or direct contact is ever collected.
- Jobvite is not covered: its listings feed must be switched on by each client and no public client with the feed active was found, so no universal public API could be confirmed.
- This is an unofficial tool, not affiliated with Breezy HR, Pinpoint, JazzHR or any employer listed. Check the site terms that apply to your use.

## FAQ

**Does it work for any company, or only the ones in the default list?**
Any company whose career page is hosted on Breezy HR, Pinpoint or JazzHR. Use its subdomain in the matching input field.

**How do I find a company's subdomain?**
Open the company's own "Careers" page; if it redirects to a `*.breezy.hr`, `*.pinpointhq.com` or `*.applytojob.com` address, the part before that domain is the subdomain each field expects.

**Does it need a browser or a proxy?**
No. Every platform answers plain HTTP requests, so runs are fast and cheap.

**Can I get the data in Excel or Google Sheets?**
Yes. Export the dataset as CSV, Excel or JSON from the Apify console, or connect the Apify Google Sheets integration.

**Something does not work?**
Open an issue in the Issues tab. Every public actor is test-run daily and the broken ones are repaired.

## Try it

**[Open multi-ats-jobs-3-scraper on the Apify Store](https://apify.com/bibian/multi-ats-jobs-3-scraper)**

The code in `examples/` is released under the MIT license (see [LICENSE](LICENSE)). The data belongs to its original publishers.
