# Privacy Sensitivity Annotation Dataset for Social Media Images

This repository contains annotation labels and pre-computed image embeddings for the accompanying submitted paper.

The dataset provides annotations for 4,999 Korean social media images along three independent ordinal privacy sensitivity dimensions: identifiability, location sensitivity, and activity sensitivity. It also includes privacy-relevant information types, blur presence, sharing allowance judgments, and image category labels.

Due to the sensitive nature of the image content, we do not release the original images. Instead, we release annotation labels and pre-computed CLIP ViT-B/16 image embeddings. For reviewer reference only, this repository also includes 10 anonymized sample images with corresponding annotations.

This repository has been prepared for double-anonymous review. Author names, affiliations, contact information, personal repository links, and institution-specific paths are omitted.

## Repository Contents

```text
├── README.md
├── privacy_sensitivity_annotation.csv     # 4,999 annotation records
├── clip_vitb16_embeddings.npz             # CLIP ViT-B/16 embeddings, shape (4999, 512)
├── sample_annotation.csv                  # 10 sample annotations for reviewer reference
└── sample_images/                         # 10 anonymized sample images for reviewer reference
```

## Data

**TL;DR:** The annotation table is `privacy_sensitivity_annotation.csv`, and the image embeddings are `clip_vitb16_embeddings.npz`. Match records by `image_id`.

### Annotation Table

Each row in `privacy_sensitivity_annotation.csv` corresponds to one image record.

The columns are:

```text
image_id              : Image identifier matching image_ids in the embeddings file.
information_type      : 8-dimensional binary vector of privacy-relevant information types.
identifiability       : Ordinal score (0–2) for individual identifiability.
location_sensitivity  : Ordinal score (0–2) for privacy harm of disclosing the depicted location.
activity_sensitivity  : Ordinal score (0–2) for privacy harm of the depicted or implied activity.
blur_presence         : Binary label for the presence of intentional privacy-protective modification.
sharing_allowance     : 6-dimensional binary vector of audience tiers with whom sharing is appropriate.
high_level_category   : High-level retrieval category used during data collection.
subcategory           : More specific retrieval subcategory.
```

### Image Embeddings

`clip_vitb16_embeddings.npz` contains two arrays:

```text
image_ids   : shape (4999,), image identifiers matching the image_id column.
embeddings  : shape (4999, 512), CLIP ViT-B/16 image embeddings.
```

The `i`-th row of `embeddings` corresponds to `image_ids[i]`.

### Sample Images for Reviewer Reference

To help reviewers understand the annotation schema, we include 10 representative sample images in `sample_images/` along with their annotations in `sample_annotation.csv`.

The samples illustrate diverse combinations of sensitivity scores across the three dimensions.

Visible faces and identifying cues in these samples have been anonymized where applicable. These images are provided solely for reviewer verification and are not part of the publicly released dataset.

The columns of `sample_annotation.csv` follow the same schema as `privacy_sensitivity_annotation.csv`.

## Annotation Schema

### Ordinal Sensitivity Dimensions

Each sensitivity dimension uses a 0–2 ordinal scale:

```text
0 : low sensitivity
1 : moderate sensitivity
2 : high sensitivity
```

The three dimensions are:

```text
identifiability       : Whether a specific individual can be identified.
location_sensitivity  : Whether the depicted location may reveal sensitive information.
activity_sensitivity  : Whether the depicted or implied activity may carry stigma or privacy risk.
```

### Information Type

The `information_type` field is represented as an 8-dimensional binary vector.

The vector positions correspond to:

```text
INFORMATION_TYPE = {
    0: "Personal identity and sensitive information",
    1: "Social relations",
    2: "Location information",
    3: "Body and appearance",
    4: "Personal preferences",
    5: "Socio-cultural sensitive information",
    6: "Risk and crime-related information",
    7: "Other"
}
```

For example:

```text
[1,0,1,0,1,0,0,0]
```

indicates that the image contains personal identity or sensitive information, location information, and personal preferences.

### Sharing Allowance

The `sharing_allowance` field is represented as a 6-dimensional binary vector.

The vector positions correspond to:

```text
SHARING_ALLOWANCE = {
    0: "No sharing",
    1: "Close relations",
    2: "Normal relations",
    3: "Acquaintance",
    4: "Public",
    5: "Broadcast"
}
```

For example:

```text
[0,1,1,0,0,0]
```

indicates that sharing is allowed with close relations and normal relations, but not with broader audiences.

## Gold Label Aggregation

Each released label is a gold label aggregated from two independent annotators following a shared annotation guideline.

The aggregation rules are:

```text
information_type      : union, logical OR
identifiability       : maximum score
location_sensitivity  : maximum score
activity_sensitivity  : maximum score
blur_presence         : union, logical OR
sharing_allowance     : intersection, logical AND
```

These rules were used to construct one final annotation record per image.

## Missing Values

Some annotation fields contain missing values, reflecting cases where a final aggregated label was unavailable.

```text
identifiability       : 8 missing values
location_sensitivity  : 12 missing values
activity_sensitivity  : 11 missing values
blur_presence         : 61 missing values
```

Missing values do not indicate the absence of the corresponding attribute.

When evaluating a specific target, only rows missing that target should be excluded. For example, when evaluating `identifiability`, only rows with missing `identifiability` should be excluded.

## Loading the Data

```python
import numpy as np
import pandas as pd

annotations = pd.read_csv("privacy_sensitivity_annotation.csv")
emb = np.load("clip_vitb16_embeddings.npz")

image_ids = emb["image_ids"]
embeddings = emb["embeddings"]

id_to_index = {img: i for i, img in enumerate(image_ids)}

print(annotations.shape)
print(image_ids.shape)
print(embeddings.shape)
```

Example: training a classifier on `identifiability`.

```python
target = "identifiability"

df = annotations.dropna(subset=[target]).copy()
y = df[target].astype(int).values

X = np.stack([
    embeddings[id_to_index[img]]
    for img in df["image_id"]
])

print(X.shape)
print(y.shape)
```

The same procedure applies to `location_sensitivity` and `activity_sensitivity`.

## Data Release Policy

Original images are not released.

The released dataset consists of annotation labels and pre-computed CLIP ViT-B/16 image embeddings. The 10 anonymized sample images included in `sample_images/` are provided solely for reviewer reference and will not be part of the publicly released dataset.

This release strategy is intended to support reproducibility and downstream analysis while reducing the privacy risks associated with releasing original images.

## Ethical Use

This dataset is intended for research on privacy, image understanding, human-centered AI, and related areas.

Users should not attempt to reconstruct, identify, or infer the original images or individuals from the released embeddings or annotations.

Users should not use this dataset for surveillance, re-identification, profiling, or other applications that may increase privacy risks.

## Anonymous Review

This repository is provided for double-anonymous review.

Author names, affiliations, contact information, personal repository links, and institution-specific paths are omitted.

Citation and license information will be added after publication.
