# Privacy Sensitivity Annotation Dataset for Social Media Images

This repository contains annotation labels and pre-computed image embeddings for a privacy sensitivity annotation dataset of social media images.

The dataset is designed to support research on multidimensional image privacy sensitivity. Instead of treating image privacy only as a matter of identifiability, the dataset provides annotations along multiple privacy-relevant dimensions, including identifiability, location sensitivity, and activity sensitivity.

This repository has been prepared for double-anonymous review. It does not include author names, affiliations, personal repository links, institution-specific paths, or other identifying information.

## Overview

The dataset contains 4,999 image records.

Due to the sensitive nature of the image content, this repository does not include original images. Instead, we provide:

- annotation labels for each image record
- pre-computed CLIP ViT-B/16 image embeddings
- documentation for the annotation schema and released files

The released data can be used for downstream analysis, model training, privacy sensitivity prediction, and reproducibility checks without requiring access to the original images.

## Repository Structure

```text
.
├── README.md
├── data/
│   ├── privacy_sensitivity_annotation.csv
│   └── clip_vitb16_embeddings.npz
└── appendix/
    └── full_privacy_keyword_table.pdf
```

If the appendix file is not included in this repository, the full privacy keyword table is provided in the supplementary appendix of the submitted paper.

## Data Files

### `data/privacy_sensitivity_annotation.csv`

This file contains the released annotation table.

Each row corresponds to one image record. The `image_id` column is used to match each annotation record with the corresponding image embedding in `clip_vitb16_embeddings.npz`.

The CSV contains the following columns:

| Column | Description |
|---|---|
| `image_id` | Image identifier. This corresponds to an entry in the `image_ids` array of `clip_vitb16_embeddings.npz`. |
| `information_type` | Multi-label binary vector indicating which types of privacy-relevant information are present in the image. |
| `identifiability` | Ordinal privacy sensitivity score for whether a specific individual can be identified. |
| `location_sensitivity` | Ordinal privacy sensitivity score for the potential privacy harm of disclosing the depicted location. |
| `activity_sensitivity` | Ordinal privacy sensitivity score for the potential privacy harm, stigma, or risk associated with the depicted or implied activity. |
| `blur_presence` | Binary label indicating whether intentional privacy-protective modification is visible, such as blurring, mosaicking, stickers, or covering. |
| `sharing_allowance` | Multi-label binary vector indicating the audience tiers with whom the image could appropriately be shared. |
| `high_level_category` | High-level image category used for dataset organization and analysis. |
| `subcategory` | More specific image subcategory. |

### `data/clip_vitb16_embeddings.npz`

This file contains pre-computed CLIP ViT-B/16 image embeddings.

The file contains two arrays:

| Array | Shape | Description |
|---|---:|---|
| `image_ids` | `(4999,)` | Image identifiers. |
| `embeddings` | `(4999, 512)` | CLIP ViT-B/16 image embeddings. |

The `i`-th row of `embeddings` corresponds to the image identifier `image_ids[i]`.

The `image_ids` array in `clip_vitb16_embeddings.npz` matches the `image_id` column in `privacy_sensitivity_annotation.csv`.

## Annotation Schema

### Privacy Sensitivity Dimensions

Each image record is associated with three ordinal privacy sensitivity dimensions:

| Dimension | Description |
|---|---|
| `identifiability` | Whether a specific individual can be identified from the image. |
| `location_sensitivity` | Whether the depicted location may reveal privacy-sensitive information. |
| `activity_sensitivity` | Whether the depicted or implied activity may reveal privacy-sensitive, stigmatizing, risky, or socially sensitive information. |

Each dimension uses a 0--2 ordinal scale:

| Value | Meaning |
|---:|---|
| `0` | Low sensitivity |
| `1` | Moderate sensitivity |
| `2` | High sensitivity |

### Information Type

The `information_type` field is represented as an 8-dimensional binary vector.

The vector positions correspond to the following information types:

| Position | Information type |
|---:|---|
| 1 | Personal identity and sensitive information |
| 2 | Social relations |
| 3 | Location information |
| 4 | Body and appearance |
| 5 | Personal preferences |
| 6 | Socio-cultural sensitive information |
| 7 | Risk and crime-related information |
| 8 | Other |

For example:

```text
[1,0,1,0,1,0,0,0]
```

indicates that the image contains personal identity or sensitive information, location information, and personal preferences.

### Sharing Allowance

The `sharing_allowance` field is represented as a 6-dimensional binary vector.

The vector positions correspond to the following audience tiers:

| Position | Audience tier |
|---:|---|
| 1 | No sharing |
| 2 | Close relations |
| 3 | Normal relations |
| 4 | Acquaintance |
| 5 | Public |
| 6 | Broadcast |

For example:

```text
[0,1,1,0,0,0]
```

