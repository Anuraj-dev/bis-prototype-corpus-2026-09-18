# BIS public corpus snapshot

Generated 2026-09-18 from the public BIS website/API.

## Contents

- `bis_corpus_manifest.json` — source endpoints, counts, and file locations.
- `data/standards_metadata.ndjson` — 24,133 published-standard catalogue records.
- `data/standard_documents.ndjson` — one Gazette attachment response per catalogue record.
- `data/files.ndjson` — 12,057 deduplicated public Gazette URLs, with provenance; 283 local PDFs were downloaded during this run and the rest are indexed as public URLs.
- `files/` — downloaded public attachments.

NDJSON is one JSON object per line and is suitable for streaming ingestion into a vector/database pipeline. Every attachment record retains `sourceStandardId`, `sourceStandardNumber`, `sourceRef`, and the original public URL.

The snapshot covers catalogue metadata and Gazette attachments. The BIS service also exposes product-manual, corrigendum, summary, standard-format, licensing, laboratory, and scheme endpoints; those should be crawled in a separate rate-limited pass if needed. Do not treat catalogue metadata or attachment URLs as a substitute for checking the current BIS page, licensing conditions, or the latest standard revision.

To resume local file downloads for the indexed URLs, run:

```sh
BIS_DOWNLOAD_FILES=1 python scripts/scrape_bis.py
```
