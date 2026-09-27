# Business Entity Resolution — Amazon ML Challenge 2026

## 1. Problem Understanding

The objective is to resolve business entities across three sources. Source 1 acts as the reference dataset, while Source 2 and Source 3 contain records that may correspond to the same real-world businesses. Each Source 1 entity can have zero, one, or multiple matching entities across Source 2 and Source 3.

The solution therefore performs candidate generation followed by match prediction. The primary objective is to identify genuine matches while avoiding incorrect merges, since the evaluation uses macro-averaged F0.5, which places greater emphasis on precision.

## 2. Data Preprocessing

The datasets were provided as tab-separated files and were processed using pandas with `sep="\t"`.

The following fields were used:

* `entity_id`
* `business_name`
* `business_address`
* `country`

Text normalization was applied to business names and addresses. The preprocessing included:

* conversion to lowercase
* Unicode normalization using NFKC
* replacement of punctuation and symbols with spaces
* whitespace normalization
* extraction of address numbers
* extraction of a short normalized business-name prefix

Country was retained as an important blocking attribute and was not restricted to a fixed set of countries. This allows the same approach to operate on the additional France records present in the test data.

## 3. Candidate Generation / Blocking

Comparing every Source 1 record with every Source 2 and Source 3 record would result in an impractically large number of comparisons. Therefore, blocking was used to create a smaller candidate set before matching.

The final candidate-generation approach used three country-aware lookup strategies:

1. **Normalized business name + country**

   Records with the same normalized business name within the same country were considered candidates.

2. **Normalized business address + country**

   Records with the same normalized address within the same country were considered candidates.

3. **First address number + business-name prefix + country**

   A combined key was constructed from the first numerical component of the address, a four-character normalized business-name prefix, and country.

Candidate IDs from these strategies were combined and deduplicated for each Source 1 entity.

To avoid uncontrolled candidate expansion from extremely common keys, lookup groups were capped at a maximum of 50 candidate IDs per key.

The final candidate set was written to `candidate_pairs.tsv`. Every Source 1 entity received one candidate-list row, including entities for which no candidates were found.

## 4. Match Prediction

A supervised binary classification approach was developed using labelled Source 1–candidate pairs constructed from the training data.

The model used similarity and matching features including:

* business-name character similarity
* token-sorted business-name similarity
* address character similarity
* token-sorted address similarity
* country equality
* exact-name match
* exact-address match
* name containment
* address containment
* name-length difference
* address-length difference
* first-address-number agreement

An XGBoost binary classifier was trained on positive and negative entity pairs. A group-based train/validation split was used so that Source 1 entities did not overlap between the training and validation groups.

The model demonstrated strong pair-level validation performance on the sampled validation pairs. Threshold experiments were also performed using the competition-style F0.5 calculation at the Source 1 entity level. A probability threshold of 0.15 was selected for the developed matching model based on those experiments.

For the final test-generation pipeline, conservative matching evidence was used to reduce the possibility of false entity merges. Exact normalized-name, exact normalized-address, and strong combined-key evidence were used to determine final predictions.

## 5. Output Generation

Two required files were generated:

### `matching_results.tsv`

Contains:

* `source1_entity_id`
* `matched_entity_ids`

There is exactly one row for every Source 1 entity in the test dataset. Multiple matched IDs are comma-separated, while entities without a predicted match have an empty value.

### `candidate_pairs.tsv`

Contains:

* `source1_entity_id`
* `candidate_entity_ids`

There is exactly one row for every Source 1 entity. Candidate IDs are deduplicated and are restricted to Source 2 and Source 3 entities.

The final files contain **1,732,544 Source 1 rows each**, matching the number of records in the test Source 1 dataset.

## 6. Reproducibility and Implementation

The final pipeline processes the large TSV files in chunks rather than loading all Source 2 and Source 3 records into a single in-memory DataFrame. Lookup structures are constructed incrementally and final outputs are written incrementally.

The submission package contains:

```text
output/
├── matching_results.tsv
└── candidate_pairs.tsv

code/
└── business_entity_resolution/
    ├── README.md
    └── requirements.txt

Documentation_template.md
```

The approach is designed to avoid all-pairs entity comparison and to provide a scalable blocking-based solution for the supplied datasets.