indicates that sharing is allowed with close relations and normal relations, but not with broader audiences.

## Gold Label Aggregation

The released labels are gold labels aggregated from two independent annotators.

The aggregation rules are as follows:

| Field | Aggregation rule |
|---|---|
| `information_type` | Union rule |
| `identifiability` | Maximum score |
| `location_sensitivity` | Maximum score |
| `activity_sensitivity` | Maximum score |
| `blur_presence` | Union rule |
| `sharing_allowance` | Intersection rule |

These rules were used to construct one final annotation record per image.

## Missing Values

Some annotation fields contain missing values.

Missing values indicate cases where a final aggregated label was unavailable for that field. They do not indicate the absence of the corresponding attribute.

The released CSV contains 4,999 image records. The number of missing values in each field is as follows:

| Field | Number of missing values |
|---|---:|
| `identifiability` | 8 |
| `location_sensitivity` | 12 |
| `activity_sensitivity` | 11 |
| `blur_presence` | 61 |

The following fields contain no missing values:

- `image_id`
- `information_type`
- `sharing_allowance`
- `high_level_category`
- `subcategory`

For experiments, rows with missing labels should be excluded only for the specific target being evaluated. For example, when evaluating `identifiability`, only rows with missing `identifiability` should be excluded.

## Loading the Data

The annotation CSV can be loaded with `pandas`:

```python
import pandas as pd

annotations = pd.read_csv("data/privacy_sensitivity_annotation.csv")

print(annotations.head())
print(annotations.shape)
```

The embedding file can be loaded with `numpy`:

```python
import numpy as np

emb = np.load("data/clip_vitb16_embeddings.npz")

image_ids = emb["image_ids"]
embeddings = emb["embeddings"]

print(image_ids.shape)
print(embeddings.shape)
```

To match annotations with embeddings:

```python
import numpy as np
import pandas as pd

annotations = pd.read_csv("data/privacy_sensitivity_annotation.csv")
emb = np.load("data/clip_vitb16_embeddings.npz")

image_ids = emb["image_ids"]
embeddings = emb["embeddings"]

id_to_index = {image_id: i for i, image_id in enumerate(image_ids)}

# Example: get the embedding for the first annotated image
image_id = annotations.loc[0, "image_id"]
embedding = embeddings[id_to_index[image_id]]

print(image_id)
print(embedding.shape)
```

## Example Use Case

A common use case is to train a classifier that predicts one of the privacy sensitivity dimensions from image embeddings.

For example, to train a model for `identifiability`, users should remove only the rows where `identifiability` is missing, while keeping the rest of the dataset:

```python
import numpy as np
import pandas as pd

target = "identifiability"

annotations = pd.read_csv("data/privacy_sensitivity_annotation.csv")
emb = np.load("data/clip_vitb16_embeddings.npz")

image_ids = emb["image_ids"]
embeddings = emb["embeddings"]
id_to_index = {image_id: i for i, image_id in enumerate(image_ids)}

df = annotations.dropna(subset=[target]).copy()
y = df[target].astype(int).values

X = np.stack([
    embeddings[id_to_index[image_id]]
    for image_id in df["image_id"]
])

print(X.shape)
print(y.shape)
```

The same procedure can be used for `location_sensitivity` and `activity_sensitivity`.

## Privacy Keyword Table

The supplementary appendix includes the full keyword-based privacy-relevant information taxonomy used for hashtag-based retrieval.

The taxonomy includes high-level privacy-relevant categories, subcategories, and keywords or visual cues. It was used to guide data collection and dataset coverage.

## Data Release Policy

This repository does not include original images.

The original images are not publicly released due to the sensitive nature of the image content. The released dataset consists of annotation labels and pre-computed image embeddings.

The embeddings are provided to support reproducibility and downstream analysis while reducing the privacy risks associated with releasing original images.

## Ethical Considerations

This dataset concerns privacy-sensitive visual content.

Users should not attempt to reconstruct, identify, or infer the original images or individuals from the released embeddings or annotations.

Users should use the dataset only for research purposes related to privacy, safety, image understanding, human-centered AI, or related areas.

Users should not use this dataset for surveillance, re-identification, profiling, or other applications that may increase privacy risks.

## Notes for Double-Anonymous Review

This repository is provided for double-anonymous review.

Author-identifying information, including author names, affiliations, personal repository links, institution-specific paths, and contact information, has been omitted.

Citation, license, and contact information will be added after publication.

## Citation

Citation information will be added after publication.

For double-anonymous review, author-identifying citation information is omitted from this repository.

## License

License information will be added after review.

During double-anonymous review, this repository is provided only for reviewer access and reproducibility inspection.

## Contact

Contact information will be added after publication.

For double-anonymous review, author-identifying contact information is omitted.
