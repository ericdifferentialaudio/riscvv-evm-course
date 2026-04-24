# EVM_ROADMAP.md — Industry Adoption Feature Roadmap
# Workspace: c:\evm

**Created:** 2026-04-23  
**Purpose:** Defines the prioritized feature roadmap for EVM to achieve industry adoption in embedded/FPGA verification. This is a living document — update status as features are implemented.

**Context:** EVM targets embedded FPGA verification, NOT ASIC-scale flows. Bugs caught in simulation are preferred, but lab catches are acceptable — every bug caught later is ~10x more expensive. The goal is 100% of UVM's *critical* features at 10% of UVM's complexity.

---

## ✅ Completed

| Feature | Date | Files Changed |
|---------|------|---------------|
| P0.3 — Expected error / negative test support | 2026-04-23 | `evm_report_handler.sv`, `evm_log.sv` |
| P0.1 — CRV infrastructure (rand items + seed) | 2026-04-24 | `evm_sequence_item.sv`, `evm_rand_sequence.sv` (new), `evm_cmdline.sv`, `evm_pkg.sv` |
| P0.2 — Functional coverage base class + reporting | 2026-04-24 | `evm_coverage.sv`, `evm_base_test.sv`, `python/evm_cov_merge.py` (new) |
| P1.1 — SPI agent (initiator + target + device models) | 2026-04-24 | `evm_spi_agent/` (9 new files), `evm_vkit_pkg.sv` |
| P1.2 — I2C agent (initiator + target + register models) | 2026-04-24 | `evm_i2c_agent/` (9 new files), `evm_vkit_pkg.sv` |

---

## 📊 Active Roadmap

| Priority | Feature | Effort | Impact |
|----------|---------|--------|--------|
| 🟡 P1 | UART agent | Low | High |
| 🟡 P1 | APB agent | Medium | High |
| 🟡 P1 | Backdoor / HDL force-release | Low-Medium | High |
| 🟡 P1 | Transaction waveform recording | Medium | High |
| 🟢 P2 | Sequence priority/arbitration | Medium | Medium |
| 🟢 P2 | CI/CD + JUnit XML output | Low | Medium |
| 🔵 P3 | ISS / firmware co-sim agent (stretch) | High | Transformational |

---

## 🔴 P0 — Blockers to Adoption (Implement First)

### P0.1 — Constrained Random Verification (CRV) Infrastructure

**Why it's a blocker:** Without CRV, EVM supports directed testing only. Every real verification project uses randomization to find bugs that directed tests miss. This is the #1 gap vs. UVM.

**What SystemVerilog already provides:** `rand` keyword, `constraint` blocks, `randomize()` — EVM just needs the *pattern* and supporting infrastructure.

**Files to modify:**
- `evm-sv/vkit/src/evm_sequence_item.sv` — add `randomize_item()` helper
- `evm-sv/vkit/src/evm_sequence.sv` — document CRV usage pattern

**New files:**
- `evm-sv/vkit/src/evm_rand_sequence.sv` — `evm_rand_sequence` base class

**API design:**

```systemverilog
// In evm_sequence_item — add to base class:
virtual function bit randomize_item();
    // Calls SV randomize(), logs seed, returns success/fail
    // Seed comes from evm_cmdline::get_seed() (+evm_seed plusarg)
    bit ok;
    ok = this.randomize();
    if (!ok) log_error($sformatf("[CRV] randomize() failed on %s", get_type_name()));
    else log_info($sformatf("[CRV] %s randomized (seed captured in evm_cmdline)", get_type_name()), EVM_DEBUG);
    return ok;
endfunction

// New class: evm_rand_sequence (extends evm_sequence)
// Automatically calls randomize_item() on each added item before execute()
// Pattern for user-defined randomizable items:
class my_txn extends evm_sequence_item;
    rand logic [31:0] addr;
    rand logic [31:0] data;
    rand logic [3:0]  strb;
    
    constraint addr_aligned { addr[1:0] == 2'b00; }
    constraint strb_valid   { strb inside {4'b1111, 4'b0011, 4'b1100, 4'b0001}; }
    
    virtual function bit randomize_item();
        return super.randomize_item();  // logs + calls randomize()
    endfunction
endclass

// Usage in sequences:
class my_rand_seq extends evm_rand_sequence;
    virtual task execute();
        my_txn txn;
        repeat (10) begin
            txn = new("txn");
            if (!txn.randomize_item()) return;
            // optionally: txn.randomize() with { txn.addr == 32'h100; }
            add_item(txn);
        end
    endtask
endclass
```

