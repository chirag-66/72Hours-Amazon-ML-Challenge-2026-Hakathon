# Amazon ML Challenge 2026 — Business Entity Resolution

> **Machine Learning Challenge | Amazon ML Challenge 2026**
> **Problem:** Business Entity Resolution / Record Linkage
> **Task:** Identify matching business entities across multiple data sources.

---

## 📌 Project Overview

This repository contains our solution for the **Amazon ML Challenge 2026**, focused on the problem of **Business Entity Resolution**.

The objective is to identify which business records from different sources refer to the **same real-world business entity**, despite differences in business names, addresses, formatting, spelling, abbreviations, and other textual variations.

Unlike a conventional supervised classification problem where each row directly contains a target label, this challenge requires constructing a matching system capable of discovering relationships between records from different sources.

Our solution follows a multi-stage entity-resolution pipeline:

```text
Raw Business Data
       ↓
Data Cleaning & Normalization
       ↓
Candidate Pair Generation
       ↓
Name Similarity
       +
Address Similarity
       +
Country / Supporting Features
       ↓
Candidate Matching
       ↓
Entity-Level Predictions
       ↓
matching_results.tsv
```

---

# 🎯 Problem Statement

The challenge provides business records originating from different sources.

Records representing the same business may not be identical because of:

* Different spellings
* Abbreviations
* Punctuation differences
* Different capitalization
* Address formatting variations
* Missing information
* Additional words or tokens
* Different representations of the same business

For example:

```text
Source A:
Starbucks Coffee, 123 Main Street

Source B:
STARBUCKS COFFEE
123 Main St.
```

Although the records are textually different, they may represent the same real-world entity.

The goal is therefore to determine the correspondence between entities across the provided sources.

---

# 📂 Dataset Structure

The challenge data used in our solution contains the following major files/data structures.

## Source 1 — S1

Contains business entities from the first source.

| Column             | Description                        |
| ------------------ | ---------------------------------- |
| `entity_id`        | Unique identifier of the business  |
| `business_name`    | Name of the business               |
| `business_address` | Business address                   |
| `country`          | Country associated with the entity |

Example:

```text
entity_id | business_name       | business_address | country
----------|---------------------|------------------|--------
12345     | Starbucks Coffee    | Main Street      | US
```

---

## Source 2 — S2

Contains matching relationships associated with Source 1.

| Column               | Description              |
| -------------------- | ------------------------ |
| `source1_entity_id`  | Entity ID from Source 1  |
| `matched_entity_ids` | Corresponding entity IDs |

---

## Source 3 — S3

Contains another set of business records.

| Column             | Description       |
| ------------------ | ----------------- |
| `entity_id`        | Unique identifier |
| `business_name`    | Business name     |
| `business_address` | Business address  |
| `country`          | Country           |

The key challenge is determining which S3 entities correspond to entities in S1.

---

# 🧠 Why This Problem Is Different From Typical ML Datasets

This is not a straightforward:

```text
X → y
```

classification problem.

Instead, the problem can be represented as:

```text
S1 Entity
   ↓
Generate possible S3 candidates
   ↓
Compare each candidate
   ↓
Calculate similarity
   ↓
Determine whether candidates represent
the same real-world business
   ↓
Return matched entity IDs
```

A simplified candidate-level representation can be thought of as:

| Feature            | Example |
| ------------------ | ------: |
| Name similarity    |    0.92 |
| Address similarity |    0.81 |
| Country match      |       1 |
| Overall similarity |    0.87 |
| Match              |       1 |

The final output, however, must be converted back into the challenge's required **entity-level format**.

---

# 🔬 Solution Approach

Our solution was designed as a multi-stage entity-resolution pipeline.

## 1. Data Understanding

We first inspected the structure and schema of the supplied files.

The main columns identified were:

```python
S1:
['entity_id', 'business_name', 'business_address', 'country']

S2:
['source1_entity_id', 'matched_entity_ids']

S3:
['entity_id', 'business_name', 'business_address', 'country']

Ground Truth:
['source1_entity_id', 'matched_entity_ids']
```

Understanding these relationships was important because the challenge's target is not simply a single binary column.

---

## 2. Text Preprocessing

Business names and addresses can contain significant formatting differences.

