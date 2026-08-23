# Repository Map and Handoff Notes

Last audited: 2026-08-23

## Current State

This repository is very small. The tracked project contents are:

- `README.md`: high-level project description.
- `TextAttack.ipynb`: the only executable experiment artifact currently in the repo.
- `pyproject.toml`: dependency manifest for recreating the environment with `uv`.
- `artifact_lineage.csv`: machine-readable map of files, scripts, outputs, and dependencies.

The Git working tree was clean at audit time, and `main` matched `origin/main` at commit `95ca4f2` (`got toxicity and sentiment attacks done`).

## Can This Instance Be Deleted?

The repo itself can be recreated from GitHub without losing tracked files, assuming the remote remains accessible:

```bash
git clone https://github.com/TongmengXie/adversarial_robustness.git
cd adversarial_robustness
```

However, deletion of this instance will remove local machine state that is not tracked by Git:

- Python/Conda environments, including the previously used `steering-env`.
- Hugging Face model cache files.
- NLTK data at paths such as `/home/tom/nltk_data`.
- Any untracked files outside this repository.

At audit time, no standalone raw output CSV/JSON/JSONL/Parquet files were found inside this repository. The only run outputs found here are embedded in `TextAttack.ipynb` as notebook cell outputs.

## Raw Outputs

Raw outputs currently live inside `TextAttack.ipynb`.

Important embedded results:

- Sentiment attack:
  - Model: `textattack/roberta-base-imdb`
  - Attack recipe: `TextFoolerJin2019`
  - Input: `The movie was absolutely wonderful!`
  - Adversarial output shown in notebook: `The photographer was absolutely sumptuous!`
  - Result summary: 1 successful attack out of 1, average queries 98.
- Toxicity bypass demo:
  - Primary toxicity model: `unitary/toxic-bert`
  - Fallback toxicity model: `martin-ha/toxic-comment-model`
  - Input: `You are not intelligent and show no motivations.`
  - Four simple perturbation methods were attempted: punctuation, whitespace, leetspeak, unicode.
  - Embedded result summary: 0/4 bypass success rate.

If future runs need auditable raw outputs, modify the notebook or convert it to a script that writes timestamped files under a tracked or backed-up directory such as `outputs/`.

## Script and File Roles

`TextAttack.ipynb` has three code cells:

1. Installs/checks `nltk` and `tensorflow_hub`, then downloads the NLTK `averaged_perceptron_tagger_eng` resource.
2. Loads `textattack/roberta-base-imdb`, builds a TextFooler attack, attacks one sentiment example, and prints the attack result.
3. Defines `CompactAdversarialDemo`, loads sentiment and toxicity models, runs simple toxicity perturbation checks, and prints a summary.

There are no standalone Python scripts in the repo yet.

## Dependencies

Use `pyproject.toml` as the environment source of truth. The versions were inferred from the Conda environment recorded in the notebook outputs:

- Python 3.11
- `textattack==0.3.10`
- `transformers==4.51.3`
- `torch==2.7.0`
- `tensorflow==2.19.0`
- `tensorflow-hub==0.16.1`
- `tf-keras==2.19.0`
- `nltk==3.9.1`
- `numpy==2.1.3`
- `pandas==2.2.3`

Recommended setup on a new GPU instance:

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
uv sync --locked
uv run python -m nltk.downloader averaged_perceptron_tagger_eng
uv run jupyter lab
```

Keep `uv.lock` committed and use `uv sync --locked` for reproducible installs.

## Target Models and Judge Consistency

The notebook currently uses fixed Hugging Face model names, but it does not pin model revisions or record dataset checksums. For strict reproducibility across judges or target models, future runs should record:

- Hugging Face model revision/commit SHA.
- Random seeds for Python, NumPy, PyTorch, and TextAttack.
- CUDA, PyTorch, TensorFlow, driver, and GPU details.
- Raw result files, not only notebook-rendered output.
- Exact attack configuration and input dataset file.

## Suggested New Instance Size

For the current notebook-scale workload:

- Minimum practical GPU: 8 GB VRAM.
- Comfortable GPU: 16 GB VRAM.
- Stronger/recommended GPU if expanding experiments: 24 GB VRAM or more.
- Minimum disk: 50 GB.
- Comfortable disk: 100 GB.
- Recommended disk if caching multiple Hugging Face models/datasets: 200 GB.

The current tracked repository is under 1 MB, but model caches and environments can easily consume tens of GB.