**Seed management:**
- `+evm_seed=<int>` already handled in `evm_cmdline.sv`
- `evm_cmdline::set_random_seed()` already calls `srandom()` on the process
- `evm_sequence_item::randomize_item()` should log the active seed at DEBUG verbosity so failed runs are 100% reproducible: `+evm_seed=<logged_value>`

**Success criteria:**
- [ ] `evm_rand_sequence` compiles and runs in Vivado/Questa
- [ ] Failed randomize() prints error; test is marked FAILED
- [ ] Seed is logged and replayable via `+evm_seed=`
- [ ] Example in `examples/example1` updated to show a randomized test

---

### P0.2 — Functional Coverage Infrastructure

**Why it's a blocker:** Without coverage, there is no answer to "when are we done testing?" — this question kills adoption conversations.

**Current state:** `evm-sv/vkit/src/evm_coverage.sv` exists as a stub — needs real implementation.

**Files to modify:**
- `evm-sv/vkit/src/evm_coverage.sv` — implement base class
- `evm-sv/vkit/src/evm_base_test.sv` — aggregate coverage in report_phase

**API design:**

```systemverilog
// evm_coverage_model — virtual base class (extends evm_component)
// User extends this and declares covergroups inside
virtual class evm_coverage_model extends evm_component;
    string      model_name;
    real        coverage_target = 0.0;  // 0 = no threshold check; set via set_target()
    bit         enabled = 1;
    
    function new(string name="evm_coverage_model", evm_component parent=null);
    
    // ── User overrides these ──────────────────────────────────────────────
    pure virtual function void sample();         // call this from your monitor
    pure virtual function real get_coverage();   // return 0.0–100.0
    // ─────────────────────────────────────────────────────────────────────
    
    function void set_target(real pct);          // minimum coverage % for PASS
    function real get_target();
    function void enable();
    function void set_disabled();
    function bit  is_enabled();
    
    // report_phase: prints coverage%, PASS/FAIL vs target
    virtual function void report_phase();
endclass

// Usage pattern (in monitor or env):
class my_coverage extends evm_coverage_model;
    covergroup addr_cg;
        addr_cp: coverpoint txn.addr {
            bins low    = {[32'h0000:32'h00FF]};
            bins mid    = {[32'h0100:32'hFEFF]};
            bins high   = {[32'hFF00:32'hFFFF]};
        }
        rw_cp: coverpoint txn.is_write { bins wr={1}; bins rd={0}; }
        cross addr_cp, rw_cp;
    endgroup
    
    my_txn txn;  // set before calling sample()
    
    function new(string name, evm_component parent);
        super.new(name, parent);
        addr_cg = new();
        set_target(90.0);  // require 90% coverage for PASS
    endfunction
    
    virtual function void sample();
        addr_cg.sample();
    endfunction
    
    virtual function real get_coverage();
        return addr_cg.get_inst_coverage();
    endfunction
endclass

// In monitor run_phase — call sample:
cov_model.txn = collected_txn;
cov_model.sample();

// evm_base_test.report_phase() — aggregates all coverage models:
// Finds all evm_coverage_model children in hierarchy, prints summary table
```

**Coverage reporting format (in report_phase):**
```
[EVM COV] ===== Coverage Summary =====
[EVM COV]   my_coverage:addr_cg       87.5%  (target: 90.0%) BELOW TARGET
[EVM COV]   my_coverage:rw_cp        100.0%  (target: 90.0%) PASS
[EVM COV] Total coverage: 87.5% — BELOW TARGET (90.0%)
```

**Note on simulator integration:** `get_inst_coverage()` is standard SystemVerilog (IEEE 1800-2012 §19). Works in Questa, VCS, Xcelium, and Vivado xsim.

**Success criteria:**
- [ ] `evm_coverage_model` compiles in Vivado/Questa
- [ ] `get_coverage()` returns correct value from a covergroup
- [ ] `report_phase()` prints the summary table
- [ ] FAIL is triggered when coverage < target
- [ ] Example shows coverage in an AXI-Lite monitor

---

### P0.3 — Expected Error / Negative Test Infrastructure

**Why it's a blocker:** Negative testing (error injection, illegal operations, fault conditions) is routine in embedded verification. Without a way to declare expected errors, any negative test will log errors and cause false failures.

**Files to modify:**
- `evm-sv/vkit/src/evm_report_handler.sv` — add expect_error/expect_warning API
- `evm-sv/vkit/src/evm_log.sv` — add `log_expected_error()` convenience wrapper

