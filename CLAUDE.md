# CLAUDE.md — Root Session Start File
# Workspace: c:\evm

**Last Updated:** 2026-04-23  
**Purpose:** Read this file at the start of every session to orient yourself.

---

## 📁 Workspace Overview

This workspace (`c:\evm`) contains **three projects**:

| Project | Path | Priority | Purpose |
|---------|------|----------|---------|
| **evm-sv** | `evm-sv/` | 🔴 HIGH | EVM framework — the main project we build and extend |
| **uvm** | `uvm/uvm-core/` | 🟡 MEDIUM | Reference copy of UVM (Accellera 1800.2-2020) |
| **riscv-evm-course** | `riscv-evm-course/` | 🟢 LOW | RISC-V course materials — not currently active |

---

## 📋 Detailed Reference Files

| File | What it covers |
|------|----------------|
| `evm-sv/CLAUDE.md` | **Deep EVM reference** — every class, method, task, coding pattern |
| `CLAUDE_UVM.md` | **Deep UVM reference** — UVM architecture, all major classes, macros |

**Session start pattern:** Read this file first, then read `evm-sv/CLAUDE.md` if working on EVM, or `CLAUDE_UVM.md` if comparing to UVM.

---

## 🔍 Project Summaries

### evm-sv — Embedded Verification Methodology
- **Goal:** 100% of UVM's critical features at 10% of UVM's complexity
- **Target:** Embedded systems (FPGA, microcontroller DUTs), not ASIC-scale
- **Status:** Production-ready. All core features complete.
- **Key innovation:** No config_db (direct VIF), Quiescence Counter (evm_qc), 3-phase reset, built-in test/sequence registries
- **Language:** SystemVerilog (.sv), MIT License
- **Entry point:** `evm-sv/vkit/src/evm_pkg.sv` (core), `evm-sv/vkit/evm_vkit/evm_vkit_pkg.sv` (protocol agents)
- **See full details:** `evm-sv/CLAUDE.md`

### uvm — Universal Verification Methodology (reference copy)
- **Goal:** Industry standard for ASIC/FPGA verification (IEEE 1800.2-2020)
- **Role here:** Reference implementation to understand what EVM replaces/simplifies
- **Complexity:** Full-scale — factory, config_db, TLM1+TLM2, RAL, callbacks, DPI, phasing domains
- **Location:** `uvm/uvm-core/src/`
- **See full details:** `CLAUDE_UVM.md`

### riscv-evm-course — RISC-V Course (deprioritized)
- **Goal:** Educational course materials combining RISC-V + EVM verification
- **Status:** Active but lower priority than evm-sv development
- **Structure:** `labs/`, `lectures/`, `reference/`, `docs/`, `common/`
- **Next steps:** See `riscv-evm-course/next.txt`

---

## ⚖️ EVM vs UVM — Key Differences at a Glance

| Feature | UVM | EVM |
|---------|-----|-----|
| Virtual Interface | `uvm_config_db::set/get()` | Direct: `agent.set_vif(vif)` |
| Object creation | `uvm_factory` + macros | Direct `new()` + `evm_test_registry` |
| Phase argument | All phases take `uvm_phase phase` | No arg — just override the task/function |
| Objections | `phase.raise_objection(this)` | `evm_root::get().raise_objection("name")` |
| Reset support | Single `reset_phase` | 3-phase: `pre_reset/reset/post_reset` + mid-sim events |
| Test completion | Manual objection management | Optional `evm_qc` (quiescence counter) |
| Sequence to driver | `start_item/finish_item` | `seq_item_port.get_next_item() / item_done()` |
| TLM2 | Full sockets (b_transport) | Not implemented (not needed for embedded) |
| DPI | Full C/C++ integration | None needed |
| Compile time | Minutes | Seconds |
| Learning curve | Weeks | Days |

---

## 🎯 Critical EVM Rules (Always Remember)

1. **ALWAYS call `super.X()` first** in every overridden phase (build, connect, run, main, etc.)
2. **No config_db** — pass VIF directly: `agent.driver.set_vif(vif)` in connect_phase
3. **Phase signatures have NO arguments** — e.g., `virtual task main_phase();` (not `main_phase(uvm_phase phase)`)
4. **Objection pattern:** `raise_objection("desc")` at start, `drop_objection("desc")` at end of main_phase
5. **run_phase runs parallel** to sequential phases — monitors/scoreboards go in run_phase
6. **EVM_REGISTER_TEST / EVM_REGISTER_SEQUENCE** macros → use `+EVM_TESTNAME=` plusarg

---

## 🗂️ File You Should Read for Each Task

| If working on... | Read |
|-----------------|------|
| EVM framework changes | `evm-sv/CLAUDE.md` |
| Adding new agent/protocol | `evm-sv/CLAUDE.md` → section "Protocol Agents" |
| Register model / RAL | `evm-sv/CLAUDE.md` → section "Register Model" |
| Comparing EVM vs UVM | `CLAUDE_UVM.md` |
| Course lab development | `riscv-evm-course/next.txt` + `riscv-evm-course/README.md` |
