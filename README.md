# -XRD-[Uploading README.md…]()
# materials-plot

**Publication-quality figures from materials characterisation data — with the numbers extracted alongside the picture.**

把材料测试数据（XRD、FTIR、DSC、TGA、拉伸、EIS、CV、电池循环……）从 txt/csv/xlsx 变成
可直接投稿的图，同时把峰位、结晶尺寸、弹性模量、分解温度这些数值一起算好给你。

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Python 3.9+](https://img.shields.io/badge/python-3.9%2B-blue.svg)](https://www.python.org/)
[![Agent Skill](https://img.shields.io/badge/Agent%20Skill-compatible-6f42c1.svg)](#use-as-an-agent-skill)

---

## The problem it solves

You ran the test. The instrument spat out a `.txt` with three lines of metadata,
a tab-separated table, and a column header in Chinese. You need a figure that
looks like the ones in your group's papers — white background, inward ticks,
Times New Roman, error bars, the right column width so it drops into the
manuscript at 100 % — and you need it again next week for a different sample.

`materials-plot` turns that into one command and a JSON recipe you can re-run.

```bash
python scripts/mplot.py raw/LFP.txt raw/LFP-C.txt -t xrd -l "LFP" "LFP/C" \
    -o figures/fig2a -f png,pdf --preset journal --journal elsevier --columns 1 \
    --normalize max --baseline linear --smooth 9 --label-peaks
```

You get `figures/fig2a.pdf`, `figures/fig2a.png`, and `figures/fig2a.report.md`
containing every peak position, d-spacing and Scherrer size.

---

## Gallery

> Regenerate every image below with `python scripts/make_examples.py`.

| | |
| --- | --- |
| ![XRD overlay](examples/xrd_overlay.png) | ![XRD stacked](examples/xrd_stacked.png) |
| **XRD** — overlay with reference stick pattern and Scherrer sizes | **XRD** — stacked with (hkl) labels |
| ![FTIR](examples/ftir_stacked.png) | ![Raman](examples/raman_stacked.png) |
| **FTIR** — ALS baseline, stacked, labelled bands | **Raman** — GO vs rGO, stacked |
| ![DSC](examples/dsc.png) | ![TGA](examples/tga_dtg.png) |
| **DSC** — Tg marked, melting peak with extrapolated onset | **TGA + DTG** — twin axis, onset marked, residual labelled |
| ![Tensile](examples/tensile_curves.png) | ![Tensile bar](examples/tensile_uts_bar.png) |
| **Tensile** — stress-strain with σ_y and σ_b annotated | **Tensile** — bar comparison across composites |
| ![EIS](examples/eis_nyquist.png) | ![EIS Bode](examples/eis_bode.png) |
| **EIS** — Nyquist, equal aspect | **EIS** — Bode, log frequency |
| ![CV](examples/cv_scanrates.png) | ![Cycling](examples/cycling.png) |
| **CV** — multiple scan rates + Randles-Ševčík panel | **Cycling** — capacity and Coulombic efficiency |
| ![XPS](examples/xps_c1s.png) | ![UV-Vis](examples/uvvis.png) |
| **XPS** — high-resolution C 1s with components | **UV-Vis** — band-edge comparison |

---

## Features

**Fifteen plot types.** XRD · FTIR · UV-Vis · Raman · NIR · XPS · DSC · TGA/DTG ·
tensile stress-strain · EIS Nyquist & Bode · CV · battery cycling · generic X-Y.

**Reads whatever the instrument wrote.** Metadata blocks, tab/comma/semicolon/
space delimiters, three-line headers, units in parentheses, Excel sheets, ragged
rows. Run `--inspect` and see what it found before you plot.

**Origin-style by default.** White canvas, inward mirrored ticks on all four
spines, minor ticks, Times New Roman, unframed legend, Origin's incremental
colour order. Presets for `journal`, `thesis`, `ppt`, `poster`.

**Exact journal geometry.** `--journal elsevier --columns 1` gives a 3.35 in
(90 mm) figure so it drops into the template at 100 % with no rescaling.

**Exports PNG · TIFF · PDF · EPS · SVG · JPEG** at 600 dpi, with
`pdf.fonttype: 42` so vector text stays editable at proof stage.

**Computes the numbers, not just the picture.** Peak positions, FWHM, d-spacing,
Scherrer crystallite size, ALS/linear baseline subtraction, Savitzky-Golay
smoothing, DSC onset/peak temperatures and ΔH, TGA onset and residual mass,
Young's modulus with an explicit fit range, 0.2 % offset yield strength, UTS,
elongation at break, toughness, EIS Rs/Rct, CV peak potentials, first-cycle
Coulombic efficiency and capacity retention — all written to a reproducible
report.

**Colour-blind and greyscale safe.** Palettes: `origin`, `journal`, `okabe-ito`,
`grayscale`, or your own hex list. Markers cycle independently of colours.

**Says when it is unsure.** Every renderer appends caveats — that a baseline
correction made intensities relative, that a modulus fit range was auto-detected,
that an EIS Rs is geometry-read rather than circuit-fitted. Those notes go into
the report so they reach the caption.

**Origin bridge.** `origin_bridge.py` writes Origin-import-ready CSVs plus a
LabTalk scaffold on any machine, and can drive a local Origin through `originpro`
when one is installed.

---

## Install

```bash
git clone https://github.com/<you>/materials-plot.git
cd materials-plot
pip install -r requirements.txt
```

`requirements.txt`: `numpy`, `pandas`, `matplotlib`, `scipy`, `openpyxl`, `pillow`.

`scipy` is optional — without it, smoothing and baseline correction fall back to
moving averages and linear fits, and peak detection uses a simpler local-maximum
search. Install it.

Nothing else is needed. **Origin is not required.**

---

## Quick start

### 1. See what is in the file

```bash
python scripts/mplot.py --inspect raw/sample.txt
```

```
File      : /abs/path/raw/sample.txt
Rows      : 1751
Columns   : 2
Delimiter : '\t'
Header    : '  TwoTheta(Deg)         Intensity(CPS)'

  #  name                 unit          n          min          max
  0  TwoTheta             Deg        1751           10           80
  1  Intensity            CPS        1751        38.44       184.6
```

### 2. Pick a style

```bash
python scripts/mplot.py --list
```

### 3. Render

```bash
# one-liner
python scripts/mplot.py raw/sample.txt -t xrd -o figures/xrd -f png,pdf

# or a reproducible recipe
python scripts/mplot.py --template xrd -o recipe.json
python scripts/mplot.py --spec recipe.json
```

### 4. Use it as a Python library

```python
import sys; sys.path.insert(0, "scripts")
from mplotlib import spec

result = spec.run({
    "technique": "tga",
    "style": {"preset": "journal", "journal": "elsevier", "columns": 1},
    "options": {"dtg": True, "to_percent": "auto"},
    "series": [{"path": "raw/TGA_S1.txt", "label": "S1 (air)"}],
    "export": {"out": "figures/tga", "formats": ["png", "pdf"]},
})
print(result["report"]["series"][0]["onset_T"])
```

---

## Plot types

| `--type` | Data | Highlights |
| --- | --- | --- |
| `xrd` | 2θ vs intensity | Overlay/stack, (hkl) labels, reference stick patterns, Scherrer crystallite size |
| `spectra` | Wavenumber / wavelength | FTIR, UV-Vis, Raman, NIR; ALS baseline, stacked offsets, band labels |
| `xps` | Binding energy vs counts | Survey and high-resolution, component/fit/background roles, inverted axis |
| `dsc` | Temperature vs heat flow | Tg/Tm/Tc marking, extrapolated onset, optional ΔH |
| `tga` | Temperature vs mass | DTG on a twin axis, onset temperature, residual mass, auto % conversion |
| `tensile` | Strain vs stress | E with explicit fit range, 0.2 % offset yield, UTS, elongation, toughness, bar mode |
| `eis` | Z′ / −Z″ / phase | Nyquist with equal aspect, Bode dual panel, Rs and Rct estimates |
| `cv` | Potential vs current | Peak detection, multi scan rate, i_p vs v^1/2 with R² |
| `cycling` | Cycle vs capacity | Charge/discharge curves or capacity vs cycle, Coulombic efficiency, retention |
| `xy` | anything | Line/scatter/bar, stacked offsets, dual axis, peak marking |

Full option tables: [`references/techniques.md`](references/techniques.md).

---

## The report

Every run writes a companion report next to the figure.

```
[figure] /abs/path/figures/fig2a.pdf
[figure] /abs/path/figures/fig2a.png
[report] /abs/path/figures/fig2a.report.md
[report] /abs/path/figures/fig2a.report.json

LFP:
  residual percent                       18.42
  total loss percent                     81.58
  onset T                                207.5
  T max rate                             224.1

[note] T_onset uses the extrapolated-baseline tangent method; it shifts with
       heating rate. Always state the heating rate in the caption.
```

The Markdown version has a table per series and a peak table where relevant, so
it can be pasted straight into a lab notebook or a supplementary file.

---

## Use as an Agent Skill

`SKILL.md` follows the Agent Skills format, so this repository doubles as a skill
for Claude Code, WorkBuddy and any other agent that reads `SKILL.md`.

```bash
# user-level skill (available in every project)
git clone https://github.com/<you>/materials-plot.git ~/.workbuddy/skills/materials-plot

# or project-level, committed with your repo
git clone https://github.com/<you>/materials-plot.git .workbuddy/skills/materials-plot
```

The agent then picks it up automatically when you ask for a plot, and works from
`SKILL.md` plus the four reference files.

---

## Origin bridge

```bash
# works anywhere: Origin-importable CSVs + a LabTalk scaffold + HOWTO.txt
python scripts/origin_bridge.py --spec recipe.json --out-dir origin_project

# is originpro usable on this machine?
python scripts/origin_bridge.py --check

# drive a local Origin and save a real project
python scripts/origin_bridge.py --spec recipe.json --opju out.opju
```

Automation needs `pip install originpro` and a licensed Origin ≥ 2021 on Windows.
The export path needs nothing. See
[`references/origin-automation.md`](references/origin-automation.md).

---

## Repository layout

```
materials-plot/
├── SKILL.md                  agent instructions and workflow
├── scripts/
│   ├── mplot.py              CLI entry point
│   ├── origin_bridge.py      Origin export + automation
│   ├── make_examples.py      synthetic data + gallery regeneration
│   └── mplotlib/
│       ├── loader.py         tolerant data parsing
│       ├── style.py          presets, journals, palettes
│       ├── analyze.py        peaks, baselines, modulus, onsets
│       ├── primitives.py     axis/legend/export plumbing
│       ├── techniques.py     the fifteen renderers
│       ├── spec.py           spec validation + pipeline
│       └── cli.py            argument handling
├── references/               techniques, spec schema, styles, formats, Origin
├── assets/example_data/      generated synthetic datasets
├── examples/                 generated figures
└── tests/                    smoke tests
```

---

## Development

```bash
pip install -r requirements.txt
python scripts/make_examples.py          # regenerate data + all figures
python tests/test_smoke.py               # parser and renderer smoke tests
```

`make_examples.py` renders every technique and fails loudly on the first error —
it is the fastest way to check a change.

---

## Caveats worth repeating

This tool produces **apparent** values, not refined ones.

* Scherrer sizes are not instrument-broadening corrected.
* EIS Rs/Rct are read off the plot geometry, not circuit-fitted.
* XPS components must come from a fit you performed elsewhere.
* Modulus fit ranges are auto-detected unless you supply `fit_range` — supply it
  for anything you intend to publish.
* Auto-detected mass-percent conversion assumes the first point is 100 %.

Every one of these is stated in the run's report. Read the notes.

---

## Contributing

Bug reports and new technique templates are welcome. See
[CONTRIBUTING.md](CONTRIBUTING.md). Adding a technique is roughly 80 lines in
`scripts/mplotlib/techniques.py` plus a reference section.

## License

MIT — see [LICENSE](LICENSE).

## 中文说明

材料专业学生的日常：测试数据散落在记事本、Excel、仪器导出的 txt 里，出图靠 Origin 手点。

这个工具把这件事变成一条命令 + 一份 JSON 配方：

* **读数据不用整理** — 仪器导出的三行表头、Tab/逗号/空格混用、单位写在括号里、Excel 多
  sheet，先跑 `--inspect` 看清楚，再出图。
* **默认就是 Origin 的样子** — 白底、内向刻度、四边镜像、Times New Roman、图例无边框。
  另有 `journal`（投稿用，更细的线、600 dpi）、`thesis`（学位论文，大字）、`ppt`、`poster`。
* **尺寸直接按期刊栏宽** — `--journal elsevier --columns 1` 出 90 mm 宽的图，插进模板
  不用缩放，字号就是规定的 8 pt。
* **出图同时给数** — 峰位、半高宽、d 值、Scherrer 晶粒尺寸、DSC 起始/峰值温度、
  TGA 失重与残余、弹性模量（含拟合区间）、屈服强度、EIS 的 Rs/Rct、首圈库仑效率与容量
  保持率，全部写进 `.report.md`，可以直接抄进论文或实验记录。
* **该提醒的都提醒** — 基线校正后强度是相对值、模量拟合区间是自动选的、EIS 的阻值是从
  图形上读的不是拟合的——这些都会以 note 的形式进报告，别忘了写进图注。
* **不想装 Origin 也能用** — 需要 `.opju` 时，`origin_bridge.py` 会导出 Origin 能直接
  导入的 CSV 和一份 LabTalk 脚本；本机装了 Origin 还能自动跑。

一份可复现的配方长这样（`recipe.json`）：

```json
{
  "technique": "xrd",
  "style": {"preset": "journal", "journal": "elsevier", "columns": 1, "palette": "journal"},
  "figure": {"xlabel": "2θ (degree)", "ylabel": "Intensity (a.u.)", "xlim": [10, 70]},
  "options": {"mode": "overlay", "normalize": "max", "baseline": "linear",
              "smooth": 9, "label_peaks": true},
  "series": [{"path": "data/LFP.txt", "label": "LFP"},
             {"path": "data/LFP_C.txt", "label": "LFP/C"}],
  "export": {"out": "figures/fig2a", "formats": ["png", "pdf", "tiff"]}
}
```

```bash
python scripts/mplot.py --spec recipe.json
```

数据换了、样本多了，改一行 `series` 重跑即可。配方文件和数据放一起，明年你或者你师弟
都能复现同一张图。

---
name: materials-plot
description: >
  Turn raw materials-characterisation data into publication-quality figures.
  This skill should be used whenever test data lives in txt / dat / csv / xlsx /
  Origin exports and the user needs a proper plot: powder XRD patterns (with
  (hkl) labels, reference stick patterns, Scherrer size), FTIR / UV-Vis / Raman /
  NIR spectra, XPS survey and high-resolution spectra, DSC and TGA/DTG thermal
  analysis, tensile stress-strain curves with modulus and yield-strength
  extraction, EIS Nyquist and Bode plots, cyclic voltammetry, and battery
  charge-discharge cycling. It renders Journal- and thesis-ready figures in an
  Origin-like visual style, exports PNG / TIFF / PDF / EPS / SVG at 600 dpi,
  and writes a companion report of every extracted number. Use it also when the
  user asks to "make an Origin plot", "画个图", "做个 XRD 图", "处理一下测试数据",
  "帮我出图/作图", "合并多条曲线", "标峰位/算结晶度/算弹性模量", or needs a .opju
  project file. Do NOT use it for schematic diagrams, flowcharts, or non-data
  illustrations.
agent_created: true
license: MIT
---

# materials-plot

Publication-quality figures from materials characterisation data, with the
numbers extracted alongside the picture.

Output is **matplotlib-based by default** so it runs anywhere with no Origin
licence. An optional Origin bridge produces Origin-import-ready files and a
LabTalk scaffold, and can drive a local Origin through `originpro` when one is
installed.

---

## 1. When to use this skill

**Use it for**: XRD · FTIR / UV-Vis / Raman / NIR · XPS · DSC · TGA & DTG ·
tensile / compression stress-strain · EIS (Nyquist, Bode) · CV · galvanostatic
cycling. Any plot where the input is a data file of numbers.

**Do not use it for**: crystal-structure schematics, mechanism diagrams, flow
charts, or any drawing that is not derived from measured data.

**Trigger phrases** (any of these, in English or Chinese): "画个 XRD 图",
"帮我把这些测试数据作图", "Origin 风格的图", "处理一下这个 excel 数据",
"标一下峰位", "算一下弹性模量", "画 Nyquist 图", "出期刊能用的图",
"合并多条曲线", "make a plot from this txt".

---

## 2. Environment

The scripts need `numpy`, `pandas`, `matplotlib`, and `scipy` (optional but
strongly recommended — it enables Savitzky-Golay smoothing, ALS baselines and
reliable peak finding). `openpyxl` is needed only for `.xlsx` input.

Install into the isolated environment, never globally:

```bash
PY="<python>"                       # the managed interpreter for this session
"$PY" -m pip install numpy pandas matplotlib scipy openpyxl pillow
```

If a dependency is missing at runtime the code degrades gracefully instead of
crashing, but always install the full set before producing a final figure.

Set `MPLBACKEND=Agg` when running headless.

---

## 3. The workflow — follow it in order

### Step 1 · Inspect the data before plotting anything

Never guess the column layout. Run:

```bash
python scripts/mplot.py --inspect "data/sample.txt"
```

This prints the detected delimiter, the header line, per-column min/max, and
whether each column is monotonic. For spreadsheets it also lists sheet names.
Pass `--json` for a machine-readable form. `--sheet`, `--delimiter`,
`--skiprows`, `--header-rows` override the automatic detection when it is wrong.

Read the output carefully and confirm: which column is x, which is y, what the
units are, how many rows were dropped as non-numeric.

### Step 2 · Choose the technique and read its reference section

| Data | `--type` | Extra reading |
| --- | --- | --- |
| 2θ vs intensity | `xrd` | `references/techniques.md#xrd` |
| Wavenumber / wavelength vs absorbance | `spectra` (`--set options.kind=ftir\|uvvis\|raman\|nir`) | `#spectra` |
| Binding energy vs counts | `xps` | `#xps` |
| Temperature vs heat flow | `dsc` | `#dsc` |
| Temperature vs mass | `tga` | `#tga` |
| Strain vs stress | `tensile` | `#tensile` |
| Z′ vs −Z″ or f | `eis` | `#eis` |
| Potential vs current | `cv` | `#cv` |
| Cycle vs capacity | `cycling` | `#cycling` |
| Anything else | `xy` | `#xy` |

`references/techniques.md` documents every option, the default axis labels and
the conventions (axis inversion, sign conventions, which quantities get
annotated) for each technique. Consult it before inventing options.

### Step 3 · Choose the output style

```bash
python scripts/mplot.py --list           # presets, journals, palettes, techniques
```

* **`--preset origin`** (default) — white canvas, inward mirrored ticks, framed
  axes, Times New Roman. Looks like a hand-made Origin graph.
* **`--preset journal`** — thinner strokes, no chart title, 600 dpi. Submission-ready.
* **`--preset thesis`** — larger type for a Chinese degree thesis.
* **`--preset ppt` / `poster`** — scaled up for projection and print.

Add `--journal elsevier --columns 1` (or `acs`, `rsc`, `wiley`, `nature`,
`springer`, `ieee`, `cell`) to get the exact column width in inches, so the
figure drops into the manuscript at 100 % scale with no rescaling.

Pick a palette deliberately: `origin`, `journal` (greyscale-safe), `okabe-ito`
(colour-blind safe), `grayscale`. Default to `journal` or `okabe-ito` for
anything going to a journal.

### Step 4 · Write a spec file, then run it

For anything beyond a trivial plot, write a JSON spec rather than a long flag
line. The spec is reproducible, greppable, and can be re-run after the data
changes.

```bash
python scripts/mplot.py --template xrd -o recipe.json   # starter spec
python scripts/mplot.py --spec recipe.json
```

Minimal spec:

```json
{
  "technique": "xrd",
  "style": {"preset": "journal", "journal": "elsevier", "columns": 1, "palette": "journal"},
  "figure": {"xlabel": "2θ (degree)", "ylabel": "Intensity (a.u.)", "xlim": [10, 70]},
  "options": {"mode": "overlay", "normalize": "max", "baseline": "linear", "label_peaks": true},
  "series": [
    {"path": "data/LFP.txt", "label": "LFP", "color": "#0072B2"},
    {"path": "data/LFP_C.txt", "label": "LFP/C", "color": "#C1272D"}
  ],
  "export": {"out": "figures/xrd_overlay", "formats": ["png", "pdf", "tiff"]}
}
```

Every field, including every technique-specific option, is documented in
`references/spec-reference.md`. Read it before guessing a key name.

Flag mode is fine for one-off plots:

```bash
python scripts/mplot.py a.txt b.txt -t xrd -l S1 S2 -o fig/xrd -f png,pdf \
    --preset journal --journal elsevier --normalize max --smooth 9 --label-peaks
```

Use `--set dotted.path=value` to reach any spec field from the CLI:

```bash
python scripts/mplot.py dsc.txt -t dsc --set options.exo=up --set style.preset=journal
python scripts/mplot.py eis.txt -t eis --set options.mode=bode
```

### Step 5 · Verify the figure before declaring success

This step is not optional.

1. **Read the exported PNG back** with the image-reading tool and look at it.
   Confirm: axis labels present and correctly spelled, units present, no legend
   covering data, no clipping at the edges, every curve visible and
   distinguishable, annotations not overlapping.
2. **Read the report** (`<out>.report.md` or `.report.json`) and sanity-check the
   extracted numbers against the physics: a modulus in the right order of
   magnitude, peak positions where you expect them, mass loss summing sensibly.
3. **Iterate.** Most first attempts need one adjustment: a wider `xlim`, a
   larger `smooth` window, `options.legend_loc`, or `label_top` reduced.

If the figure is wrong, fix the spec and re-run. Never present an unchecked
figure.

---

## 4. Judgement calls that matter

**Never fabricate or "improve" data.** Do not interpolate across gaps, do not
smooth so hard that peaks merge or shift, and do not let a baseline correction
clip a real shoulder. Every transform applied is recorded in the spec — keep it
that way so the figure can be reproduced and defended.

**State the fitting conditions.** Modulus from `tensile` uses a linear fit whose
range is either auto-detected or explicitly given via `fit_range`. The report
marks `E_auto_range: true` when it was automatic. A publication figure must use
an explicit range — set `"fit_range": [0.001, 0.004]` in the series spec and
report that range in the caption.

**Mind the sign and direction conventions.** These bite silently:

* FTIR wavenumber axes run high → low. Handled by default; override with
  `figure.xlim` if you want otherwise.
* XPS binding-energy axes run high → low. Inverted by default.
* EIS `−Z″` is derived from the sign of the imaginary part. The report notes when
  this was automatic — verify against the instrument export.
* DSC endo/exo is instrument-dependent. `options.exo` is `asis` by default, so
  the sign is left as measured and the report says so. Set it explicitly once you
  know your instrument, and say which convention you used in the caption.
* TGA onset temperature shifts with heating rate — always put the heating rate in
  the caption.

**Distinguish measurement from model.** EIS Rs/Rct read off the plot geometry are
not equivalent-circuit fit results. XPS component curves must come from a real
fit performed in CasaXPS/Origin — pass them in as extra series with
`"role": "component"`. Peak positions from `find_peaks` are apparent, not
refined by profile fitting. Say so when handing over.

**Label everything the reader needs.** Axis label, unit, and (for stacked or
offset plots) an explicit statement in the caption that curves were offset.
`options.mode=stack` changes the y-axis meaning — never let that pass silently.

---

## 5. Origin interoperability

Default output is an image. When the user explicitly needs an Origin project:

```bash
# Always works, no Origin required: writes Origin-importable CSVs + a LabTalk scaffold
python scripts/origin_bridge.py --spec recipe.json --out-dir origin_project
python scripts/origin_bridge.py --check          # is originpro usable here?
python scripts/origin_bridge.py --spec recipe.json --opju out.opju   # drives Origin
```

The `--out-dir` files are a real deliverable: tab-delimited CSVs laid out the way
Origin's ASCII importer expects (Long Name row plus Units row), a `build_graph.ogs`
LabTalk script, and a `HOWTO.txt` with three graded paths (run the scaffold /
import by hand / full automation). Prefer this when the user says "I need to
tweak it in Origin afterwards".

Full automation needs `pip install originpro` plus a licensed Origin ≥ 2021. See
`references/origin-automation.md` for the caveats.

---

## 6. Troubleshooting

| Symptom | Cause | Fix |
| --- | --- | --- |
| Chinese labels render as boxes | No CJK font on the machine | Install SimSun / Microsoft YaHei / Noto Sans CJK SC. A `RuntimeWarning` names the problem. |
| Figure text is not Times New Roman | Font missing | The code falls back through Nimbus Roman → Liberation Serif → DejaVu Serif and warns. |
| "No numeric rows found" | Wrong delimiter, or a metadata block at the top | `--inspect` first, then `--delimiter` / `--skiprows` / `--header-rows`. |
| All curves on top of each other | Not offset | `--set options.mode=stack` or `--offset auto`. |
| Peak labels overlapping | Too many labels | `--set options.label_top=4`, or `options.label_prominent_only=true`. |
| Everything looks too small/large | Wrong preset for the medium | Re-pick from `--list`; use `--journal X --columns N` for manuscripts. |
| `scipy` import error | Missing optional dep | Install scipy; ALS baseline and Savitzky-Golay degrade to simpler fallbacks otherwise. |
| Report numbers look impossible | Wrong column bound to x or y | Re-check with `--inspect`, then set `xcol` / `ycol` explicitly. |

---

## 7. Reference files

Read these when you need detail rather than guessing:

* `references/techniques.md` — every option for every technique, plus the axis
  and annotation conventions for each measurement type.
* `references/spec-reference.md` — the complete spec schema with field-by-field
  meaning and defaults.
* `references/style-presets.md` — journal column widths, font-size and
  line-width requirements, palette selection, colour-blind and greyscale safety.
* `references/data-formats.md` — how the tolerant parser handles headers,
  delimiters, metadata blocks, Excel quirks, and European decimal commas.
* `references/origin-automation.md` — originpro setup, what is automated, and
  what still has to be checked by hand.
* `scripts/make_examples.py` — run it to regenerate every example figure and the
  synthetic datasets; the fastest way to see correct usage.

---

## 8. A worked example

Turning two messy instrument logs into one publication figure:

```bash
# 1. what is actually in these files?
python scripts/mplot.py --inspect raw/sample_A.txt raw/sample_B.txt

# 2. render, journal-sized, both formats, peaks labelled
python scripts/mplot.py raw/sample_A.txt raw/sample_B.txt -t xrd \
    -l "As-prepared" "Annealed" -o figures/fig2a -f png,pdf \
    --preset journal --journal elsevier --columns 1 --palette journal \
    --normalize max --smooth 9 --baseline linear \
    --set options.label_peaks=true --set options.label_top=4 \
    --set figure.xlim=[10,70]

# 3. read figures/fig2a.png back and check it
# 4. read figures/fig2a.report.md for the extracted peak table
```

Then quote the peak positions and FWHM from the report for the Scherrer
calculation, and state the Scherrer constant and wavelength in the caption.