**API design:**

```systemverilog
// In evm_report_handler — new static methods:

// Register an expected error (must be seen for test to PASS)
// pattern: substring match against the full error message
// count: how many times this exact pattern is expected (default=1)
static function void expect_error(string pattern, int count=1);
static function void expect_warning(string pattern, int count=1);

// Clear all pending expectations (e.g., between test phases)
static function void clear_expected();

// Query — call in check_phase or report_phase
static function int  get_unexpected_error_count();   // errors NOT in expected list
static function int  get_unmet_expectation_count();  // expected errors never seen
static function void print_expectation_report();     // table of expected vs actual

// print_summary() — updated logic:
//   FAIL if get_error_count() > 0 AND any errors were unexpected
//   FAIL if get_unmet_expectation_count() > 0 (expected error never occurred)
//   PASS if all errors were expected AND all expectations were met

// In evm_log — convenience wrapper:
function void log_expected_error(string msg);
    // Logs as EVM_WARNING (not ERROR) to avoid incrementing error count
    // Prefixes with "[EXPECTED]" for easy grep
    log_warning($sformatf("[EXPECTED ERROR] %s", msg));
endfunction
```

**Usage pattern:**

```systemverilog
virtual task main_phase();
    super.main_phase();
    raise_objection("negative_test");
    
    // Tell EVM we expect an error response
    evm_report_handler::expect_error("AXI SLVERR on write to read-only register");
    
    // Drive the illegal operation
    env.agent.write(CSR_STATUS_ADDR, 32'hDEADBEEF, 4'hF, resp);
    
    // The SLVERR response triggers log_error() inside the scoreboard
    // Because we registered the expectation, it won't count as a failure
    
    drop_objection("negative_test");
endtask

// check_phase:
virtual function void check_phase();
    super.check_phase();
    // evm_report_handler::print_expectation_report() is called automatically
    // from evm_base_test.report_phase()
endfunction
```

**Matching rule:** Simple case-insensitive substring match. No regex required for embedded scale. If `pattern` is a substring of the logged error message, it matches.

**Success criteria:**
- [ ] `expect_error()` suppresses matching errors from failure count
- [ ] Unmet expectation (expected error never occurred) causes test FAIL
- [ ] Multiple calls to `expect_error()` with `count=N` require N occurrences
- [ ] `print_expectation_report()` shows matched/unmatched table

---

## 🟡 P1 — Strong Differentiators (Implement Second)

### P1.1 — SPI Agent

**Why it matters:** Nearly every embedded system has SPI peripherals. Engineers verifying SPI flash controllers, SPI ADCs, SPI DACs, or any SPI peripheral device need this on day 1.

**New directory:** `evm-sv/vkit/evm_vkit/evm_spi_agent/`

**Files to create:**
```
evm_spi_if.sv          — interface (SCLK, MOSI, MISO, CS_N[N])
evm_spi_cfg.sv         — config: CPOL, CPHA, frequency, word_size, cs_count
evm_spi_txn.sv         — transaction: data_out[63:0], data_in[63:0], cs_sel, word_size
evm_spi_driver.sv      — SPI master driver
evm_spi_monitor.sv     — passive bus observer
evm_spi_agent.sv       — agent (extends evm_agent, direct API)
```

**Interface:**
```systemverilog
interface evm_spi_if #(int CS_COUNT=1) (input logic clk);
    logic              sclk;
    logic              mosi;
    logic              miso;
    logic [CS_COUNT-1:0] cs_n;
    // timing parameters
    realtime setup_time = 2ns;
    realtime hold_time  = 2ns;
endinterface
```

**Config:**
```systemverilog
class evm_spi_cfg;
    int  cpol       = 0;    // Clock polarity: 0=idle low, 1=idle high
    int  cpha       = 0;    // Clock phase:    0=sample on leading edge
    int  word_size  = 8;    // Bits per transfer (8, 16, 24, 32)
    int  frequency_hz = 1_000_000;  // SPI clock frequency
    int  cs_count   = 1;    // Number of chip selects
    bit  lsb_first  = 0;    // Bit order
endclass
```

**Transaction:**
```systemverilog
class evm_spi_txn extends evm_sequence_item;
    rand logic [63:0] data_out;   // MOSI data (MSB or LSB first per cfg)
    logic [63:0]      data_in;    // MISO data captured
    rand int          cs_sel;     // Which CS to assert (0-based)
    rand int          word_size;  // Override cfg word_size (0=use cfg)
    bit               cs_error;   // CS deasserted unexpectedly
    virtual function string convert2string();
endclass
```

