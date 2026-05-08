# Getting Started with CHAP-Core – For Master Students

A structured, pedagogical introduction to:

1. Working with a model and a metric **in isolation**, without CHAP
2. Integrating the same logic into **CHAP-Core**
3. *(Optional, later)* connecting it to the **DHIS2 Modeling App**

The goal is for you to **understand what is happening**, not just to run
code.


## Learning objectives

By the end of this tutorial you will be able to:

- Explain what CHAP-Core is and what it is used for
- Run a model and a metric in isolation, without CHAP
- Understand how predictions and observations are represented
- Implement a simple evaluation metric in pandas
- Understand how the same metric is wrapped to be CHAP-compatible
- Prepare a model for integration into CHAP


## Prerequisites

The tutorial requires:

- Python 3.10 or newer
- A terminal (Terminal, iTerm, PowerShell)
- Git

The tutorial does **not** require:

- Pre-installed Python libraries
- Prior knowledge of CHAP, DHIS2, or MLflow


---

## 1. Setup on a blank machine with `uv`

This tutorial uses [**`uv`**](https://docs.astral.sh/uv/) to:

- Create and manage the virtual environment
- Install dependencies quickly
- Pin versions reproducibly in `uv.lock`

We use the modern `pyproject.toml` flow (not `requirements.txt`), because
that is what `uv sync` is built for and the recommended Python packaging
standard today.

### 1.1 Why `pyproject.toml` + `uv.lock`?

`pyproject.toml` is a declarative list of the packages your project
needs. `uv.lock` is an exact version lock that lets the environment be
**reproduced byte-for-byte** on another machine later.

Concretely:

- All students get the same versions → less debugging
- You can reproduce results from your thesis years from now
- You avoid "works on my machine, not on yours"

### 1.2 Why `uv sync` instead of `pip install`?

`uv sync`:

- Creates a virtual environment automatically (`.venv/`)
- Installs everything declared in `pyproject.toml`
- Regenerates `uv.lock` when needed

Students need **a single command** to set everything up — no need to
explain `python -m venv`, activation, and `pip install` as separate
steps.

### 1.3 Step by step

#### 1.3.1 Install `uv` (once per machine)

**macOS / Linux:**

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

**Windows (PowerShell):**

```powershell
irm https://astral.sh/uv/install.ps1 | iex
```

Verify:

```bash
uv --version
```

#### 1.3.2 Create the project

Move into the directory where you want your tutorial project, and run:

```bash
uv init chap-tutorial
cd chap-tutorial
```

`uv init` creates `pyproject.toml`, `.python-version`, `README.md`, and
an empty `main.py`.

#### 1.3.3 Add dependencies

```bash
uv add chap-core pandas numpy pyyaml scikit-learn
```

Each `uv add` call:

- Adds the package to `pyproject.toml`
- Installs it into `.venv/`
- Updates `uv.lock`

To add more later, the command is the same: `uv add <package>`. To
remove: `uv remove <package>`.

#### 1.3.4 Sync the environment

If you clone a project that already has `pyproject.toml` and `uv.lock`,
all you need is:

```bash
uv sync
```

This recreates `.venv/` with the exact versions in the lock file.

#### 1.3.5 Run a Python script

```bash
uv run python isolated_metric.py
```

`uv run` executes the command inside the virtual environment without
requiring you to activate it manually.


---

## 2. The isolated phase (before using CHAP)

In this section we work **completely without CHAP-Core**.

The goal is to:

- Understand how a model and a metric work in practice
- See exactly what the input, computation, and output are
- Test and debug the logic without complex infrastructure

> **Important:** in this phase we use plain Python and pandas only.
> No CHAP code is used until section 5.

### 2.1 What does "isolated" mean in practice?

"Isolated" means we work with:

- CSV files
- pandas DataFrames
- ordinary Python functions

We do **not** use:

- `chap-core`
- `MetricBase`
- `MetricSpec`
- the CHAP CLI

This way you can verify the math yourself, test small changes quickly,
and understand *what CHAP later automates for you*.

### 2.2 What do we build in isolation?

A single evaluation metric based on:

- predictions (`forecast`)
- observations (`disease_cases`)

returning the **absolute error** per `location` and `time_period`.

### 2.3 Why is this phase important before CHAP?

Without it, you risk that:

- CHAP feels "magical"
- Errors become hard to debug
- You cannot tell whether the problem lies in the model, the metric,
  or the integration

With an isolated phase, you can always say:

> "I know the math works — if something fails now, it is the
> integration."


---

## 3. Background: how evaluation works in CHAP

When a model is evaluated in CHAP, predictions are produced indexed on
four dimensions:

- `location` — e.g. a district
- `time_period` — e.g. a week
- `horizon_distance` — how far ahead the prediction is for
- `sample` — index of a stochastic sample from the predictive
  distribution

Predictions are represented as a flat DataFrame:

```text
location  time_period  horizon_distance  sample  forecast
loc1      2023-W01     1                 0       10
loc1      2023-W02     2                 0       12
```

Observations have the same shape but without `horizon_distance` and
`sample`:

```text
location  time_period  disease_cases
loc1      2023-W01     11.0
loc1      2023-W02     13.0
```

This flat representation is the contract between the model and the
evaluation metric — both in isolation and inside CHAP.


---

## 4. Isolated example: implement a simple metric

### 4.1 The code

```python
import pandas as pd

forecasts = pd.DataFrame(
    {
        "location": ["loc1", "loc1", "loc2", "loc2"],
        "time_period": ["2023-W01", "2023-W02", "2023-W01", "2023-W02"],
        "horizon_distance": [1, 2, 1, 2],
        "sample": [1, 1, 1, 1],
        "forecast": [10, 12, 21, 23],
    }
)

observations = pd.DataFrame(
    {
        "location": ["loc1", "loc1", "loc2", "loc2"],
        "time_period": ["2023-W01", "2023-W02", "2023-W01", "2023-W02"],
        "disease_cases": [11.0, 13.0, 19.0, 21.0],
    }
)


def my_metric(forecasts, observations):
    merged = forecasts.merge(
        observations,
        on=["location", "time_period"],
        how="left",
    )
    merged["metric"] = (merged["forecast"] - merged["disease_cases"]).abs()
    return merged[["location", "time_period", "metric"]]


print(my_metric(forecasts, observations))
```

Save this as `isolated_metric.py` and run it with:

```bash
uv run python isolated_metric.py
```

### 4.2 Expected output

```text
  location time_period  metric
0     loc1    2023-W01     1.0
1     loc1    2023-W02     1.0
2     loc2    2023-W01     2.0
3     loc2    2023-W02     2.0
```

The model was off by 1 in `loc1` and by 2 in `loc2`. The metric is
computed per `location` and `time_period`, exactly as CHAP expects.


---

## 5. Bridging to a CHAP-compatible metric (conceptually)

Once the metric works in isolation, it is wrapped for CHAP by:

- Defining a class that inherits from `MetricBase`
- Implementing `compute()` with the same logic as `my_metric` above
- Defining a `MetricSpec` that declares:
  - which dimensions the output has (`time_period`, `location`, …)
  - the metric name and id
  - a short description

> **Key point:**
> The math does **not change** — only how it is packaged.
> This is the bridge between the isolated phase and the CHAP integration:
> the same `forecasts.merge(...).abs()` chain lives on inside a class
> that CHAP can discover and run automatically.


---

## 6. Starting point: clone `minimalist_example`

In this tutorial we do **not** start from an empty project. We clone
[`minimalist_example`](https://github.com/dhis2-chap/minimalist_example)
and use it as the base structure, the working directory, and later as
the entry point into CHAP.

The repo already contains:

- `train.py` — training code
- `predict.py` — prediction code
- `MLproject` — the CHAP integration file
- example data

### 6.1 Clone the repo

```bash
git clone https://github.com/dhis2-chap/minimalist_example
cd minimalist_example
```

### 6.2 Install the dependencies

`minimalist_example` ships with a `requirements.txt`. You can import it
straight into a modern `uv` setup:

```bash
uv init
uv add -r requirements.txt
uv sync
```

`uv add -r` reads `requirements.txt` and writes the packages into a
freshly created `pyproject.toml` with a corresponding `uv.lock`. From
this point on, you use `uv run python …` as usual.

### 6.3 First phase: use the repo **without** CHAP

In this phase you should:

- Use `train.py` and `predict.py` as ordinary Python code
- Read and write CSV files manually
- Test the logic with `uv run python train.py` and
  `uv run python predict.py`
- Implement and test your metric in isolation

You should **not** use CHAP, and you should **not** worry about
`MLproject` yet.

### 6.4 Where does the isolated metric fit in?

Once you have `my_metric(forecasts, observations)`, you can apply it
directly to the CSV output from `predict.py` to verify that:

- The predictions are sensible
- The error behaves as expected (not all errors = 0, no extreme
  outliers, …)

This still happens **without CHAP**. At this point you have a working
end-to-end flow: train → predict → evaluate, all in plain Python.


---

## 7. Important rules

- Always clone `minimalist_example` first
- Always work in isolation inside the repo before integrating with CHAP
- Use CHAP **only** once everything works in isolation
- Never start directly with CHAP
- Use `uv sync` / `uv add` for all installation — never manual
  `pip install`


---

## 8. Next steps

Once you have completed this tutorial, you are ready to:

- Implement your own metric
- Test it in isolation inside the repo
- Wrap it as a `MetricBase` subclass in CHAP
- Run it through `MLproject` and the CHAP CLI
- Use it further in the DHIS2 Modeling App

This is a complete development cycle:
**idea → isolated test → CHAP-Core → DHIS2 Modeling App**.
