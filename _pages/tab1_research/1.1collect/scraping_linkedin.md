---
layout: single
title: "Building a LinkedIn Data Pipeline for Alumni Outcomes"
permalink: /Research/Data-Collection/linkedin-alumni-pipeline/
#date: 2026-07-28
author_profile: true
excerpt: "How to build a pragmatic, six-step Python pipeline using Apify to scrape LinkedIn for alumni employment and education data—bridging the gap when survey response rates fall short."
---
> {{ page.excerpt }}

**Contents**

- [Introduction & Narrative Hook](#introduction--narrative-hook)
- [Step 1: Identify Who's Missing — Filtering Survey Non-Respondents](#step-1-identify-whos-missing--filtering-survey-non-respondents)
- [Step 2: Chunk It Up — Preparing Batches for the API](#step-2-chunk-it-up--preparing-batches-for-the-api)
- [Step 3: The Engine Room — Scraping LinkedIn via Apify](#step-3-the-engine-room--scraping-linkedin-via-apify)
- [Step 4: Assemble the Pieces — Combining Results and Re-Mapping IDs](#step-4-assemble-the-pieces--combining-results-and-re-mapping-ids)
- [Step 5: From JSON Soup to Clean Columns — Extracting Structured Fields](#step-5-from-json-soup-to-clean-columns--extracting-structured-fields)
- [Step 6: The Final Enrichment — Adding Demographics and Mapping to Tableau Targets](#step-6-the-final-enrichment--adding-demographics-and-mapping-to-tableau-targets)
- [Summary, Reflection & Future Outlook](#summary-reflection--future-outlook)

This article walks through a complete six-step Python pipeline that uses Apify to scrape LinkedIn for alumni employment and education data. Whether you are tackling your first data collection project or looking to refine an existing workflow, each section below stands alone—feel free to jump to the step you need most, or read from start to finish for the full blueprint.

---

## Introduction & Narrative Hook

Every institutional researcher knows the feeling: you send the alumni survey, wait weeks, send reminders, and still land at a 10% or less response rate. Self-reported outcome data has always been the gold standard, but it's also the one that leaves you staring at half-empty spreadsheets.

We needed employment and education outcomes for thousands of alumni who never touched the survey. The mandate was clear—find the data, or leave the analysis incomplete to represent the whole population.

Enter LinkedIn Profile Search by Name. It's not a perfect substitute. Profiles go stale, alumni forget to update their jobs, and the platform's terms of service demand careful navigation. But when you're sitting on an 90% non-response rate, "imperfect but available" beats "perfect but missing" every time. The trick is building a pipeline that acknowledges those imperfections and handles them systematically.

Manual workarounds are a thing of the past. With a few scripts and a few Apify actors, we built a pipeline that **automates the process of collecting LinkedIn data for thousands of alumni**. Over six scripts and a API call for each alumni profile, we built exactly that. Here's the blueprint—warts, workarounds, and all.

<pre class="mermaid">
flowchart TD
    A[Survey&lt;br/&gt;Non-Respondents] --> B[Filter&lt;br/&gt;Contact List]
    B --> C[Chunk into&lt;br/&gt;Batches of 100]
    C --> D[Apify LinkedIn&lt;br/&gt;Search by Name]
    D --> E[Combine&lt;br/&gt;Raw Results]
    E --> F[Re-Map IDs via&lt;br/&gt;Fuzzy Matching]
    F --> G[Extract &&lt;br/&gt;Clean Fields]
    G --> H[Enrich with&lt;br/&gt;Demographics]
    H --> I[Tableau-Ready&lt;br/&gt;Dataset]
</pre>

## Step 1: Identify Who's Missing — Filtering Survey Non-Respondents

**File:** `1.filter_linkedinList.py`

### Purpose & Story

Before you scrape a single profile, you need to know *who* to scrape. The whole pipeline exists because a subset of alumni didn't respond to the survey. Step 1 is a straightforward set difference: your master contact list minus your survey respondents.

<pre class="mermaid">
flowchart TD
    A[Contact List&lt;br/&gt;all alumni] --> C{People Code ID&lt;br/&gt;in survey?}
    B[Survey Responses] --> C
    C -->|No| D[LinkedIn Target List]
    C -->|Yes| E[Already have data&lt;br/&gt;skip]
</pre>

### Hands-On Guide

Load your master contact list (with emails, People Code IDs, names) and your survey response dataset. Use a boolean mask to find contacts whose `People Code ID` does **not** appear in the survey's respondent ID column. Drop duplicates and export.

### Key Code Snippet

```python
mask = df_contact['People Code ID'].isin(df_survey['Invite Custom Field 2'])
df_linkedin = df_contact[~mask].drop_duplicates()
df_linkedin.to_csv("linkedin_targets.csv", index=False)
```

### Pro-Tips & Lessons Learned

- **Keep the People Code ID.** This internal identifier is your referential integrity anchor through the entire pipeline. Never drop it.
- **Drop duplicates early.** If the same person appears twice in your contact list, the set difference will still filter them correctly, but you'll save API credits downstream.

---

## Step 2: Chunk It Up — Preparing Batches for the API

**File:** `2create_inputList.py`

### Purpose & Story

Apify's LinkedIn Profile Search by Name actor is powerful, but it processes records one at a time. Running 900 records in a single monolithic call is a recipe for timeout disasters, runaway costs, and uncheckable progress. The solution is dead simple: split your target list into chunks of 100 rows.

<pre class="mermaid">
flowchart LR
    A[LinkedIn Targets&lt;br/&gt;900 rows] --> B[Chunk 1:&lt;br/&gt;rows 1-100]
    A --> C[Chunk 2:&lt;br/&gt;rows 101-200]
    A --> D[...]
    A --> E[Chunk 9:&lt;br/&gt;rows 801-900]
</pre>

### Hands-On Guide

Set `chunk_size = 100`, loop through the dataframe with `range(0, total_rows, chunk_size)`, slice with `iloc`, and save each slice with a human-readable filename like `linkedin_list_1_100.csv`. The script ends with a comment pointing to the Apify actor URL—a small but helpful nudge.

### Key Code Snippet

```python
chunk_size = 100
for i in range(0, total_rows, chunk_size):
    end_row = min(i + chunk_size, total_rows)
    subset = df_linkedin.iloc[i:end_row]
    filename = f'linkedin_list_{i+1}_{end_row}.csv'
    subset.to_csv(filename, index=False)
```

### Pro-Tips & Lessons Learned

- **100 rows is the sweet spot.** At ~15–30 seconds per profile, one chunk takes 20–30 minutes. This is manageable enough to monitor without being glued to your screen.
- **Filenames encode the range.** This pattern (`linkedin_list_{start+1}_{end}.csv`) makes it trivially easy to know which chunks are done, which are pending, and whether any files overlap.
- **Long lists need a concatenation helper.** If you have more than ~700 names (Apify's approximately $5 free credit limit), you may need to merge chunks. The companion script `3optional_apilonglist.py` shows how: just `pd.concat` two files sharing the same columns.

---

## Step 3: The Engine Room — Scraping LinkedIn via Apify

**Files:** `3api.env`, `3getresult_api.py`

### Purpose & Story

This is the heart of the operation. You send a name, a school filter ("Lasell University"), and a strict-search flag to Apify's pre-built LinkedIn Profile Search by Name actor. The actor returns a rich JSON payload: current position, full experience history, education, location, profile URL, connections count, and more.

<pre class="mermaid">
sequenceDiagram
    participant P as Python Script
    participant A as Apify Actor
    participant L as LinkedIn
   
    P->>A: POST firstName, lastName, school="Lasell"
    A->>L: Search LinkedIn profiles
    L-->>A: Match results
    A-->>P: Dataset with profile data
    Note over P: Append pcid to each result
    P->>P: Save chunk results to CSV
</pre>

### Hands-On Guide

1. **Create an Apify account** at [console.apify.com](https://console.apify.com). Navigate to Settings → Integrations to copy your API token.
2. **Store the token** in a `.env` file (included in the repo as `3api.env`). Use `python-dotenv` to load it. **Important:** The comments warn that you should fully exit VS Code and restart to ensure the environment variable refreshes.
3. **Configure the actor input** with `firstName`, `lastName`, `schools`, `strictSearch=True`, `maxPages=1`, and `maxItems=3`. Set `profileScraperMode` to `"Full"` to pull complete experience details.
4. **Iterate through each row** of your chunk CSV, call `client.actor(...).call(run_input=actor_input)`, and append the returned items to `all_profiles`. Crucially, inject the `pcid` (People Code ID) into each returned item *before* saving it, maintaining the link back to your institutional data.
5. **Save to CSV** with a filename reflecting the chunk range (e.g., `linkedin_results_1_100.csv`).

### Key Code Snippet

```python
load_dotenv(dotenv_path='3api.env')
client = ApifyClient(os.getenv("APIFY_TOKEN"))

for _, row in df_chunk.iterrows():
    actor_input = {
        "profileScraperMode": "Full",
        "firstName": str(row["First Name"]).strip(),
        "lastName": str(row["Last Name"]).strip(),
        "schools": ["Lasell University"],
        "strictSearch": True,
        "maxPages": 1,
        "maxItems": 3,
    }
    run = client.actor("harvestapi/linkedin-profile-search-by-name").call(run_input=actor_input)
    for item in client.dataset(run.default_dataset_id).iterate_items():
        item["pcid"] = row["People Code ID"]
        all_profiles.append(item)
```

### Pro-Tips & Lessons Learned

| Consideration               | Detail                                                                                                                                                |
| --------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Free credit**       | ~$5/month (renews on signup day). At ~$0.68 per 100 records, you get ~650–700 records per cycle.                                                     |
| **Speed**             | 15–30 seconds per profile; 20–30 minutes per 100-record chunk.                                                                                      |
| **Credit limit**      | When you run out, the script doesn't crash—it returns error rows but keeps going. You still get partial results.                                     |
| **Multiple accounts** | Use a secondary email to get a second free allocation. The`.env` file makes token swapping painless but requires a full IDE restart to take effect. |
| **Monitor runs**      | Track progress at`console.apify.com/actors/runs`.                                                                                                   |

---

## Step 4: Assemble the Pieces — Combining Results and Re-Mapping IDs

**Files:** `4combine_results.py`, `4optionaladd_pcid.py`

### Purpose & Story

After running all chunks through Apify, you'll have a scatter of result CSVs. These need to be combined into one master file. But there's a catch: the Apify search doesn't always return results in the same order you submitted names, and sometimes it returns multiple matches per name. The `pcid` you injected in Step 3 handles the first problem, but we need a secondary fuzzy-matching step for robustness.

<pre class="mermaid">
flowchart TD
    subgraph Combine
        A[results_1_100.csv] --> D[glob + pd.concat]
        B[results_101_200.csv] --> D
        C[...] --> D
        D --> E[combined_linkedin_results.csv]
    end
    subgraph Re-Map IDs
        E --> F{Match by&lt;br/&gt;first name exact +&lt;br/&gt;last name initial?}
        F -->|Yes| G[Assign pcid]
        F -->|No| H[Assign None]
        G --> I[combined_with_pcid.csv]
        H --> I
    end
</pre>

### Hands-On Guide

**Combining:** Use `glob.glob()` with a `*` wildcard pattern to collect all result files. Sort them to guarantee order, then `pd.concat` into a single dataframe.

**Re-Mapping IDs:** This is the clever part. The script iterates through returned profiles, looks for exact first-name matches in the original upload list, then checks if the first letter of the last name matches. This two-tier heuristic handles:

- Typos in the scraped data
- Middle initials in one dataset but not the other
- Duplicate common names (e.g., two "Patrick MacDonald" entries)

### Key Code Snippet

```python
# Fuzzy matching: exact first name + last name initial
for _, ret_row in df_return.iterrows():
    ret_first = str(ret_row.get("First Name", "")).strip().lower()
    ret_last = str(ret_row.get("Last Name", "")).strip().lower()
    candidates = df_upload[df_upload['clean_first'] == ret_first]
    for _, cand_row in candidates.iterrows():
        if ret_last and cand_last and ret_last[0] == cand_last[0]:
            if cand_pcid not in used_pcids:
                best_pcid = cand_pcid
                used_pcids.add(best_pcid)
                break
    matched_pcids.append(best_pcid)
```

### Pro-Tips & Lessons Learned

- **The `used_pcids` set is critical.** Without it, two "Patrick MacDonald" entries would both map to the same ID. The set ensures each ID is assigned at most once.
- **This step is optional** but *highly recommended*. If your Apify injection worked perfectly, the `pcid` is already in the data. But edge cases happen—profiles with no match, API timeouts, corrupted rows. This second pass is insurance.
- **Sort your files before concatenating.** Without the sort, `glob` returns files in filesystem order, which is unpredictable. Consistent ordering matters when you're debugging missing rows.

---

## Step 5: From JSON Soup to Clean Columns — Extracting Structured Fields

**File:** `5extract_linkedin.py`

### Purpose & Story

The raw Apify output stores experience and education data as stringified JSON arrays. A single cell in `currentPosition` looks like this:

```
"[{'position': 'Software Engineer', 'companyName': 'Google', 'employmentType': 'Full-time'}]"
```

This is a job-search disaster waiting to happen. You can't filter, pivot, or visualize stringified lists. Step 5 uses Python's `ast.literal_eval` to safely parse these strings into dictionaries, then extracts individual fields into flat, analysis-ready columns.

<pre class="mermaid">
flowchart TD
    A[Raw currentPosition&lt;br/&gt;stringified JSON] --> B[ast.literal_eval]
    B --> C[List of dicts]
    C --> D[extract_value&lt;br/&gt;val, key]
    D --> E[job_title]
    D --> F[employer]
    D --> G[job_status]
    H[Raw profileTopEducation] --> I[...]
    I --> J[school_name]
    I --> K[degree]
    I --> L[field_of_study]
</pre>

### Hands-On Guide

Define a helper function `extract_value(val, key)` that:

1. Checks for NaN
2. Uses `ast.literal_eval` to safely parse the string
3. Extracts the key from the first element if it's a list, or directly if it's a dict

Apply this function to `currentPosition` and `profileTopEducation` columns. Then deduplicate by `firstName` + `lastName`, keeping the record with the highest `connectionsCount` (the most-connected profile is most likely the correct, active one).

### Key Code Snippet

```python
def extract_value(val, key):
    if pd.isna(val):
        return None
    try:
        parsed = ast.literal_eval(val)
        if isinstance(parsed, list) and len(parsed) > 0:
            return parsed[0].get(key)
        elif isinstance(parsed, dict):
            return parsed.get(key)
    except (ValueError, SyntaxError):
        pass
    return None

df['job_title'] = df['currentPosition'].apply(lambda x: extract_value(x, 'position'))
df['employer'] = df['currentPosition'].apply(lambda x: extract_value(x, 'companyName'))
df['school_name'] = df['profileTopEducation'].apply(lambda x: extract_value(x, 'schoolName'))
```

### Pro-Tips & Lessons Learned

- **`ast.literal_eval` is safer than `eval`.** `eval` can execute arbitrary code. `literal_eval` only parses Python literals (strings, lists, dicts, etc.). Always use it when parsing unknown data.
- **Drop duplicates *after* extraction.** Sorting by `connectionsCount` descending before deduplication means you keep the most robust profile—likely the one the alumni actually maintains.
- **Handle malformed data gracefully.** The `try/except` in `extract_value` returns `None` for unparseable strings. Your pipeline should never crash on a single bad cell.

| Output Column      | Source Field            | Extracted Key      |
| ------------------ | ----------------------- | ------------------ |
| `job_title`      | `currentPosition`     | `position`       |
| `employer`       | `currentPosition`     | `companyName`    |
| `job_status`     | `currentPosition`     | `employmentType` |
| `school_name`    | `profileTopEducation` | `schoolName`     |
| `degree`         | `profileTopEducation` | `degree`         |
| `field_of_study` | `profileTopEducation` | `fieldOfStudy`   |

---

## Step 6: The Final Enrichment — Adding Demographics and Mapping to Tableau Targets

**File:** `6addCol_linkedin.py`

### Purpose & Story

You now have clean LinkedIn data, but it's still anonymous. To make it useful for reporting, you need to fold in institutional context: major, graduation year, program (UNDER vs. GRAD), and—most importantly—the **Population** classification that your Tableau dashboard expects. This step is a left join back to the original contact list, followed by a business-rule-based recoding.

<pre class="mermaid">
flowchart TD
    A[Clean LinkedIn Data&lt;br/&gt;pcid, job_title, employer, ...] --> C{Left Join on pcid}
    B[Contact List&lt;br/&gt;Major, Grad Year, Program] --> C
    C --> D[Enriched Dataset]
    D --> E{Classify Population}
    E -->|Grad Year 2019 + UNDER| F[UG 5 Year]
    E -->|Grad Year 2025 + UNDER| G[UG 6 Month]
    E -->|Grad Year 2019 + GRAD| H[Grad 5 Year]
    E -->|Grad Year 2025 + GRAD| I[Grad 6 Month]
    E -->|Other| J[Other]
    F --> K[Tableau-Ready Output]
    G --> K
    H --> K
    I --> K
    J --> K
</pre>

### Hands-On Guide

1. **Left join** the clean LinkedIn data to the contact list on `pcid`, pulling in `Major`, `postal_gradyear`, and `Program`.
2. **Classify Population** using `np.select`: if graduation year is 2019 and program is UNDER, label them "UG 5 Year"; if 2025 and UNDER, "UG 6 Month"; same logic for GRAD programs.
3. **Rename columns** to match the Tableau master schema: `job_title` → `Job Title`, `employer` → `Employer Name`, `job_status` → `Employment Status`, etc.
4. **Clean edge cases:**
   - Drop rows where `PCID` is NaN
   - If the `institution name` is "Lasell University", null out both the institution and degree (scraping your own institution's name in the education field is a false positive)
   - Remove rows where both `Job Title` and `institution name` are NaN—no useful data

### Key Code Snippet

```python
df_linkedin_add = df_linkedin_add.assign(
    postal_gradyear=pd.to_numeric(df_linkedin_add['postal_gradyear'], errors='coerce'),
    Program=df_linkedin_add['Program'].astype(str).str.upper()
).assign(
    Population=lambda df: np.select(
        [(df['postal_gradyear'] == 2019) & (df['Program'] == 'UNDER'),
         (df['postal_gradyear'] == 2025) & (df['Program'] == 'UNDER'),
         (df['postal_gradyear'] == 2019) & (df['Program'] == 'GRAD'),
         (df['postal_gradyear'] == 2025) & (df['Program'] == 'GRAD')],
        ['UG 5 Year', 'UG 6 month', 'Grad 5 Year', 'Grad 6 month'],
        default='Other'
    )
)

# Clean: Lasell University in education field is a false positive
df_linkedin_add.loc[df_linkedin_add['institution name'] == 'Lasell University', 
                    ['institution name', 'Degree name']] = np.nan
```

### Pro-Tips & Lessons Learned

- **Lasell University in the education field is almost always a false positive.** The Apify search uses the school name as a filter, so profiles of current Lasell students or employees show up, but their *education* entry for "Lasell University" is redundant—it's the same institution that sent them the survey. Null it out.
- **Column names matter.** The final rename step is tedious but essential. Your visualization tool expects a specific schema. Map early, map explicitly.
- **Graduation year as numeric.** `pd.to_numeric(..., errors='coerce')` handles stray non-numeric entries gracefully. One malformed year entry won't crash the whole pipeline.

---  

## Summary, Reflection & Future Outlook

This pipeline gave us employment and education data for roughly 650–700 alumni who otherwise would have been silent in our outcomes report. At approximately <span>$0.68</span> per 100 records, the entire operation cost less than <span>$10</span> in Apify credits—a fraction of what a survey incentive program would run.

But let's be honest about the limitations.

| Limitation                       | Impact                                   | Mitigation                                                                    |
| -------------------------------- | ---------------------------------------- | ----------------------------------------------------------------------------- |
| **Profile staleness**      | Alumni may not update LinkedIn regularly | Cross-reference with survey data when available; note data collection date    |
| **Search false positives** | Wrong person with the same name          | Strict school filter + connectionsCount deduplication helps but isn't perfect |
| **Coverage gaps**          | Not every alum has a LinkedIn profile    | Document coverage rate alongside results                                      | 
| **Terms of service**       | LinkedIn and Apify's ToS evolve          | Review before each collection cycle; use public data responsibly              |
| **One-time snapshot**      | No change tracking                       | Future: add date-stamped collection cycles for trend analysis                 |

### Where to Take This Next

If I were rebuilding this pipeline today, I'd add three features:
 
1. **Incremental collection.** Store a hash of each profile's key fields. On subsequent runs, only re-scrape profiles whose hash changed. This turns a snapshot into a panel dataset.
2. **Confidence scoring.** Not every name match is equally reliable. Add a confidence column based on the number of matching fields (name + school + location + industry) so downstream analysts can filter or weight results.
3. **Automated chunk orchestration.** Instead of manually launching each chunk, wrap the Apify calls in a queue with retry logic, credit monitoring, and Slack notifications. A lightweight Airflow DAG or even a shell loop would eliminate the 20-minute "watch the spinner" gap between chunks.

The survey isn't dead. But it's no longer the only game in town. For every institution wrestling with the 90% who don't respond, this pipeline offers a pragmatic, repeatable bridge to better data.

---

> *If this kind of practical, experience-driven breakdown was useful, there is more where that came from -- check my [YouTube](https://www.youtube.com/@LinliSharesResearch) for more.*

---

← Back to [Data Collection Articles](/Research/Data-Collection/)