**Driver direct API (most common usage):**
```systemverilog
task transfer(input logic [63:0] data_out, output logic [63:0] data_in, input int cs=0);
task write_byte(input logic [7:0] data, input int cs=0);
task read_byte(output logic [7:0] data, input int cs=0);
```

**Monitor analysis ports:**
```systemverilog
evm_analysis_port#(evm_spi_txn) ap_txn;  // completed transfer
```

**Modes to support:** CPOL=0/CPHA=0 (Mode 0), CPOL=0/CPHA=1 (Mode 1), CPOL=1/CPHA=0 (Mode 2), CPOL=1/CPHA=1 (Mode 3)

**Vivado compatibility note:** Follow parameterized base class pattern — `evm_spi_agent extends evm_agent#(virtual evm_spi_if, evm_spi_txn)`

---

### P1.2 — I2C Agent

**Why it matters:** I2C is the dominant low-speed sensor/PMIC interface. Every embedded SoC has at least one I2C master.

**New directory:** `evm-sv/vkit/evm_vkit/evm_i2c_agent/`

**Files to create:**
```
evm_i2c_if.sv          — interface: SCL, SDA (open-drain modeled with tri-state)
evm_i2c_cfg.sv         — config: speed mode, addr_width, timeout
evm_i2c_txn.sv         — transaction: addr, data[], is_write, ack[], nack
evm_i2c_driver.sv      — I2C master driver (start/addr/data/stop)
evm_i2c_monitor.sv     — passive bus observer
evm_i2c_agent.sv       — agent
```

**Interface:**
```systemverilog
interface evm_i2c_if;
    wire scl;   // open-drain — use pullup in tb_top
    wire sda;   // open-drain — use pullup in tb_top
    logic scl_o, scl_en;   // drive when en=1
    logic sda_o, sda_en;
    assign scl = scl_en ? scl_o : 1'bz;
    assign sda = sda_en ? sda_o : 1'bz;
endinterface
```

**Config:**
```systemverilog
class evm_i2c_cfg;
    typedef enum { STANDARD=100, FAST=400, FAST_PLUS=1000 } i2c_speed_e; // kHz
    i2c_speed_e speed = STANDARD;
    int         addr_width = 7;   // 7 or 10 bit addressing
    int         timeout_us = 100; // Bus timeout
endclass
```

**Transaction:**
```systemverilog
class evm_i2c_txn extends evm_sequence_item;
    rand logic [9:0]  addr;       // 7-bit or 10-bit device address
    rand logic [7:0]  data[];     // byte array: write data or read data captured
    rand int          num_bytes;
    bit               is_write;
    bit               ack[];      // ACK received for each byte (1=ACK, 0=NACK)
    bit               arb_lost;   // bus arbitration lost
    virtual function string convert2string();
endclass
```

**Driver direct API:**
```systemverilog
task write(input logic [9:0] addr, input logic [7:0] data[], output bit ack);
task read(input logic [9:0] addr, input int num_bytes, output logic [7:0] data[], output bit ack);
task write_reg(input logic [9:0] dev_addr, input logic [7:0] reg_addr, input logic [7:0] data);
task read_reg(input logic [9:0] dev_addr, input logic [7:0] reg_addr, output logic [7:0] data);
```

---

### P1.3 — UART Agent

**Why it matters:** UART is the universal debug/comms interface for embedded systems. Console output, command interfaces, and protocol testing all use UART.

**New directory:** `evm-sv/vkit/evm_vkit/evm_uart_agent/`

**Files to create:**
```
evm_uart_if.sv         — interface: tx, rx (+ optional cts, rts)
evm_uart_cfg.sv        — config: baud_rate, data_bits, parity, stop_bits
evm_uart_txn.sv        — transaction: data_byte, parity_err, framing_err, break_detect
evm_uart_driver.sv     — UART transmitter
evm_uart_monitor.sv    — UART receiver + error detection
evm_uart_agent.sv      — agent
```

**Config:**
```systemverilog
class evm_uart_cfg;
    int baud_rate   = 115200;
    int data_bits   = 8;        // 7, 8, or 9
    typedef enum {NONE, ODD, EVEN, MARK, SPACE} parity_e;
    parity_e parity = NONE;
    real stop_bits  = 1.0;      // 1, 1.5, or 2
    bit  flow_control = 0;      // enable CTS/RTS
endclass
```

