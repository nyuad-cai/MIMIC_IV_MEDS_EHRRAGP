> [!IMPORTANT]
> This repository is a customized research extension of the original
> [MIMIC_IV_MEDS](https://github.com/Medical-Event-Data-Standard/MIMIC_IV_MEDS)
> extraction pipeline used in the EHR-RAGp project.
>
> For this fork, please install directly from source using the instructions in:
>
> - [EHR-RAGp Customizations](#ehr-ragp-customizations)

# MIMIC-IV MEDS Extraction ETL

[![PyPI - Version](https://img.shields.io/pypi/v/MIMIC-IV-MEDS)](https://pypi.org/project/MIMIC-IV-MEDS/)
[![codecov](https://codecov.io/gh/Medical-Event-Data-Standard/MIMIC_IV_MEDS/graph/badge.svg?token=E7H6HKZV3O)](https://codecov.io/gh/Medical-Event-Data-Standard/MIMIC_IV_MEDS)
[![tests](https://github.com/Medical-Event-Data-Standard/MIMIC_IV_MEDS/actions/workflows/tests.yaml/badge.svg)](https://github.com/Medical-Event-Data-Standard/MIMIC_IV_MEDS/actions/workflows/tests.yml)
[![code-quality](https://github.com/Medical-Event-Data-Standard/MIMIC_IV_MEDS/actions/workflows/code-quality-main.yaml/badge.svg)](https://github.com/Medical-Event-Data-Standard/MIMIC_IV_MEDS/actions/workflows/code-quality-main.yaml)
![python](https://img.shields.io/badge/-Python_3.11-blue?logo=python&logoColor=white)
[![license](https://img.shields.io/badge/License-MIT-green.svg?labelColor=gray)](https://github.com/Medical-Event-Data-Standard/MIMIC_IV_MEDS#license)
[![PRs](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](https://github.com/Medical-Event-Data-Standard/MIMIC_IV_MEDS/pulls)
[![contributors](https://img.shields.io/github/contributors/Medical-Event-Data-Standard/MIMIC_IV_MEDS.svg)](https://github.com/Medical-Event-Data-Standard/MIMIC_IV_MEDS/graphs/contributors)

This pipeline extracts the MIMIC-IV dataset (from PhysioNet) into the MEDS format.

## Table of Contents

- [Usage](#usage)
- [Examples and More Info](#examples-and-more-info)
- [Expected Runtime and Compute Needs](#expected-runtime-and-compute-needs)
- [Common Issues / FAQ](#-common-issues--faq)
- [EHR-RAGp Customizations](#ehr-ragp-customizations)
- [Installation for This Fork](#installation-for-this-fork)
- [Running the Pipeline](#running-the-pipeline)
- [Relation to Original Repository](#relation-to-original-repository)

## Usage

```bash
export DATASET_DOWNLOAD_USERNAME=$PHYSIONET_USERNAME
export DATASET_DOWNLOAD_PASSWORD=$PHYSIONET_PASSWORD

MEDS_extract-MIMIC_IV root_output_dir=$ROOT_OUTPUT_DIR
```

When you run this, the program will:

1. Download the needed raw MIMIC files for the currently supported version into
    `$ROOT_OUTPUT_DIR/raw_input`.

2. Perform initial, pre-MEDS processing on the raw MIMIC files, saving the results in
    `$ROOT_OUTPUT_DIR/pre_MEDS`.

3. Construct the final MEDS cohort, and save it to
    `$ROOT_OUTPUT_DIR/MEDS_cohort`.

You can also specify the target directories more directly with:

```bash
export DATASET_DOWNLOAD_USERNAME=$PHYSIONET_USERNAME
export DATASET_DOWNLOAD_PASSWORD=$PHYSIONET_PASSWORD

MEDS_extract-MIMIC_IV \
	raw_input_dir=$RAW_INPUT_DIR \
	pre_MEDS_dir=$PRE_MEDS_DIR \
	MEDS_cohort_dir=$MEDS_COHORT_DIR
```

## Examples and More Info

You can run:

```bash
MEDS_extract-MIMIC_IV --help
```

for more information on arguments and options.

You can also run:

```bash
MEDS_extract-MIMIC_IV root_output_dir=$ROOT_OUTPUT_DIR do_demo=True
```

to execute the pipeline on the publicly available MIMIC-IV demo dataset.

## Expected Runtime and Compute Needs

This pipeline can be successfully run over the full MIMIC-IV dataset on a 5-core machine using approximately 165GB of memory in around 7 hours, including dataset download time.

The output folder size is approximately 9.8 GB.

## 🔧 Common Issues / FAQ

### ❓ Ubuntu Symlink Issues During `pre_MEDS`

Some users may encounter symlink-related issues during the `pre_MEDS` stage on Ubuntu systems.

To avoid symlink creation entirely, enable file copying instead:

```bash
MEDS_extract-MIMIC_IV root_output_dir=$ROOT_OUTPUT_DIR do_copy=True
```

______________________________________________________________________

# EHR-RAGp Customizations

This repository contains a research-oriented extension of the original `MIMIC_IV_MEDS` extraction pipeline developed for the EHR-RAGp project.

The modifications introduced in this fork are designed to support retrieval-augmented longitudinal EHR modeling, richer timeline construction, and downstream foundation model training.

## Added Features (pre_MEDS.py)

### 1. Admission-Level Age Computation

The preprocessing pipeline was extended to compute `age_at_admission` for each hospital admission using:

- `anchor_age`
- `anchor_year`
- `admittime`

This enables age-aware longitudinal timeline construction and demographic event encoding for downstream modeling.

### 2. ICU Event Metadata Enrichment

ICU event tables are enriched by joining them with `icu/d_items` using `itemid`.

The following metadata fields are added:

- `category`
- `label`
- `abbreviation`

This enhancement provides semantically meaningful ICU event representations beyond raw numeric item identifiers.

The enrichment is applied to:

- `icu/chartevents`
- `icu/procedureevents`
- `icu/inputevents`
- `icu/outputevents`

## Added Features (event_configs.yaml)

### 3. EHR-RAGp Event Configuration

The event extraction configuration was customized to produce a richer and more model-ready MEDS timeline for EHR-RAGp.

Compared with the original configuration, this fork modifies the event vocabulary and metadata structure to better support longitudinal EHR foundation model training.

Key changes include:

- Renaming hospital and ICU boundary events using explicit EHR-RAGp event names, such as:

    - `ADMISSION-AT-HOSPITAL`
    - `DISCHARGE-FROM-HOSPITAL`
    - `ADMISSION-AT-ICU`
    - `DISCHARGE-FROM-ICU`

- Separating hospital admission attributes into distinct events, including:

    - `ADMISSION-TYPE`
    - `ADMISSION-LOCATION`
    - `DISCHARGE-LOCATION`
    - `AGE_AT_ADMISSION`

- Adding `AGE_AT_ADMISSION` as a numeric event derived during the `pre_MEDS` stage.

- Renaming clinical event families to EHR-RAGp-specific event types:

    - `DIAGNOSIS-ICD`
    - `PROCEDURE-ICD`
    - `LAB`
    - `MICROBIOLOGY`
    - `MEDICATION`
    - `ICU-CHART`
    - `ICU-PROCEDURE`
    - `ICU-INFUSION`
    - `ICU-FLUID-OUTPUT`

- Adding source-table identifiers directly into event codes, such as:

    - `hosp/admissions`
    - `hosp/labevents`
    - `icu/chartevents`
    - `icu/inputevents`

- Enriching events with additional metadata fields needed for downstream modeling, including diagnosis/procedure sequence numbers, lab reference ranges, lab flags, ICU labels, ICU categories, and item abbreviations.

- Including microbiology events through `hosp/microbiologyevents`, which were not present in the original event configuration.

- Simplifying selected event families by disabling some original event definitions, such as ED registration/out events, birth events, OMR events, transfer events, medication stop events, infusion end events, and subject-weight-at-infusion events.

These changes make the extracted MEDS timeline more suitable for EHR-RAGp’s tokenization, care-stage-aware timeline construction, retrieval-oriented chunking, and downstream clinical prediction tasks.

## Installation for This Fork

Clone the repository:

```bash
git clone https://github.com/nyuad-cai/MIMIC_IV_MEDS_EHRRAGP.git
cd MIMIC_IV_MEDS_EHRRAGP
```

Install locally:

```bash
pip install -e .
```

## Running the Pipeline

Set your PhysioNet credentials:

```bash
export DATASET_DOWNLOAD_USERNAME=$PHYSIONET_USERNAME
export DATASET_DOWNLOAD_PASSWORD=$PHYSIONET_PASSWORD
```

Run the extraction pipeline:

```bash
HYDRA_FULL_ERROR=1 MEDS_extract-MIMIC_IV \
	root_output_dir=$ROOT_OUTPUT_DIR \
	do_copy=True
```

The pipeline will:

1. Download the required MIMIC-IV files
2. Perform customized `pre_MEDS` preprocessing
3. Construct the MEDS cohort

The `do_copy=True` option is recommended to avoid symlink-related issues on some systems.

## Relation to Original Repository

Original repository:

- https://github.com/Medical-Event-Data-Standard/MIMIC_IV_MEDS

This repository is an independent research extension intended for EHR-RAGp experiments and is not an official MEDS release.