We therefore normalize textual fields before comparison.

Typical preprocessing operations include:

* Lowercasing
* Removing unnecessary punctuation
* Normalizing whitespace
* Standardizing textual representations
* Handling missing values
* Removing formatting inconsistencies

Example:

```text
"Starbucks Coffee, Inc."
```

may be normalized into a representation closer to:

```text
starbucks coffee inc
```

This allows similarity algorithms to focus more on the actual content rather than formatting.

---

# 🔎 3. Candidate Pair Generation

Directly comparing every S1 business with every S3 business would be computationally expensive.

Instead, the pipeline generates a smaller set of **candidate pairs**.

Conceptually:

```text
S1
 │
 ├── Entity A ──────┐
 ├── Entity B ──────┼── Candidate Generation
 ├── Entity C ──────┤
 └── Entity D ──────┘
                    ↓
               Candidate Pairs
                    ↓
                  Matching
```

Candidate generation significantly reduces the number of comparisons that need to be evaluated.

The generated candidate dataset contains information necessary for comparing potential matches.

---

# 🧮 4. Similarity Features

Potential matches are evaluated using textual and entity-level features.

Important signals include:

### Business Name Similarity

Business names provide one of the strongest signals for identifying potential matches.

Examples of useful comparison concepts include:

* Token overlap
* Character-level similarity
* Normalized string similarity
* Partial matching
* Exact matching after normalization

---

### Address Similarity

Addresses can vary considerably between sources.

For example:

```text
123 Main Street, New Delhi
```

and

```text
123 Main St New Delhi
```

may refer to the same location.

Address similarity therefore provides an additional signal alongside business-name similarity.

---

### Country Consistency

Country information can be used as a supporting feature.

If two records belong to different countries, the probability of them representing the same entity may be significantly reduced.

---

# 🔗 5. Entity Matching

The candidate-level similarity signals are combined to determine likely entity correspondences.

Conceptually:

```text
Name Similarity
       +
Address Similarity
       +
Country Information
       ↓
Matching Score
       ↓
Candidate Ranking / Decision
       ↓
Matched Entity IDs
```

The system must preserve the relationship between the original entity IDs throughout this process.

---

# 📊 6. Validation

Validation is particularly important in this challenge because the final output is evaluated externally.

The development process considers:

* Candidate generation quality
* Matching accuracy
* False matches
* Missed matches
* Duplicate predictions
* Output formatting
* Entity-ID consistency

Where appropriate, candidate-level validation can be used to test whether the matching logic is identifying meaningful relationships before producing the final submission.

---

# 📤 7. Final Submission

The final prediction is generated in the required TSV format.

The primary prediction file is:

```text
matching_results.tsv
```

The expected structure is:

```text
source1_entity_id    matched_entity_ids
```

Example:

```text
12345    67890
12346    67891,67892
12347
```

The exact formatting of the final file follows the challenge submission requirements.

---

# 📦 Submission Package

The final submission package prepared for the challenge contains:

```text
README
requirements
src/
documentation/
matching_results.tsv
candidate_pairs.tsv
```

The final ZIP prepared for submission was approximately:

```text
260 MB
```

Large intermediate/generated datasets are intentionally not included in this GitHub repository where unnecessary.

---

# 🗂️ Repository Structure

```text
amazon-ml-challenge-2026/
│
├── README.md
├── requirements.txt
├── .gitignore
│
├── src/
│   ├── preprocessing.py
│   ├── candidate_generation.py
│   ├── matching.py
│   └── inference.py
│
├── notebooks/
│   └── exploration.ipynb
│
├── documentation/
│   └── approach.md
│
└── results/
    └── README.md
```

> File names may differ slightly depending on the final implementation.

---

# ⚙️ Technologies Used

### Programming

* Python

### Data Processing

* Pandas
* NumPy

### Machine Learning / Similarity

* Scikit-learn
* Text similarity techniques
* Feature-based matching

### Development

* Jupyter Notebook
* VS Code
* Git
* GitHub

### Data Format

* CSV
* TSV

---

# 🚀 Running the Project

## 1. Clone the Repository

```bash
git clone https://github.com/chirag-66/amazon-ml-challenge-2026.git
cd amazon-ml-challenge-2026
```