**Transaction:**
```systemverilog
class evm_uart_txn extends evm_sequence_item;
    rand logic [8:0] data;      // up to 9 data bits
    bit parity_error;
    bit framing_error;
    bit break_detect;
    virtual function string convert2string();
endclass
```

**Driver direct API:**
```systemverilog
task send_byte(input logic [8:0] data);
task send_string(input string str);   // sends each char as a UART byte
task send_bytes(input logic [7:0] data[]);
```

**Monitor analysis ports:**
```systemverilog
evm_analysis_port#(evm_uart_txn) ap_rx;   // received byte
evm_analysis_port#(evm_uart_txn) ap_err;  // framing/parity errors
```

---

### P1.4 — APB Agent

**Why it matters:** APB (AMBA Peripheral Bus) is the dominant register bus in ARM Cortex-M/A SoC designs on FPGA. Almost all ARM-based FPGA SoC peripherals (GPIO, UART, SPI, Timers) hang off an APB bus.

**New directory:** `evm-sv/vkit/evm_vkit/evm_apb_agent/`

**Files to create:**
```
evm_apb_if.sv          — APB3/APB4 interface
evm_apb_cfg.sv         — config: addr_width, data_width, has_strb, has_prot
evm_apb_txn.sv         — transaction: addr, data, is_write, strb, resp
evm_apb_driver.sv      — APB master (setup→access phase state machine)
evm_apb_monitor.sv     — passive APB observer (7 analysis ports)
evm_apb_agent.sv       — agent with direct API matching AXI-Lite pattern
```

**Interface (APB4):**
```systemverilog
interface evm_apb_if (input logic pclk, input logic presetn);
    logic        psel;
    logic        penable;
    logic        pwrite;
    logic [31:0] paddr;
    logic [31:0] pwdata;
    logic [31:0] prdata;
    logic        pready;
    logic        pslverr;
    logic [3:0]  pstrb;     // APB4
    logic [2:0]  pprot;     // APB4
endinterface
```

**Transaction:**
```systemverilog
class evm_apb_txn extends evm_sequence_item;
    rand logic [31:0] addr;
    rand logic [31:0] data;
    bit               is_write;
    logic [3:0]       strb;
    logic [2:0]       prot;
    bit               slverr;
    int               wait_states;  // PREADY deasserted cycles
    virtual function string convert2string();
endclass
```

**Driver direct API (matches AXI-Lite pattern for easy migration):**
```systemverilog
task write(input logic [31:0] addr, input logic [31:0] data,
           input logic [3:0] strb=4'b1111, output bit slverr);
task read(input logic [31:0] addr, output logic [31:0] data, output bit slverr);
task write_check(input logic [31:0] addr, input logic [31:0] data, input logic [3:0] strb=4'b1111);
task read_check(input logic [31:0] addr, output logic [31:0] data);
task rmw(input logic [31:0] addr, input logic [31:0] mask, input logic [31:0] value);
```

**Note:** APB state machine (IDLE → SETUP → ACCESS → IDLE) is strictly 2-cycle min per transfer.

---

### P1.5 — Backdoor / HDL Force-Release

**Why it matters:** Embedded verification frequently needs to inject faults, initialize memory (load firmware), or set DUT state without going through the bus. Pure `$force`/`$release` is sufficient for FPGA targets — no DPI required.

**New file:** `evm-sv/vkit/src/evm_backdoor.sv`  
**Files to modify:** `evm-sv/vkit/src/evm_reg.sv` (add `backdoor_write`/`backdoor_read`)  
**Package inclusion:** Add to `evm-sv/vkit/src/evm_pkg.sv`

**API design:**

