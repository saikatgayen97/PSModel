# PSModel Developer Rules & Workspace Conventions

This document defines the development rules, directory structure, and execution standards for the `PSModel` project. All developers, scripts, pipelines, and AI assistants working in this repository must strictly adhere to these rules.

---

## 1. Directory Structure & File Placement Rules

| Data / Artifact Type | Directory | Rule & Constraints |
| :--- | :--- | :--- |
| **Data Files** | `data/` | All binary arrays, simulation matrices, and astronomical images in `.npy`, `.npz`, `.fits`, or `.fit` format must be saved in `data/`. No raw data files may be written to the root directory. |
| **Catalogues** | `catalogue/` | All source catalogues, tabular outputs, and cross-matched tables in `.csv` (as well as `.tsv`, `.txt`, `.vot`) format must be saved in `catalogue/`. |
| **Documentation** | `doc/` | All user guides, design specifications, scientific notes, READMEs, and technical documentation must be stored in `doc/`. |
| **Python Scripts** | `scripts/` | All Python scripts (`.py`), utility modules, CLI programs, and wrapper scripts must be placed in `scripts/`. |
| **Plots & Visualizations** | `plots/` | All generated plots, figures, charts, diagnostics, and images (`.png`, `.pdf`, `.svg`, `.eps`) must be output directly to `plots/`. |
| **Working Directory & Logs** | `workdir/` | All commands, simulation runs, tests, and pipeline executions must be executed from `workdir/`. All runtime logs, stdout/stderr captures, and batch run logs must be written to `workdir/logs/`. |
| **Developer Rules** | `developer/` | All project conventions, developer guidelines, coding standards, and architectural rules must reside in `developer/`. |
| **Releases** | `release/` | The `release/` directory must contain the `VERSION` file (storing the current semantic version number) and all packaged release tarballs (`.tar.gz`, `.tar.bz2`). |

---

## 2. Command Execution Standard

* **Current Working Directory**:
  Whenever invoking command-line tools, simulations, or Python scripts, the working directory must be set to `workdir/`:
  ```bash
  cd <PSModel_root>/workdir
  ```
* **Relative Path Conventions**:
  Scripts executed from `workdir/` should reference directories using relative paths:
  * Input data: `../data/<filename>`
  * Catalogues: `../catalogue/<filename>`
  * Scripts: `../scripts/<script_name>.py`
  * Plots output: `../plots/<figure_name>.png`
  * Execution logs: `logs/<run_name>.log`

---

## 3. Release & Versioning Standard

* The file `release/VERSION` tracks the canonical semantic version (e.g. `X.Y.Z`).
* Packaged distribution tar files must follow the naming pattern:
  ```text
  release/PSModel-v<VERSION>.tar.gz
  ```
* Creating a new release archive should package the repository according to these standards while excluding temporary logs and cache files.

---

## 4. Agent Lock Protocol (`agent.lock`)

* The repository root contains an `agent.lock` file.
* **State Values**:
  * `true`: Lock is free / agent is idle (not modifying files).
  * `false`: Lock is acquired / agent is actively modifying files.
* **Workflow**:
  1. **Check**: Before an agent starts modifying files, it reads `agent.lock`.
  2. **Acquire**: If `agent.lock` is `true`, the agent proceeds and sets `agent.lock` to `false`.
  3. **Release**: Once file modifications and operations are finished, the agent sets `agent.lock` back to `true`.

