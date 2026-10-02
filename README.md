# Multi-Domain Human and AI Text Dataset

The dataset contains 150,912 English texts: 18,864 human documents and
132,048 AI-generated counterparts. Each of 12 source families contributes
1,572 human documents. Each human document is paired with one generated
text from each of seven models.

The source families are BBC, CMUBookSummary, CMV, CNN_Dailymail,
GPABenchmark_written, IMDb, KaggleCompetition (PERSUADE 2.0 standalone
assignments), NarrativeQA, Newsroom, PoemsDataset, SQuAD, and Yelp.
They cover news, reviews, scientific abstracts, argumentative essays,
book and document summaries, poetry, and encyclopedia passages.

The generators are `deepseek-r1:14b`, `gemini-3.7-flash`, `gemma4:12b`,
`gpt-5.6-luna`, `llama3.1:8b`, `ministral-3:14b`, and `qwen3:14b`.

## Files

- `data/*.parquet`: one file per source family, with 12,576 records per
  file, identical schemas, and Zstandard compression.
- `multi-domain-text-dataset.jsonl.gz`: the same records in UTF-8,
  gzip-compressed JSON Lines format.
- `sources.csv`: source families and public source references.
- `statistics.csv`: record counts by source family, label, and model.
- `schema.json`: field names and data types.
- `validation.json`: corpus integrity and coverage checks.
- `SHA256SUMS`: release-file SHA-256 checksums.

## Fields

| Field | Description |
|---|---|
| `id` | Unique public text identifier. |
| `pair_id` | Public identifier linking a human document and its seven generated counterparts. |
| `source_group_id` | Groups records that share the same human source text; keep groups together when partitioning the dataset. |
| `source_dataset` | Source family. |
| `source_subset` | Original source subset, where available; this is not a benchmark partition. |
| `domain` | Broad genre annotation. |
| `subdomain` | More specific category, or null. |
| `text` | Human or AI-generated text. |
| `label` | `human` or `ai`. |
| `generator_model` | Generator model identifier; null for human records. |
| `provider` | Generation provider; null for human records. |
| `prompt` | Exact prompt used to generate this AI text; null for human records. |
| `word_count` | Number of whitespace-delimited words in `text`. |
| `target_word_count` | Requested generation length; null for human records. |
| `text_sha256` | SHA-256 of `text` encoded as UTF-8. |
| `prompt_sha256` | SHA-256 of `prompt` encoded as UTF-8; null for human records. |

The identifiers are opaque strings. Use `pair_id` to join human and AI
records; use `source_group_id` when assigning evaluation partitions.
Prompts can differ between models for the same human document, so use the
prompt on the individual AI record.

## Loading and evaluation

From the dataset directory:

```python
from datasets import load_dataset

corpus = load_dataset(
    "parquet",
    data_files={"corpus": "data/*.parquet"},
    split="corpus",
)
human = corpus.filter(lambda row: row["label"] == "human")
ai = corpus.filter(lambda row: row["label"] == "ai")
```

The `corpus` name refers to the entire collection, not a training allocation.
No official train/validation/test split is supplied. Keep each source group and its
generated counterparts together when creating partitions.

The overall human-to-AI ratio is 1:7. Comparing human records with one
generator at a time gives balanced classes. Some prompts contain source
metadata or summaries; generated counterparts are not necessarily
paraphrases of the human documents.

## Integrity

From the dataset directory, verify the file checksums with:

```bash
sha256sum -c SHA256SUMS
```

The Parquet and JSON Lines copies contain equivalent records. Content hashes
refer to the exact published strings; word counts use whitespace separation.
