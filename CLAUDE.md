# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository purpose

This repository is primarily a local protocol/documentation knowledge workspace, not an application with a root build system. It contains:

- `UVM/uvm-1.2/`: Accellera UVM 1.2 source kit, examples, HTML reference, license, and release notes.
- `amba_prot/`: AMBA/APB/AXI/ACE protocol PDFs used as reference material.
- `agents/.claude/settings.local.json`: local Claude Code permission settings for this workspace.

## Commands

There is no repository-level `make`, lint, or test command. UVM examples are run from individual example directories with simulator-specific makefiles.

### UVM environment

Use the bundled UVM tree unless intentionally testing another installation:

```bash
export UVM_HOME="$PWD/UVM/uvm-1.2"
```

UVM users compile `UVM/uvm-1.2/src/uvm.sv` first and add `+incdir+$UVM_HOME/src`; user code imports `uvm_pkg::*` and includes `uvm_macros.svh` when macros are needed.

### Run UVM examples

From an example directory, choose the makefile for the installed simulator:

```bash
cd UVM/uvm-1.2/examples/simple/hello_world
make -f Makefile.questa all   # Questa: builds DPI lib/work lib as needed, compiles, runs, checks log
make -f Makefile.vcs all      # VCS
make -f Makefile.ius all      # Cadence IUS/irun
```

Single-example targets are defined by each example makefile. Common targets include:

```bash
make -f Makefile.questa comp
make -f Makefile.questa run
make -f Makefile.questa clean
make -f Makefile.questa help
```

The shared example makefiles check simulator logs for expected `UVM_ERROR` and `UVM_FATAL` counts via `CHECK`. Override expected counts or settings on the make command line when an example intentionally emits errors:

```bash
make -f Makefile.questa all N_ERRS=1 N_FATALS=0 UVM_VERBOSITY=UVM_HIGH BITS=64
make -f Makefile.vcs all UVM_HOME=/path/to/other/uvm
```

For command previews without running a simulator:

```bash
make -f Makefile.questa -n all
```

### Reference extraction

The AMBA protocol files are PDFs. When answering source-backed questions from them, prefer extracting the relevant page range instead of reading the whole PDF at once, for example:

```bash
pdftotext -f <start-page> -l <end-page> -layout amba_prot/AXIACE5.pdf /tmp/axiace5-pages.txt
```

## Architecture overview

### UVM source layout

`UVM/uvm-1.2/src/uvm.sv` is the top-level compile unit and simply includes `uvm_pkg.sv`. `uvm_pkg.sv` defines `package uvm_pkg` and includes the major subsystem aggregators in dependency order:

1. `dpi/uvm_dpi.svh` for DPI declarations and simulator-facing C/C++ support under `src/dpi/`.
2. `base/uvm_base.svh` for core infrastructure: objects, factory/registry, resources/config DB, policies, events/callbacks, reporting, phasing, components, objections, command-line processing, and traversal.
3. `dap/uvm_dap.svh` for data-access policy helpers.
4. `tlm1/uvm_tlm.svh` for TLM1 interfaces, ports/exports/imps, analysis ports, FIFOs, request/response channels, and sequencer connections.
5. `comps/uvm_comps.svh` for standard components such as driver, monitor, agent, env, test, subscriber, scoreboard, and comparators.
6. `seq/uvm_seq.svh` for sequences, sequence items, sequencers, arbitration, and built-in sequence utilities.
7. `tlm2/uvm_tlm2.svh` for TLM2 interfaces, sockets, generic payload, time, and related ports/exports/imps.
8. `reg/uvm_reg_model.svh` for the register layer: register/memory objects, maps, fields, adapters, predictors, backdoor support, and built-in register/memory sequences.

The `.svh` aggregator files are the best starting points when tracing subsystem dependencies; individual classes live in same-named files below each subsystem directory.

### Examples layout

`UVM/uvm-1.2/examples/` has shared simulator makefiles at its root and per-example makefiles under `examples/simple/*` and `examples/integrated/*` that include those shared makefiles. Examples compile local SystemVerilog files plus `$(UVM_HOME)/src/uvm.sv`.

Representative example families:

- `examples/simple/hello_world`: minimal producer/consumer-style UVM example.
- `examples/simple/factory`, `callbacks`, `interfaces`, `objections`, `trivial`: focused demonstrations of individual UVM mechanisms.
- `examples/integrated/apb` and `examples/integrated/codec`: larger examples combining agents, register models, scoreboard/environment code, and DUT/top-level testbench files.

### Documentation/reference files

- `UVM/uvm-1.2/README.txt` documents installation, prerequisites, use of `UVM_HOME`, compiling `src/uvm.sv`, DPI requirements, and example execution.
- `UVM/uvm-1.2/UVM_Reference.html` and `UVM/uvm-1.2/docs/html/` provide local UVM reference documentation.
- `amba_prot/*.pdf` are protocol references; cite exact pages/sections when using them for answers or notes.
