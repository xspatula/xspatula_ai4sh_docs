---
title: "ML Pre-processing 1: Delete Dataframes"
layout: single
sidebar:
  nav: "machine_learning"
excerpt: "Deleting the redundant pre-processed dataframes left by exploring pre-processing options, while keeping the raw datasets and the pipelines you need."
permalink: /machine_learning/ml_preprocess1/delete/
author_profile: false
date: 2026-10-30 08:00:00 +0100
last_modified_at: 2026-10-31 08:00:00 +0100
---

Section 3 of the notebook `ml_preprocess_separated.ipynb`. Exploring pre-processing options leaves many
dataframes that you no longer need once you have chosen your pipeline. The process `delete_dataframes`
deletes the pre-processed dataframes (parquet + companion JSON) that you do not need to keep, and removes them from
`.previous_dataframe`. See [ML pre-processing 1][pp1] for naming and the companion JSON.

## 3 Delete redundant dataframes

**Process file**: `project_neospectra/process/ml_preprocess/spectra_delete_dataframes.json`

```json
{
  "process": [
    {
      "process": "delete_dataframes",
      "parameters": {
        "project_root_fp": "./project_neospectra/data/spectra/neospectra/ds2/ai4sh",
        "keep_array": ["raw", "previous"]
      }
    }
  ]
}
```

| Parameter (`delete_dataframes`) | Description |
|---|---|
| `project_root_fp` | The folder to clean up |
| `keep_array` | The dataframes to keep; default (and empty) = `["raw", "previous"]` |

`keep_array` takes:

- `raw`: the selected (not pre-processed) datasets. These are never deleted anyway.
- `previous`: the dataframe(s) of the latest `.previous_dataframe` record, i.e. the one you saved last.
- the names of pre-processed dataframes to keep (`.parquet` optional), e.g.
  `"1350-2500_10.snv.sg5.d1.mc.wc40.pca5"`.

### What is deleted

- Only **pre-processed** dataframes are candidates: files with a pre-processing suffix, or with a companion
  JSON that has a non-empty `preprocessing_chain`. Selected (raw) datasets are never deleted.
- A name in `keep_array` that does not exist stops the process **before anything is deleted**
  (`ERROR: dataframe to keep not found: <name> — nothing deleted.`). A typo therefore cannot cause the
  dataframe you meant to keep to be deleted.
- The process lists what is kept and what will be deleted, with the total size, and asks before deleting:

```
    Delete these <n> dataframes (parquet + companion JSON)? [y/n]:
```

- `y` deletes the parquet and companion JSON of each listed dataframe. The deleted files, and any files
  already missing, are removed from `.previous_dataframe`.
- Plots (`plot/`) and model results are not deleted.

## Is it safe to delete intermediate dataframes?

Yes, for a kept pipeline. The companion JSON of the final dataframe records the complete chain and the
raw dataset. [ML modeling][ml_model] refits the chain from that raw
dataset and does not read the intermediate files (see [Exploration versus modelling][explore]). Keep the raw dataset and the final dataframe with its
companion JSON. With [chain_from][chain_from] the same pipeline can also be run again at any time.

## Next step

[ML modeling][ml_model]: train regressors on the pre-processed dataframe.

[pp1]: {{ site.baseurl }}/machine_learning/ml_preprocess1/
[pp1_chain]: {{ site.baseurl }}/machine_learning/ml_preprocess1/chain/
[explore]: {{ site.baseurl }}/machine_learning/ml_preprocess1/#exploration-versus-modelling
[chain_from]: {{ site.baseurl }}/machine_learning/ml_preprocess1/chain/#273-load-existing-pre-processing-chain
[ml_model]: {{ site.baseurl }}/machine_learning/ml_model/
