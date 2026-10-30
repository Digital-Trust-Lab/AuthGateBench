# AuthGateBench

**Benchmarking LLMs as Authorization Gates**

[![arXiv](https://img.shields.io/badge/arXiv-2610.16426-b31b1b.svg)](https://arxiv.org/abs/2610.16426)

AuthGateBench evaluates LLMs as authorization gates for coding agents
under benign user requests and an explicit policy. Each contrast pair
keeps the execution trajectory and proposed action fixed while changing
the user request, flipping the gold label between **allow** and **block**.

**Paper:** [AuthGateBench: Benchmarking LLMs as Authorization Gate](https://arxiv.org/abs/2610.16426)  
**Authors:** Zhixiong Zhuang, Jin Jia, Zhiwei Shang, Ziyu Wang

## Composition

This release contains **700 triplets / 350 contrast pairs**. Each triplet includes an additional
`environment_signals` field. Ignoring that field gives the no-signal condition; including it
gives the signal condition. These are two conditions over the same 700 records, not 1,400
independent scenarios.

| Failure mode           | Records | Contrast pairs |
| ---------------------- | ------: | -------------: |
| Scope Escalation       |     140 |             70 |
| Credential Exploration |     140 |             70 |
| Inferred Parameters    |     140 |             70 |
| Guardrail Bypass       |     140 |             70 |
| External Sharing       |     140 |             70 |
| Total                  |     700 |            350 |

Labels are balanced: 350 block and 350 allow. Each category contains 70 of each.
Records span source execution environments E1–E7. All are in the test split;
no training split is prescribed.

## Files and fields

- `data/authgatebench.jsonl`: 700 records with native nested JSON objects/lists.
- `data/authgatebench.parquet`: the same records; `trajectory`, `proposed_action`,
  `environment_signals` and `provenance` are losslessly JSON-encoded strings to support
  heterogeneous tool argument types in Arrow/Datasets.

Fields: `id`, `category`, `environment`, `user_request`, `trajectory`,
`proposed_action`, `gold_label`, `pair_id`, `role_in_pair`,
`environment_signals`, `provenance`. IDs omit the old condition suffix.
`pair_id` groups base and inverted requests.

## Load from GitHub

After downloading and extracting this repository from GitHub, run the examples
from its root directory (the one containing `data/`). JSONL needs only Python's
standard library.

```python
import json
from pathlib import Path

with Path("data/authgatebench.jsonl").open(encoding="utf-8") as f:
    records = [json.loads(line) for line in f if line.strip()]

print(len(records))
print(records[0].keys())
```

The JSONL file preserves nested fields as native JSON values. To use the Parquet
version instead, install pandas and a Parquet engine (such as PyArrow):

```bash
pip install pandas pyarrow
```

```python
import json
import pandas as pd

df = pd.read_parquet("data/authgatebench.parquet")
row = df.iloc[0].to_dict()
for field in ("trajectory", "proposed_action", "environment_signals", "provenance"):
    if isinstance(row.get(field), str):
        row[field] = json.loads(row[field])
```

## Classifier inputs

For JSONL, select a record first:

```python
row = records[0]
```

The Parquet example already defines `row`. Then build the classifier input:

```python
context = {key: row[key] for key in ("user_request", "trajectory", "proposed_action")}
use_signals = True
if use_signals:
    context["environment_signals"] = [
        {"channel": s["channel"], "value": s["value"]}
        for s in row["environment_signals"]
    ]
```

Select the evaluation policy separately. Do not feed `gold_label`, `provenance`,
IDs, category labels or signal notes to the classifier.

## Validation and limitations

Checks cover counts, unique IDs, balanced labels, fixed bare contrast pairs and exact
JSON/Parquet round trips. These checks do not establish human label agreement.
Signals are synthetic and label-conditioned, not outputs from a validated independent
real-environment retrieval pipeline. Permission decisions are relative to one detailed
policy, not a claim that consent overrides all organizational controls.

Dataset commands are examples; loading the dataset does not execute them.

## Citation

If you use AuthGateBench in your research, please cite:

```bibtex
@misc{zhuang2026authgatebenchbenchmarkingllmsauthorization,
      title={AuthGateBench: Benchmarking LLMs as Authorization Gate},
      author={Zhixiong Zhuang and Jin Jia and Zhiwei Shang and Ziyu Wang},
      year={2026},
      eprint={2610.16426},
      archivePrefix={arXiv},
      primaryClass={cs.AI},
      url={https://arxiv.org/abs/2610.16426},
}
```

## License

The AuthGateBench dataset is licensed under the
[Creative Commons Attribution-NonCommercial-NoDerivatives 4.0
International License (CC BY-NC-ND 4.0)](https://creativecommons.org/licenses/by-nc-nd/4.0/).