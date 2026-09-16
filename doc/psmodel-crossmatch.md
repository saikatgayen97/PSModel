# Catalogue Cross-Matcher Guide (`psmodel-crossmatch`)

[`scripts/psmodel-crossmatch`](../scripts/psmodel-crossmatch) is a command-line tool that performs positional cross-matching between two astronomical catalogues based on angular separation (search radius).

The tool merges the matched sources into a single new catalogue containing **all columns from both input catalogues** (disambiguated using customizable suffixes, defaulting to `_1` and `_2`, yielding $2 \times$ old columns), plus an optional `match_sep_arcsec` column recording the separation distance.

---

## 1. Directory & File Conventions

In accordance with PSModel developer rules:
* **Catalogues (`.csv`)**: Saved in [`catalogue/`](../catalogue/).
* **Execution & Logs**: Executed from [`workdir/`](../workdir/), with logs saved in [`workdir/logs/`](../workdir/logs/).
* **Scripts**: Located in [`scripts/`](../scripts/).
* **Documentation**: Located in [`doc/`](../doc/).
* **Agent Lock (`agent.lock`)**: Must be `true` when idle.

---

## 2. Command-Line Reference

```bash
../scripts/psmodel-crossmatch <cat1> <cat2> [options]
```

### Arguments & Options

| Option | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `cat1`, `cat2` | `path` | Required | Paths to the two input CSV catalogues. |
| `--radius-arcsec` / `-r` | `float` | `5.0` | Search radius threshold in **arcseconds**. |
| `--radius-arcmin` | `float` | `None` | Search radius threshold in **arcminutes**. |
| `--ra1`, `--dec1` | `str` | Auto-detect | RA and Dec column names in Catalogue 1 (auto-detected from `ra`, `dec`, `RAJ2000`, etc.). |
| `--ra2`, `--dec2` | `str` | Auto-detect | RA and Dec column names in Catalogue 2. |
| `--suffix1` | `str` | `_1` | Column suffix appended to Catalogue 1 columns. |
| `--suffix2` | `str` | `_2` | Column suffix appended to Catalogue 2 columns. |
| `--mode` | `str` | `nearest` | Matching mode: `nearest` (closest match within radius), `mutual` (mutual nearest neighbors), or `all` (all pairs within radius). |
| `--output` / `-o` | `str` | Auto | Output CSV filename/path (defaults to `catalogue/<cat1>_<cat2>_matched.csv`). |
| `--no-sep` | `flag` | `False` | Omit the `match_sep_arcsec` column to output strictly $2 \times$ old columns. |

---

## 3. Usage Examples

All commands are run from `workdir/`:

### Example 1: Basic Cross-Matching (5 arcsec radius)
```bash
cd /Users/sgayen/Documents/Work/PSModel/workdir
../scripts/psmodel-crossmatch \
    ../catalogue/trecs-400-fov-46-mf-0.11-pbeffect_400MHz.csv \
    ../catalogue/trecs-400-fov-46-mf-0.11-pbeffect_610MHz.csv \
    --radius-arcsec 2.0
```

### Example 2: Custom Suffixes and Output Name
```bash
cd /Users/sgayen/Documents/Work/PSModel/workdir
../scripts/psmodel-crossmatch \
    ../catalogue/cat_400MHz.csv \
    ../catalogue/cat_610MHz.csv \
    --radius-arcsec 1.5 \
    --suffix1 _400 \
    --suffix2 _610 \
    -o matched_400_610.csv
```

### Example 3: Mutual Nearest Neighbors
```bash
cd /Users/sgayen/Documents/Work/PSModel/workdir
../scripts/psmodel-crossmatch \
    ../catalogue/cat1.csv \
    ../catalogue/cat2.csv \
    --radius-arcmin 0.1 \
    --mode mutual
```
