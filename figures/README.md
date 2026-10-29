# Paper Figures

PNG versions are embedded in `../paper-v2.md`; SVG and PDF versions are provided for layout. Asset basenames are stable and do not track manuscript numbering.

| Paper figure | Asset basename | Content |
|---|---|---|
| 1 | `figure-2-pull-path` | Pull architecture and concurrency scopes (schematic) |
| 2 | `figure-7-write-volume` | Received bytes, retained data, and observed writes |
| 3 | `figure-1-aws-pull` | AWS network and extraction timeline |
| 4 | `figure-5-fsverity-commit` | Commit mechanism schematic and measured completion |
| 4a | `figure-12-fsverity-patch` | Baseline vs patch, one Set 1 observation each |
| 4b | `figure-13-fsverity-overlap` | Recorded fs-verity intervals on normalized trace axes |
| 9a | `figure-14-erofs-unpack` | PyTorch largest-layer comparison and whole-image EROFS unpack |
| 5 | `figure-11-ingestion` | Local download-only throughput |
| 5a | `figure-15-gzip-vs-zstd` | Sequential gzip/zstd full pulls, reusing the two local sequential configurations from Figure 10 |
| 6 | `figure-4-sequential-vs-parallel-unpack` | Matched sequential/parallel measured timelines |
| 7 | `figure-8-codec-cost` | Codec elapsed time and selected CPU costs |
| 8 | `figure-6-layer-scaling` | Synthetic layer-layout parallelism |
| 9 | `figure-9-layer-sizes` | PyTorch data concentration |
| 10 | `figure-3-full-pull` | End-to-end configuration improvements |
| 11 | `figure-10-delivery-paths` | Native path and optional dependencies (schematic) |

## One renderer, in paper order

All figure source now lives in `../render-figures.py`. Each figure has its own clearly headed `render_*` function, ordered as it appears in the paper. Edit titles, labels, annotations, and data in that function. Shared colors and drawing helpers are at the top; command-line selection is at the bottom. Existing image filenames are unchanged.

From the repository root, render all current figures:

```sh
uv run --with matplotlib python -I pull-optimization/render-figures.py
```

Or list/select figures:

```sh
uv run --with matplotlib python -I pull-optimization/render-figures.py --list
uv run --with matplotlib python -I pull-optimization/render-figures.py --figure 1 4 4a
```

Figure selectors match the paper numbering, including `4a` and `9a`. The standalone parallel-unpack schematic is `parallel-example`. Removed figures remain available as the optional `fsverity-overlap` and `delivery-paths` selectors; they are not part of the default build.

Figures 3, 6, and 9a read original benchmark logs from `~/git/tmp` by default. Override their location with `--raw-root PATH`. Use `--output-dir PATH` to render previews without overwriting the paper assets.

Result charts export JSON; measured timelines export CSV and metadata. Inputs are the corrected results in `RESULTS.md` and the local logs named in the timeline metadata. No new benchmarks were run. fs-verity's schematic has no measured time axis because the complete original trace is unavailable; its completion bars are measured. Network timeline curves use 1 s averages. Timing alignment and tiny-layer visibility details are recorded in the relevant metadata.

## AWS Full Pull

- `figure-1-aws-pull.svg`: vector version for layout.
- `figure-1-aws-pull.pdf`: vector version for paper layout.
- `figure-1-aws-pull.png`: raster preview.
- `figure-1-network.csv`: original network-rate samples, approximately 100 ms apart.
- `figure-1-extraction.csv`: extraction intervals derived from containerd's `diff applied` events.
- `figure-1-metadata.json`: event landmarks and alignment method.

The figure uses the AWS PyTorch full pull from Run 7b. The network plot displays time-weighted 1 s averages. The original approximately 100 ms samples, including the 3.6521 Gbps peak at 5.48 s, remain available in the CSV. The bulk-transfer endpoint is the last sample exceeding 0.01 Gbps, excluding low-rate background NIC traffic. Total command duration is 266.6529 s; bulk transfer ends at 90.93 s, leaving 175.7229 s until completion.

Extraction intervals use each `diff applied` event's timestamp minus its logged `d` duration. These intervals measure actual layer application rather than the broader `layer unpacked` timer, which can include waiting. Tiny layers may appear as narrow marks at this scale. Gaps are left unclassified.

The original sampler saved command-relative network times but did not retain the command's absolute start timestamp. Runtime events are therefore aligned by treating the first log event as elapsed zero. Extraction endpoints are approximate on the network time axis, and the figure labels them accordingly. The last extraction is approximately 266.23 s; the whole-image `image unpacked` event follows at approximately 266.55 s. The first extraction begins at approximately 0.72 s, while downloading is still active.

Regenerate from the original local evidence:

```sh
uv run --with matplotlib python -I pull-optimization/render-figures.py --figure 3
```

Network and extraction colors use the dataviz reference palette's blue and orange, validated for light-background contrast and color-vision separation. Phase identity also has separate panels, direct labels, and a legend. The original debug log is not copied into this directory because it includes request metadata; the plot's numerical data is exported separately.