```systemverilog
// evm_backdoor — static utility class
class evm_backdoor extends evm_log;
    
    // Force a signal to a value (immediate, holds until release)
    // hdl_path: hierarchical path from tb_top, e.g. "tb_top.dut.ctrl_reg"
    static task force_signal(string hdl_path, logic [63:0] value);
        // Implementation uses $force — pure SV, no DPI
        // Note: Vivado xsim supports $force on net/variable paths
    
    // Release a forced signal (returns to normal simulation)
    static task release_signal(string hdl_path);
    
    // Deposit (one-time assign, no hold) — equivalent to UVM's hdl_deposit
    // Note: $deposit is simulator-specific; document as Questa/VCS only
    static task deposit_signal(string hdl_path, logic [63:0] value);
    
    // Load a memory from a hex file (uses $readmemh)
    // mem_path: path to memory array in RTL, e.g. "tb_top.dut.rom.mem"
    static function void load_mem_hex(string mem_path, string hex_file);
    
    // Load a memory from a binary file (uses $readmemb)
    static function void load_mem_bin(string mem_path, string bin_file);
endclass

// Limitation note (document clearly):
// - $force/$release work in all major simulators for reg/logic targets
// - Reading HDL values without DPI requires simulator-specific VPI/PLI
// - For read-back, use monitor observation instead
// - Backdoor READ is NOT implemented (use evm_reg.read() frontdoor instead)

// RAL integration — add to evm_reg:
string hdl_path = "";  // set this to enable backdoor

virtual task backdoor_write(bit [63:0] value);
    if (hdl_path == "") begin
        log_error($sformatf("[BD] No hdl_path set for register %s", get_name()));
        return;
    end
    evm_backdoor::force_signal(hdl_path, value);
    // Also update mirror so RAL stays consistent
    predict(value, 0);
endtask

// No backdoor_read — log_warning if called; use frontdoor read()
```

**Usage pattern (firmware load):**
```systemverilog
virtual function void build_phase();
    super.build_phase();
    // Load firmware image into simulated ROM before simulation runs
    evm_backdoor::load_mem_hex("tb_top.u_rom.mem", "firmware.hex");
endfunction
```

---

### P1.6 — Transaction Waveform Recording

**Why it matters:** When a test fails, engineers debug in the waveform viewer. Seeing transactions (not just signals) in the waveform dramatically reduces debug time — often from hours to minutes.

**New file:** `evm-sv/vkit/src/evm_recorder.sv`  
**Package inclusion:** Add to `evm-sv/vkit/src/evm_pkg.sv`

**Approach:** Emit a structured transaction log in a format readable by GTKWave and other waveform viewers. Also provide hooks for simulator-native transaction recording where available.

**API design:**

```systemverilog
// evm_recorder — singleton transaction logger
class evm_recorder extends evm_log;
    
    // Singleton access
    static function evm_recorder get();
    
    // Enable/disable
    static function void enable(string log_file="evm_txn.log");
    static function void set_disabled();
    static function bit  is_enabled();
    
    // Transaction boundary logging (call from monitors)
    // Returns a unique transaction ID for begin/end pairing
    static function int  begin_transaction(string txn_type, string component_path, time t=$time);
    static function void end_transaction(int txn_id, string fields="", time t=$time);
    
    // One-shot transaction log (for monitors that already have full txn assembled)
    static function void record_transaction(
        string txn_type,         // e.g. "AXI_WRITE", "SPI_BYTE"
        string component_path,   // e.g. "env.axi_agent.monitor"
        time   start_t,
        time   end_t,
        string fields            // key=value pairs: "addr=0x100 data=0xDEAD resp=OK"
    );
    
    // Waveform annotation (writes VCD-like events to log)
    // start_of_simulation_phase: opens log file
    // final_phase: closes log file, prints record count
endclass

// Log format (evm_txn.log):
// #<time_ns> BEGIN <txn_id> <txn_type> <component>
// #<time_ns> END   <txn_id> <txn_type> <component> addr=0x100 data=0xDEAD resp=OK

// Usage in AXI-Lite monitor:
virtual task run_phase();
    super.run_phase();
    fork
        forever begin
            evm_axi_lite_write_txn txn;
            // ... collect transaction ...
            evm_recorder::record_transaction(
                "AXI_WRITE",
                get_full_name(),
                txn.start_time,
                txn.end_time,
                $sformatf("addr=0x%08x data=0x%08x resp=%s",
                    txn.addr, txn.data, txn.is_okay() ? "OK" : "ERR")
            );
        end
    join_none
endtask
```

**Plusarg:** `+evm_record=<filename>` enables transaction recording (similar to `+evm_log=`)

**Simulator-native notes (document):**
- Questa: `$add_attribute` / wave transactions panel
- VCS: `$vcd_define_transaction_region` / `$vcd_add_transaction`
- Vivado: No native transaction recording; use the evm_txn.log format

---

## 🟢 P2 — Nice to Have

### P2.1 — Sequence Priority / Arbitration

**Why it matters:** Multi-master embedded scenarios (AXI master + DMA engine + CPU model on same bus) need basic fairness and priority control.

**Files to modify:** `evm-sv/vkit/src/evm_sequencer.sv`

**API additions:**

