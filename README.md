# Supplementary Material

This supplementary package provides additional materials for the submitted paper.

The materials are provided for double-anonymous review only.

## Contents

This package contains the following files and directories:

- `privacy_sensitivity_annotation.csv`  
  The released annotation table for the image privacy sensitivity dataset.

- `clip_vitb16_embeddings.npz`  
  Pre-computed CLIP ViT-B/16 image embeddings for all images.

- `sample_annotations.csv`  
  Annotation labels for the representative sample images included for reviewer verification.

- `sample_images/`  
  A small set of de-identified representative images. These images are included only to help reviewers inspect the annotation scheme and annotation quality.

## Data Release Policy

Due to the sensitive nature of the image content, we do not publicly release the original images.

Upon acceptance, we plan to release the dataset in the form of pre-computed image embeddings and annotation labels. The original images will not be included in the public release.

The representative sample images included in this supplementary material are provided only for reviewer verification during the review process. They are not part of the planned public dataset release.

## Anonymization

The representative sample images have been de-identified before inclusion in this supplementary material.

Where applicable, visible faces and other identifying information have been anonymized, including name tags, ID cards, license plates, screen contents, and location-revealing signs.

Metadata associated with the sample images has been removed.

## Annotation CSV

The file `privacy_sensitivity_annotation.csv` contains annotation labels for 4,999 image records.

Each row corresponds to one image. The `image_id` column is used to match annotations with the CLIP embeddings in `clip_vitb16_embeddings.npz`.

The CSV contains the following columns:

- `image_id`  
  Image identifier. This identifier corresponds to an entry in the `image_ids` array of `clip_vitb16_embeddings.npz`.

- `information_type`  
  Multi-label binary vector indicating which types of privacy-relevant information are present in the image.  
  The vector has 8 positions in the following order:

  1. personal identity and sensitive information
  2. social relations
  3. location information
  4. body and appearance
  5. personal preferences
  6. socio-cultural sensitive information
  7. risk and crime-related information
  8. other

  For example, `[1,0,1,0,1,0,0,0]` indicates that the image contains personal identity or sensitive information, location information, and personal preferences.

- `identifiability`  
  Ordinal privacy sensitivity score for whether a specific individual can be identified.  
  Values are `0`, `1`, or `2`.

- `location_sensitivity`  
  Ordinal privacy sensitivity score for the potential privacy harm of disclosing the depicted location.  
  Values are `0`, `1`, or `2`.

- `activity_sensitivity`  
  Ordinal privacy sensitivity score for the potential privacy harm, stigma, or risk associated with the depicted or implied activity.  
  Values are `0`, `1`, or `2`.

- `blur_presence`  
  Binary label indicating whether intentional privacy-protective modification is visible, such as blurring, mosaicking, stickers, or covering.  
  Values are `0` or `1`. Missing values indicate cases where a final blur label was unavailable.

- `sharing_allowance`  
  Multi-label binary vector indicating the audience tiers with whom the image could appropriately be shared.  
  The vector has 6 positions in the following order:

  1. no sharing
  2. close relations
  3. normal relations
  4. acquaintance
  5. public
  6. broadcast

  For example, `[0,1,1,0,0,0]` indicates that sharing is allowed with close relations and normal relations, but not with broader audiences.

- `high_level_category`  
  High-level image category used for dataset organization and analysis.

- `subcategory`  
  More specific subcategory within the high-level category.

## Annotation Scale

The three ordinal privacy sensitivity labels use the following scale:

- `0`: low sensitivity
- `1`: moderate sensitivity
- `2`: high sensitivity

These ordinal labels apply to the following columns:

- `identifiability`
- `location_sensitivity`
- `activity_sensitivity`

## Gold Label Aggregation

The released annotation labels are gold labels aggregated from two independent annotators.

The aggregation rules are as follows:

- `information_type`: union rule
- `identifiability`: maximum score
- `location_sensitivity`: maximum score
- `activity_sensitivity`: maximum score
- `blur_presence`: union rule
- `sharing_allowance`: intersection rule

These rules were used to construct one final annotation record per image.

## Missing Values

Some annotation fields contain missing values.

Missing values indicate cases where a final aggregated label was unavailable for that field. They do not indicate the absence of the corresponding attribute.

The released CSV contains 4,999 image records. The number of missing values in each field is as follows:

- `identifiability`: 8 missing values
- `location_sensitivity`: 12 missing values
- `activity_sensitivity`: 11 missing values
- `blur_presence`: 61 missing values

The following fields contain no missing values:

- `image_id`
- `information_type`
- `sharing_allowance`
- `high_level_category`
- `subcategory`

For experiments, rows with missing labels should be excluded only for the specific target being evaluated. For example, when evaluating `identifiability`, only rows with missing `identifiability` should be excluded.

## Embeddings

The file `clip_vitb16_embeddings.npz` contains pre-computed CLIP ViT-B/16 image embeddings.

The file contains the following arrays:

- `image_ids`: image identifiers with shape `(4999,)`
- `embeddings`: CLIP image embeddings with shape `(4999, 512)`

The `i`-th row of `embeddings` corresponds to the image identifier `image_ids[i]`.

The `image_ids` array in `clip_vitb16_embeddings.npz` matches the `image_id` column in `privacy_sensitivity_annotation.csv`.

These embeddings are provided to support reproducibility and downstream analysis without requiring access to the original images.

## Representative Samples

The representative samples are selected to cover diverse combinations of privacy sensitivity labels, including cases where identifiability is low but location or activity sensitivity is high.

These examples are intended to help reviewers understand the annotation criteria and inspect the annotation quality. They should not be interpreted as representing the full dataset distribution.

The annotations for these representative samples are provided in `sample_annotations.csv`.

## Double-Anonymous Review

This supplementary package has been prepared for double-anonymous review.

It does not include author names, affiliations, personal repository links, institution-specific paths, or other identifying information.