# Yoneda reconstruction

Published figures rebuilt from the raw files a laboratory actually wrote, with a [Yoneda](https://github.com/levylabpitt/yoneda) instance as the record of every step and an AI agent doing the reading.

A figure in a paper is the end of a chain that usually exists only on a laboratory drive and in someone's memory: which acquisitions were used, how the instrument was wired that day, which channels hold current and voltage, what calibration turns a gate voltage into a chemical potential, and which corrections were applied by hand. A reconstruction here recovers that chain from the raw bytes and stores each link as a cell that names its evidence, so the figure can be redrawn by query and every step can be checked, disputed or retracted.

![Published and regenerated](reconstructions/fig2-briggeman-2025/instance/fig2_side_by_side.png)

The first reconstruction is Fig. 2 of Briggeman et al., "Engineered chirality of one-dimensional nanowires," *Science Advances* 11, eadx4761 (2025): conductance and transconductance of a chiral superlattice and a control waveguide written at the LaAlO3/SrTiO3 interface, measured at 25 mK in 2017. It is rebuilt from 237 raw gate sweeps (66 MB). The paper's data deposit is read only by the audit, to compare.

## How a reconstruction works

**Ingest.** Every file under the relevant folder of the laboratory store enters the instance as a cell named by the SHA-256 of its bytes. A `located` cell says where the bytes were found; nothing is copied out of the store. Each acquisition becomes a `run` cell with two times, when it was measured (read from the file's own clock) and when the record was written.

**Claims.** What the files mean enters as authored claims, each pointing at its evidence: the wiring (here read from two sources in every file and checked against the conductance the instrument computed at the time), which run is behind which panel, and the calibration. A claim that fails its check is retracted by an `outcome` cell. It stays in the instance; the `active` view stops returning it.

**Derivations.** Each step of the arithmetic is a `derivation` cell whose faces are its inputs, the code that ran (by hash) and its output (by hash), and whose data are its parameters. Steps the original processing did by hand, such as removing gate jumps, are done by one stated rule and the cell records what the rule found.

**Projection.** The figure is computed from the active claims and derivations. Nothing in the drawing code names a run. Retracting a claim changes the figure without changing code.

**Audit.** `verify.py` re-hashes every raw file, reruns every derivation from the inputs and parameters its cell names, checks that the output hashes match, and redraws the figure to the same bytes. Only then does it open the published deposit and compare.

## What the AI agent did

The Fig. 2 reconstruction was carried out by an AI agent (Claude) in one session from a short request, and the audited figure existed within 45 minutes of it. The agent found the raw data by the sample tag, read undocumented instrument headers, worked out which of two conflicting channel assignments was right, matched runs to panels, recovered the calibration from the published axes, and identified two processing steps the files do not record (a uniform lock-in lag shift of the published curves and the gate-jump alignment of the published maps).

None of that is taken on trust. Each judgment is a claim cell with named evidence and each computation is a derivation that can be rerun, so a person or another agent can check any step and retract it without losing the rest. This is the arrangement the Yoneda design is meant for: records whose meaning is not settled when they are saved, and interpretations that accrue, compete and get corrected afterward.

## Why a categorical database

A file, a run, a claim about the run, a check of the claim and the retraction of the claim are all the same kind of row, a cell whose faces point at other cells. A correction four levels above the data needs no new table. By the Yoneda lemma an object is determined by what points at it, and a run here means exactly the set of cells that refer to it, which grows as interpretation accumulates. Because names are hashes of content, verifying a rebuild is checking that a diagram of squares commutes, two independent builds produce the same names, and copies of an instance merge as a union with nothing to remap. Comparing a reconstruction with a publication becomes a question of naturality: a processing step applied the same way at every field is a natural transformation, and where the squares fail, an unrecorded step has been located. [`docs/`](docs/) has the full write-up with the three diagrams.

## Layout

    yoneda/                              the Yoneda genome, a submodule pinned to the commit the instances record
    reconstructions/fig2-briggeman-2025/ code, instance, derived blobs, marimo notebook, README
    docs/                                LaTeX write-up of the method and its categorical structure

## Quick start

    git clone --recurse-submodules https://github.com/levylabpitt/yoneda-reconstruction
    cd yoneda-reconstruction/reconstructions/fig2-briggeman-2025
    pip install numpy scipy matplotlib pillow marimo
    marimo edit report.py
    python figure.py --db instance/fig2.yoneda.db --roots fig2-yoneda:blobs=instance/derived --out fig2.png

The notebook and the last command need only what is in the repository, and the last reproduces the figure byte for byte (sha256 `748312408cc81399...`). `rebuild.py` rebuilds everything from the raw files when the laboratory store is mounted.

## Adding a reconstruction

One folder per figure under `reconstructions/`. Ingest the raw files by hash from wherever they live, enter every interpretation as a claim with its evidence, record every computation as a derivation, draw the figure from active cells only, and ship an audit that recomputes and compares by hash. Public instances record what the files do and do not contain; questions for the people who took the data belong in the laboratory's own instance.

## Credits and license

Code is BSD 3-Clause (see `LICENSE`). The measurements are by the authors of the reconstructed paper. The published figure in `reconstructions/fig2-briggeman-2025/reference/` is from Briggeman et al., *Sci. Adv.* 11, eadx4761 (2025), distributed under CC BY 4.0.
