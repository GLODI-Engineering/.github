# General Simulator — power-electronics circuit simulation, from the schematic to the waveform

An open-source simulator for **power electronics and mixed circuit / block-diagram systems**,
written in Rust, with **Omni Schematics** — a schematic editor that runs **inside VS Code**.

Its distinguishing choice is in the device model: switches, diodes and other piecewise-linear
devices are resolved by solving a **linear complementarity problem** at each step rather than by
Newton–Raphson iteration. A converter's switching instants are therefore found, not iterated
towards — and a run that violates a device rating says so, with the time it happened and the
component that did it, instead of returning a plausible-looking curve.

```bash
# the engine
cargo install general-simulator-cli

# the editor: download omni-schematics.vsix from the Omni Schematics releases, then
code --install-extension omni-schematics.vsix
```

Prebuilt engine binaries for Linux (x86-64 and ARM), macOS (Apple silicon) and Windows are
attached to every [engine release](https://github.com/GLODI-Engineering/general-simulator/releases),
so neither piece needs a toolchain.

**[Documentation](https://glodi-engineering.github.io/general-simulator/)** ·
**[Walk-through: simulating an LLC resonant converter from zero](https://github.com/GLODI-Engineering/omni-schematics/blob/main/docs/articles/simulating-an-llc-converter.md)**

## How the pieces fit

You draw. The compiler translates. The engine solves. The results come back to the editor to be
plotted — and the editor never decides what a component *is*: the engine declares that, and the
editor displays it faithfully.

```
schematic (.gsim_sch)  →  gsim-core  →  netlist  →  general-simulator  →  waveforms + events
      the editor           compiler                      the engine          the result file
```

## The repositories

| Repository | What it is |
|---|---|
| **[general-simulator](https://github.com/GLODI-Engineering/general-simulator)** | The engine. Piecewise-linear circuits and mixed circuit/block-diagram systems, LCP device resolution, native control blocks, and the rule engine that reports a violated rating as an event. Rust, AGPL-3.0. [crates.io](https://crates.io/crates/general-simulator-cli) |
| **[general-mna](https://github.com/GLODI-Engineering/general-mna)** | Modified nodal analysis: descriptor-DAE circuit assembly, hierarchy flattening, and the netlist model the engine stamps. Rust, AGPL-3.0. [crates.io](https://crates.io/crates/general-mna) |
| **[gsim-core](https://github.com/GLODI-Engineering/gsim-core)** | The document model and netlist compiler, plus the GSF v2 parametric component-contract format, with Python and WebAssembly bindings. Rust, MIT. [crates.io](https://crates.io/crates/gsim-core) |
| **[Omni Schematics](https://github.com/GLODI-Engineering/omni-schematics)** | The VS Code extension: schematic capture, component authoring, running a simulation and viewing the waveforms. Named apart from the engine because it is built to drive others. TypeScript, AGPL-3.0. |
| **[gsim-tools](https://github.com/GLODI-Engineering/gsim-tools)** | Build schematics and read results from Python, over the `gsim_py` bindings. Python, MIT. |
| **[gs-waveform-measurements](https://github.com/GLODI-Engineering/gs-waveform-measurements)** | Measurements on non-uniform time series: max/min/RMS, integrals, threshold crossings, TRIG–TARG, Fourier analysis, error norms. Rust, AGPL-3.0. |
| **[spice-lsp](https://github.com/GLODI-Engineering/spice-lsp)** | SPICE netlist tooling: a dialect-agnostic lexer, parser and symbol table (ngspice, Xyce). Rust, AGPL-3.0. |
| **[gs-ui-probe](https://github.com/GLODI-Engineering/gs-ui-probe)** | Drive a web UI the way a person does — real pointer travel, real drags, real keystrokes. Playwright underneath, no knowledge of any particular application. TypeScript, AGPL-3.0. |

## About GLODI Engineering

GLODI Engineering builds simulation tools for power electronics: converters, their magnetics, and
the control that runs them. The tools are written for the engineer who has to defend a number,
not only plot one.

The work starts from a failure mode rather than a feature list — a simulation that looks right. A
transformer wired with the wrong sign absorbs power on both sides and still draws a clean primary
waveform. A magnetising current drifts from 40 A to 245 A and nothing says so until someone
measures it. A solver failure arrives as a bare string, with no time, no step, and no indication
of which switches moved.

Two decisions follow from that. Piecewise-linear devices are resolved as a **linear
complementarity problem** at every step instead of by Newton–Raphson with voltage limiting, so a
switching instant is found rather than iterated towards. And a run that breaks a rating, a
balance or a bound **reports it as an event** carrying the time, the value and the component
responsible, because a wrong simulation is expected to say so.

The editor is named apart from the engine because a drawing should outlive the choice of engine
underneath it: Omni Schematics runs `general-simulator` today and is built to run others.

Everything published here is open source, released on crates.io, and documented with examples
that are run before they are written down. Issues are read and answered.

## What it is for

Converter design work where the answer has to be trustworthy rather than merely plotted:
resonant and hard-switched topologies, ZVS boundaries, volt-second balance, conduction and
switching loss, efficiency, and the question of *which* component a bad number came from.
The documentation builds a buck converter, open-loop and under PID, and a boost/LLC resonant
stage from scratch; the LLC walk-through above then checks the simulator against the
closed-form gain law — including the places where the two disagree, and which of them is wrong
there.
