# Amazon ML Challenge 2026 - Approach

## Problem
Business entity resolution across multiple source datasets.

## Approach
The solution normalizes business names and addresses and uses country-aware
blocking to generate scalable candidate sets. Candidate records are then
filtered using exact normalized entity evidence.

## Candidate Generation
Candidates are generated using multiple blocking keys based on normalized
business name, normalized address, and combined address/name information.

All final predictions are constrained to the generated candidate set.

## Prediction
A conservative evidence-based filtering strategy was used to reduce false
matches. This is particularly important because the evaluation metric
penalizes false positive matches more heavily than missed matches.

## Output
- `output/matching_results.tsv`
- `output/candidate_pairs.tsv`
