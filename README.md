# brkraw-dataset

This repository hosts example datasets for the BrkRaw project.

## Purpose

The datasets are required for unit tests that verify whether brkraw can convert them and whether orientation is converted correctly.

## Adding a new dataset

Please add new datasets via a Pull Request. Use the brkraw `prune` command with `--anonymize` (brkraw 0.6 or later; it applies the built-in pruner spec `anonymize`, which replaced `deid4share`).

The parameter file generated during pruning is excluded via `.gitignore` (`*.prune.yaml`); keep it locally for your reference.

Use versioned folders when uploading datasets, for example: `PV5.1`, `PV6.0.1`, `PV7.0`, `PV360.3.7`.

Dataset naming convention: `<institution-abbrev>_PV<version>_<included-sequences>`. Place the dataset and its metadata file in the matching version folder.

If needed, include a separate metadata file with institution, author, license, subject, scan details, and other metadata. Add extra comments if needed.

Example:

- Dataset: `UNC_PV5.1_BOLD-EPI_TurboRARE.zip`
- Metadata file: `UNC_PV5.1_BOLD-EPI_TurboRARE.metadata.yaml`

```yaml
institution: Center for Animal MRI, UNC at Chapel Hill
author:
  - Sung-Ho Lee
  - Yen-Yu Ian Shih
license: CC BY 4.0
subject: SD-rat
scans:
  3: FLASH
  4: FieldMap
  9: TurboRARE
  11: EPI
notes: Paravision 5.1 example dataset
comments: Add any additional context if needed
```

## Git LFS

This repository uses Git LFS for large dataset files, and `.gitattributes` is already configured. If Git LFS is not installed on your machine, install it first: https://git-lfs.com/

Clone the repo and initialize Git LFS for your local environment:

```bash
git clone https://github.com/BrkRaw/brkraw-dataset
cd brkraw-dataset
git lfs install
```

Then add your dataset by following the upload naming convention, selecting the correct version folder, and placing the file there.

For example:

```bash
mv UNC_PV6.0.1_FLASH_TurboRARE_EPI.zip PV6.0.1/
git add PV6.0.1/UNC_PV6.0.1_FLASH_TurboRARE_EPI.zip
git commit -m "Add PV6.0.1 example dataset"
```

If you need to add a new file extension for LFS tracking, run:

```bash
git lfs track "*.newext"
git add .gitattributes
```

If you already have a dataset file tracked by Git without LFS, run:

```bash
git lfs migrate import --include="*.zip"
```

## Reduce dataset size for sharing

For sharing, detailed scan-related information should be de-identified, and unnecessary large files should be removed to reduce size. Since this is a test dataset for 2dseq reconstructed image conversion, large rawdata files such as `fid` or `rawdata.job0` are removed. The final dataset should keep only `method`, `acqp`, `visu_pars`, `2dseq`, and `reco`.

brkraw (0.6 or later) ships an example anonymization spec named `anonymize`,
used with `brkraw prune --anonymize`. It keeps only `subject`, `acqp`,
`method`, `reco`, `visu_pars` and `2dseq`, replaces the subject and study IDs,
removes names, operators, institution, dates and UIDs, and strips `$$`
comment lines. It is a starting point, not a guarantee: check the result.

Check the file first with `brkraw info`:

```bash
brkraw info /path/to/dataset.zip
```

See what would change, then write the zip:

```bash
brkraw prune /path/to/dataset.zip --anonymize --subject-id 01exp --study-id 01 \
  -s 3 4 9 11 -r 1 -o /path/to/output.zip --dry-run
brkraw prune /path/to/dataset.zip --anonymize --subject-id 01exp --study-id 01 \
  -s 3 4 9 11 -r 1 -o /path/to/output.zip
brkraw info /path/to/output.zip
```

brkraw 0.5.x used other names (`--spec-name deid4share`, `--scan-ids`,
`--reco-ids`, `--mode keep`); they were removed in 0.6.