```systemverilog
// Priority enum
typedef enum { EVM_SEQ_HIGH=0, EVM_SEQ_NORMAL=1, EVM_SEQ_LOW=2 } evm_seq_priority_e;

// In evm_seq_item_pull_export — add priority variant:
task put_with_priority(REQ req, evm_seq_priority_e priority=EVM_SEQ_NORMAL);

// In evm_sequencer — priority queues:
mailbox#(REQ) high_q, normal_q, low_q;

// Arbitration: drain high_q first, then normal_q, then low_q
// Simple mutex for exclusive window:
task lock(string owner_name);    // grants exclusive access; others block
task unlock(string owner_name);  // releases mutex
```

---

### P2.2 — CI/CD Integration (JUnit XML Output)

**Why it matters:** Modern engineering teams use GitHub Actions, Jenkins, or GitLab CI. JUnit XML is the universal test result format all CI/CD tools understand.

**Files to modify:** `evm-sv/vkit/src/evm_report_handler.sv`

**API additions:**

```systemverilog
// Enable JUnit XML output — call in build_phase or before report_phase
static function void enable_junit_output(string filename="evm_results.xml");

// Called automatically from print_summary() when enabled
static function void write_junit_xml();

// JUnit XML format:
// <testsuites>
//   <testsuite name="EVM" tests="1" failures="0" errors="0" time="0.001">
//     <testcase name="basic_write_test" time="0.001">
//       <!-- on failure: -->
//       <failure message="1 error(s) detected">...</failure>
//     </testcase>
//   </testsuite>
// </testsuites>
```

**Plusarg:** `+evm_junit=<filename>` enables JUnit output from command line

**Makefile template:** `evm-sv/sim/Makefile.template` — standard targets:
- `make compile` — compile evm_pkg + evm_vkit_pkg + tb
- `make sim TESTNAME=basic_write_test` — run specific test
- `make regression` — run all registered tests
- `make clean`

---

## 🔵 P3 — Stretch Goal: ISS / Firmware Co-simulation Agent

### P3.1 — `evm_iss_agent` (RISC-V behavioral CPU model)

**Why it's transformational:** No other lightweight verification framework addresses firmware co-simulation. UVM has no equivalent. This would be EVM's unique differentiator — the ability to run actual firmware binaries alongside RTL simulation.

**New directory:** `evm-sv/vkit/evm_vkit/evm_iss_agent/`

**Concept:**
- A behavioral RISC-V ISS (Instruction Set Simulator) that drives the DUT's AXI bus as if a CPU were executing instructions
- Phase 1: Bus-Functional Model (BFM) — load hex/ELF, drive AXI reads/writes per instruction
- Phase 2: Connect to external Spike (RISC-V ISS) via DPI or Unix domain socket

**Files to create (Phase 1 — BFM):**
```
evm_iss_cfg.sv         — config: elf_file, hex_file, isa, reset_vector, stack_pointer
evm_iss_memory.sv      — extends evm_memory_model (load ELF sections)
evm_iss_core.sv        — behavioral RV32I/RV64I decoder + executor
evm_iss_driver.sv      — drives DUT AXI bus based on ISS memory requests
evm_iss_monitor.sv     — observes DUT responses, feeds back to ISS
evm_iss_agent.sv       — agent: load_elf(), run(), step(), get_pc()
```

**ISS agent API:**
```systemverilog
class evm_iss_cfg;
    string elf_file    = "";       // path to RISC-V ELF binary
    string hex_file    = "";       // alternative: Intel hex format
    bit [63:0] reset_vector = 64'h8000_0000;  // PC on reset
    bit [63:0] stack_pointer = 64'h9000_0000; // SP initial value
    string isa = "rv32imc";       // RISC-V ISA string
    int    max_instructions = 0;  // 0 = unlimited; set for bounded sims
endclass

// evm_iss_agent — main interface:
function void   load_elf(string path);       // parse ELF, load sections
function void   load_hex(string path);       // load Intel hex format
function void   set_pc(bit [63:0] addr);     // set program counter
function bit [63:0] get_pc();               // read current PC
task            step();                      // execute one instruction
task            run_until_halt();           // run until HALT or max_instructions
task            run_until_addr(bit [63:0] pc); // run until PC == addr (breakpoint)
function void   dump_registers();            // log all GPR values
function string get_disasm(bit [63:0] pc);  // disassemble at PC
```

**Integration with AXI-Lite agent:**
- `evm_iss_driver` holds a reference to `evm_axi_lite_master_agent` (or `evm_axi4_full_master_agent`)
- When ISS executes a LOAD instruction → driver calls `axi_agent.read(addr)`
- When ISS executes a STORE instruction → driver calls `axi_agent.write(addr, data)`
- ISS instruction fetch → driver calls `axi_agent.read(pc)` → returns instruction word

