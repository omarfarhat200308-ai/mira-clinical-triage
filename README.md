# TABIB — hackathon prototype (predecessor of MIRA)

> **This repository is the earlier hackathon prototype, not the MIRA v1.3 research release.**
> The corrected MIRA research artifact is on Zenodo: **[10.5281/zenodo.23136983](https://doi.org/10.5281/zenodo.23136983)** (v1.3-corrected, 2026-10-04). It is **not** the code in this repository.

TABIB was built at the Band of Agents Hackathon (lablab.ai, June 2026) as a three-agent WhatsApp clinical-triage prototype: an Intake agent, a Diagnostic agent and a Triage agent, coordinated through the Band agent platform, each calling an LLM (Claude in this code). It was later renamed and rebuilt as **MIRA**, a fixed multi-stage pipeline with a deterministic red-flag layer at its Triage stage.

## What this repository is
- The hackathon-stage source: `src/` (agents, Band orchestrator, WhatsApp bridge), `shared_prompts/`, `TIER3_PLAN.md`.
- A historical record of where the project started. It is kept public so the origin of MIRA is traceable.

## What this repository is not
- It is **not** the evaluated MIRA code and does not reproduce the v1.3 results.
- It has **not** been clinically validated, was not evaluated on real patients, and **must not be used for clinical decisions**.
- Statements in the original hackathon README such as "zero training required" and "multi-agent decision support" described the hackathon pitch; they are **not** research findings and are not claimed by the MIRA report.

## Where to find MIRA
| | |
|---|---|
| Research release (paper + code and evaluation archive) | https://doi.org/10.5281/zenodo.23136983 |
| All versions (concept DOI) | https://doi.org/10.5281/zenodo.23067418 |
| MIRA code on GitHub | https://github.com/omarfarhat200308-ai/mira-research |

## Relationship between the versions
TABIB (this repo, June 2026) → MIRA research prototype (July–September 2026) → MIRA v1.3-corrected interim report (Zenodo, 2026-10-04, supersedes v1.0–v1.2).

## Authors
Mohammed Omar Farhat (hackathon prototype and MIRA). The v1.3 report is co-authored with Syeda Fahada Zia and L. Sunil Kumar (clinical review).

## License
*(No license file is present in this repository. Add one only if you want the hackathon code licensed; MIT is used for the MIRA code archive.)*
