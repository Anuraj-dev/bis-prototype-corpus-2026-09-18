<h1 align="center">BIS public standards corpus</h1>

<p align="center">
  Catalogue records and Gazette attachments for 24,133 published standards from the Bureau of Indian Standards.<br />
  A snapshot taken on 2026-09-18, as NDJSON files and PDFs for research and prototypes.
</p>

<p align="center">
  <a href="#what-is-in-the-snapshot">Layout</a>
  ·
  <a href="#using-the-data">Use the data</a>
  ·
  <a href="https://standards.bis.gov.in/website">BIS website</a>
</p>

This repo is a point-in-time copy of the public BIS standards catalogue and the Gazette PDFs linked from it. Each catalogue record has a standard number, title, department, committee, type and publication date. Gazette data exists for 13,204 standards, and it points to 12,057 unique PDF URLs.

There is no app and no build step. You read the files with any JSON Lines reader. Before you rely on the data, check the current BIS page, the latest revision and the licensing conditions. The repo has no license file.

## What you get

- **One record per published standard.** `data/standards_metadata.ndjson` has 24,133 rows, one per `standardId`, with `standardNumber`, `standardName`, `departmentName`, `sectionalCommitteeName`, `typeOfStandardName` and `publishedOn`. 23,721 rows have a publication date, from 31 March 1886 to 28 August 2026.
- **The raw Gazette and lookup answers.** `data/standard_documents.ndjson` keeps the BIS API response for each standard under `gazette`, `product_manual`, `corrigendum`, `summary` and `standard_format`. Gazette data appears for 13,204 standards and product manuals for 24. The corrigendum, summary and standard format fields are empty in this snapshot.
- **A manifest of every attachment URL.** `data/files.ndjson` has one row per unique PDF URL, 12,057 in total. Each row has a `status`: `indexed_public_url` for 11,750 rows, `downloaded` for 283 and `error` for 24. The 24 errors are HTTP 404 responses from object storage.
- **Links back to the source standard.** Each file row has `sourceStandardId` and `sourceStandardNumber`. The ID joins to `standardId` in the metadata. Each row also keeps `sourceRef` and the original `url`, so you can trace a PDF back to its source.
- **283 PDFs saved locally.** The `files/` folder holds the downloaded PDFs. Each downloaded row in `data/files.ndjson` records the local `path` and the `bytes`. The 283 files total 242.6 MB.
- **One JSON object per line.** Each line parses on its own, so you can read a file row by row without loading all of it. The three NDJSON files are 18.6 MB, 13.3 MB and 4.5 MB.

## What is in the snapshot

| Path | Records | Contents |
| --- | --- | --- |
| `bis_corpus_manifest.json` | 1 object | Source endpoints, counts, generation time and notes. |
| `data/standards_metadata.ndjson` | 24,133 | Catalogue record per standard. |
| `data/standard_documents.ndjson` | 24,133 | BIS API answers per standard, for Gazette, product manual and other lookups. |
| `data/files.ndjson` | 12,057 | One row per unique PDF URL, with `status` and, for downloads, `path` and `bytes`. |
| `files/` | 359 PDFs | Downloaded PDFs. |

`files/` holds 359 PDFs, but `data/files.ndjson` lists only 283 of them. The other 76 appear by name in `data/standard_documents.ndjson`, but no manifest row records them. Filenames take three forms. `<id>_<timestamp>.pdf` covers 310 files, `gf_*.pdf` covers 33 and `product_manual_*.pdf` covers 16.

## Using the data

The examples use `jq` and run from the repo root.

```sh
# Count standards in the Chemical department
jq -c 'select(.departmentName | startswith("CHEMICAL"))' data/standards_metadata.ndjson | wc -l

# Count standards by type
jq -r '.typeOfStandardName' data/standards_metadata.ndjson | sort | uniq -c | sort -rn

# List the PDFs stored locally
jq -c 'select(.status == "downloaded") | {sourceStandardNumber, path}' data/files.ndjson

# Write the URLs that are not stored locally to a file
jq -r 'select(.status == "indexed_public_url") | .url' data/files.ndjson > urls.txt
```

The same loop works in Python without loading the whole file:

```python
import json

with open("data/files.ndjson", encoding="utf-8") as f:
    for line in f:
        row = json.loads(line)
        if row["status"] == "downloaded":
            print(row["sourceStandardNumber"], row["path"], row["bytes"])
```

The 11,750 `indexed_public_url` rows have no local copy. Fetch them with your own HTTP client.

## How it was collected

The snapshot came from two public BIS endpoints. The catalogue endpoint is `https://standardsadmin.bis.gov.in/proposal-service/getWebsiteIndianStandardsList`, and it returned the 24,133 records. The Gazette endpoint is `https://standardsadmin.bis.gov.in/review-service/getGazettedetails`, and it supplied the attachment links. The manifest timestamp is 2026-09-18 10:49 UTC.

The manifest says a linked PDF was downloaded only when its object-storage URL resolved. The scraper that produced the snapshot is not in this repo, so you cannot rerun the download step from here.

## Limits of the snapshot

- Product manuals appear for only 24 standards, and corrigenda, summaries and standard formats are empty. The BIS service also exposes licensing, laboratory and scheme endpoints, which this snapshot does not include. Crawl them in a separate, rate-limited pass if you need them.
- A catalogue record or attachment URL does not confirm that a standard is current. Check the current BIS page and the latest revision before you rely on it.
- The repo has no license file. Check the BIS licensing conditions before you redistribute the PDFs or data derived from them.