**Phase 2 (future) — Spike integration:**
- Connect to Spike ISS via DPI socket (Unix domain socket or named pipe)
- Send memory requests from Spike → receive AXI responses from EVM driver
- Allows running real GCC-compiled RISC-V firmware

**Success criteria for Phase 1:**
- [ ] `evm_iss_core` executes RV32I base instruction set correctly
- [ ] Load/store instructions drive AXI transactions to DUT
- [ ] `load_hex()` loads a simple firmware binary
- [ ] Basic firmware loop (write peripheral register, read status, loop) works in simulation
- [ ] Example 3 demonstrates firmware co-simulation with example1 DUT

---

## 🛑 Deliberately NOT Implementing (ASIC Overkill for Embedded)

| UVM Feature | Reason to Skip |
|-------------|----------------|
| `uvm_factory` + type overrides | Direct `new()` + registry is simpler and sufficient |
| `uvm_config_db` | Direct VIF passing is better — eliminates a major source of UVM bugs |
| TLM 2.0 sockets (b_transport) | Embedded uses AXI/APB, not generic payload model |
| `uvm_callback` framework | 3-class overhead not worth it at embedded scale |
| Phase domains (power domains) | ASIC multi-domain — not relevant for FPGA embedded |
| Full DPI/C++ integration | Keep pure SV; DPI only if needed for P3 ISS |
| UVM heartbeat | EVM's timeout + QC already covers stuck-simulation detection |
| `uvm_packer` / bit-stream packing | Not needed for embedded protocol verification |
| `uvm_printer` policy objects | `convert2string()` is sufficient |
| Formal verification integration | Outside EVM scope |

---

## 📋 Implementation Notes & Vivado Compatibility Rules

All new EVM code MUST follow these rules to maintain Vivado xsim compatibility:

1. **No `reg` as variable name** — use `csr` or `rg` instead (SV keyword conflict)
2. **No `disable` as method name** — use `set_disabled()` instead
3. **No `#delay` inside functions** — only in tasks
4. **Local variable declarations at TOP of task/function** — before any statements
5. **No `automatic` on variables inside class methods** — already implicit in classes
6. **`EVM_REGISTER_TEST` macro (contains `initial`)** — must be at MODULE scope, NOT inside packages
7. **Non-parameterized virtual interfaces inside packages fail** — always use type parameter pattern: `class my_agent extends evm_agent#(virtual my_if, my_txn)`
8. **Variables in module `initial` blocks need `automatic` qualifier**
9. **`virtual class` cannot be instantiated with `new()`** — provide concrete subclasses or factory methods

---

## 🗂️ Files Reference (What Will Be Created)

### Core vkit additions (evm_pkg)
```
evm-sv/vkit/src/evm_rand_sequence.sv      # P0.1 — CRV base sequence
evm-sv/vkit/src/evm_backdoor.sv           # P1.5 — HDL force/release
evm-sv/vkit/src/evm_recorder.sv           # P1.6 — transaction waveform logging
```

### Core vkit modifications (evm_pkg)
```
evm-sv/vkit/src/evm_coverage.sv           # P0.2 — implement stub → real base class
evm-sv/vkit/src/evm_report_handler.sv     # P0.3 + P2.2 — expect_error + JUnit
evm-sv/vkit/src/evm_sequence_item.sv      # P0.1 — add randomize_item()
evm-sv/vkit/src/evm_reg.sv                # P1.5 — add backdoor_write()
evm-sv/vkit/src/evm_sequencer.sv          # P2.1 — priority queues + lock/unlock
evm-sv/vkit/src/evm_pkg.sv                # include new files
```

### New protocol agents (evm_vkit)
```
evm-sv/vkit/evm_vkit/evm_spi_agent/      # P1.1 — SPI master
evm-sv/vkit/evm_vkit/evm_i2c_agent/      # P1.2 — I2C master
evm-sv/vkit/evm_vkit/evm_uart_agent/     # P1.3 — UART
evm-sv/vkit/evm_vkit/evm_apb_agent/      # P1.4 — APB master
evm-sv/vkit/evm_vkit/evm_iss_agent/      # P3.1 — ISS/firmware co-sim
evm-sv/vkit/evm_vkit/evm_vkit_pkg.sv     # include new agents
```

### Simulation infrastructure
```
evm-sv/sim/Makefile.template              # P2.2 — CI/CD Makefile
```