## 2. Create a Virtual Environment

Windows:

```bash
python -m venv .venv
```

Activate it:

```bash
.venv\Scripts\activate
```

---

## 3. Install Dependencies

```bash
pip install -r requirements.txt
```

---

## 4. Prepare the Dataset

Place the challenge-provided datasets in the appropriate local data directory.

For example:

```text
data/
├── source1/
├── source2/
└── source3/
```

> The original challenge datasets are not included in this repository because of their size and challenge-specific distribution restrictions.

---

## 5. Run the Pipeline

Run the preprocessing and candidate-generation stages first:

```bash
python src/preprocessing.py
python src/candidate_generation.py
```

Then execute the matching/inference pipeline:

```bash
python src/matching.py
python src/inference.py
```

The final prediction should be generated as:

```text
matching_results.tsv
```

---

# 📈 Challenge Result

The solution was submitted and evaluated on the Amazon ML Challenge platform.

Latest recorded evaluation:

| Submission                 | Date        |     Score | Status    |
| -------------------------- | ----------- | --------: | --------- |
| Final evaluated submission | 28 Sep 2026 | **0.342** | Evaluated |

The score shown above represents the latest recorded submission at the time this repository was prepared.

---

# 🧪 Lessons From the Challenge

This challenge provided practical experience with a problem that differs significantly from standard tabular ML competitions.

### Key learning areas

* Understanding large-scale entity-resolution problems
* Working with multiple related datasets
* Designing candidate-generation strategies
* Text normalization
* Business-name matching
* Address similarity
* Reducing computational complexity
* Working with very large intermediate datasets
* Creating submission-ready TSV files
* Validating entity-level predictions
* Managing large files and memory constraints
* Structuring an end-to-end ML pipeline

---

# 💡 Future Improvements

Potential improvements to the current system include:

### 1. Better Candidate Generation

Use more sophisticated blocking strategies to reduce unnecessary comparisons while maintaining high recall.

### 2. Advanced Text Features

Introduce additional features such as:

* Character n-gram similarity
* Token-based similarity
* TF-IDF similarity
* Jaccard similarity
* Edit distance
* Phonetic similarity

### 3. Learned Matching Model

Train a dedicated pairwise classification model using labeled entity pairs.

For example:

```text
Name similarity
Address similarity
Country match
Token overlap
Character similarity
        ↓
ML classifier
        ↓
Match / Non-match
```

### 4. Ensemble Matching

Combine multiple similarity models and matching signals to improve robustness.

### 5. Scalability

Further optimize the pipeline for very large datasets through:

* Efficient blocking
* Vectorization
* Batch processing
* Memory-efficient data structures
* Parallel processing

---

# 👥 Team

This project was developed as part of the **Amazon ML Challenge 2026**.

**Participant / Developer**

**Chirag Thakur**

GitHub:
https://github.com/chirag-66

LinkedIn:
https://linkedin.com/in/chirag-thakur-6583ab248

---

# 🏆 About the Challenge

This project was developed for the **Amazon ML Challenge 2026**, with the goal of applying machine-learning and data-processing techniques to a real-world business entity-resolution problem.

The challenge provided an opportunity to work with large-scale datasets and build an end-to-end solution from:

```text
Problem Understanding
        ↓
Data Exploration
        ↓
Preprocessing
        ↓
Candidate Generation
        ↓
Entity Matching
        ↓
Validation
        ↓
Submission
```

---

# ⚠️ Data & Submission Disclaimer

The original challenge datasets and evaluation infrastructure are not included in this repository.

This repository is intended to document the **approach, implementation, experiments, and learning outcomes** from the challenge.

Challenge-specific files should be obtained and used according to the official Amazon ML Challenge rules and terms.

---

# 📜 License

This repository is intended primarily for educational, portfolio, and project-documentation purposes.

Please refer to the Amazon ML Challenge rules regarding the use and redistribution of challenge-provided datasets and materials.

---

## ⭐ Acknowledgement

Participating in the Amazon ML Challenge 2026 provided practical exposure to **entity resolution, record linkage, large-scale text processing, candidate generation, and machine-learning based matching systems**.

The project demonstrates the process of taking a complex real-world data problem and developing a structured computational solution from raw data through final submission.
