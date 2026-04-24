# CLAUDE_UVM.md — UVM Deep Reference
# Source: uvm/uvm-core/ (Accellera IEEE 1800.2-2020)

**Last Updated:** 2026-04-23  
**Purpose:** Compressed reference of what UVM is and what it contains. Use for comparison with EVM.

---

## 🎯 What UVM Is

**Universal Verification Methodology (UVM)** — IEEE standard 1800.2-2020  
Industry standard for ASIC/FPGA functional verification using SystemVerilog.  
Developed by Accellera. Supported by all major simulators (Synopsys VCS, Cadence Xcelium, Mentor Questa, Xilinx XSim).

**Core philosophy:** Reusable, portable, scalable verification components. Any team can drop in a UVM-compliant VIP (Verification IP) from any vendor and it will work.

---

## 📁 Folder Structure: `uvm/uvm-core/src/`

```
src/
├── uvm_pkg.sv              # Top-level package (includes everything)
├── uvm.sv                  # Module wrapper
├── uvm_macros.svh          # All UVM macros (one include)
│
├── base/                   # Core infrastructure (~30 files)
│   ├── uvm_object.svh      # Base data object
│   ├── uvm_component.svh   # Base component (3780 lines!)
│   ├── uvm_root.svh        # Top-level singleton, run_test()
│   ├── uvm_phase.svh       # Phase scheduler
│   ├── uvm_common_phases.svh # build/connect/run/extract/check/report/final
│   ├── uvm_runtime_phases.svh # pre/post_reset, configure, pre/post_main, pre/post_shutdown
│   ├── uvm_objection.svh   # Objection mechanism
│   ├── uvm_factory.svh     # Factory + override system
│   ├── uvm_config_db.svh   # Configuration database
│   ├── uvm_resource.svh    # Resource pool
│   ├── uvm_resource_db.svh # Resource database
│   ├── uvm_report_*.svh    # Reporting (handler, server, message, catcher)
│   ├── uvm_callback.svh    # Callback mechanism
│   ├── uvm_event.svh       # UVM events + event callbacks
│   ├── uvm_barrier.svh     # Synchronization barrier
│   ├── uvm_heartbeat.svh   # Stuck-component detector
│   ├── uvm_domain.svh      # Phase domains (power-domain support)
│   ├── uvm_transaction.svh # Transaction base (for recording)
│   ├── uvm_printer.svh     # Print policy (table/tree/line printers)
│   ├── uvm_comparer.svh    # Compare policy
│   ├── uvm_copier.svh      # Copy policy
│   ├── uvm_packer.svh      # Pack/unpack to bit streams
│   ├── uvm_recorder.svh    # Transaction recording
│   ├── uvm_pool.svh        # Generic pool/queue
│   └── uvm_misc.svh        # Global utilities, uvm_void
│
├── comps/                  # Standard components
│   ├── uvm_agent.svh       # Agent base (is_active via config_db)
│   ├── uvm_driver.svh      # Driver base (seq_item_port)
│   ├── uvm_monitor.svh     # Monitor base (analysis_port)
│   ├── uvm_sequencer.svh   # Sequencer (parameterized)
│   ├── uvm_env.svh         # Environment base
│   ├── uvm_test.svh        # Test base
│   ├── uvm_scoreboard.svh  # Scoreboard base
│   ├── uvm_subscriber.svh  # Analysis subscriber (analysis_export auto-created)
│   ├── uvm_push_driver.svh # Push-mode driver variant
│   ├── uvm_random_stimulus.svh # Random item generator
│   ├── uvm_algorithmic_comparator.svh # Comparator with transform
│   └── uvm_in_order_comparator.svh  # In-order comparator
│
├── seq/                    # Sequences and sequencers
│   ├── uvm_sequence_item.svh       # Base sequence item
│   ├── uvm_sequence_base.svh       # Sequence base (body, start, etc.)
│   ├── uvm_sequence.svh            # Parameterized sequence #(REQ,RSP)
│   ├── uvm_sequencer.svh           # Parameterized sequencer #(REQ,RSP)
│   ├── uvm_sequencer_base.svh      # Sequencer base
│   ├── uvm_sequencer_param_base.svh # Param sequencer base
│   ├── uvm_push_sequencer.svh      # Push-mode sequencer
│   └── uvm_sequence_library.svh    # Sequence library
│
├── tlm1/                   # TLM 1.0
│   ├── uvm_analysis_port.svh       # Analysis port (1-to-many)
│   ├── uvm_ports.svh               # put/get/peek/transport ports
│   ├── uvm_exports.svh             # put/get/peek/transport exports
│   ├── uvm_imps.svh                # put/get/peek/transport imps
│   ├── uvm_tlm_fifos.svh           # TLM FIFOs
│   ├── uvm_sqr_connections.svh     # Sequencer-driver connection
│   └── uvm_sqr_ifs.svh             # Sequencer interface
│
├── tlm2/                   # TLM 2.0
│   ├── uvm_tlm2_generic_payload.svh # AMBA-like payload struct
│   ├── uvm_tlm2_ifs.svh            # b_transport/nb_transport interfaces
│   ├── uvm_tlm2_ports.svh          # TLM2 ports
│   ├── uvm_tlm2_exports.svh        # TLM2 exports
│   ├── uvm_tlm2_sockets.svh        # Initiator/target sockets
│   └── uvm_tlm_time.svh            # Time annotation
│
├── reg/                    # Register Abstraction Layer (RAL)
│   ├── uvm_reg_model.svh           # Top include + global types
│   ├── uvm_reg_field.svh           # Register field
│   ├── uvm_reg.svh                 # Register (holds fields)
│   ├── uvm_reg_block.svh           # Register block
│   ├── uvm_reg_map.svh             # Address map
│   ├── uvm_mem.svh                 # Memory model
│   ├── uvm_vreg.svh                # Virtual register
│   ├── uvm_vreg_field.svh          # Virtual register field
│   ├── uvm_reg_file.svh            # Register file (subblock)
│   ├── uvm_reg_item.svh            # Register access item
│   ├── uvm_reg_adapter.svh         # Bus adapter (protocol bridge)
│   ├── uvm_reg_predictor.svh       # Auto-predict from bus monitor
│   ├── uvm_reg_backdoor.svh        # HDL backdoor access
│   ├── uvm_reg_cbs.svh             # Register callbacks
│   ├── uvm_reg_sequence.svh        # Register sequence base
│   ├── uvm_mem_mam.svh             # Memory Allocation Manager
│   └── sequences/                  # Built-in test sequences
│       ├── uvm_reg_hw_reset_seq.svh  # HW reset check
│       ├── uvm_reg_bit_bash_seq.svh  # Bit-bash walk
│       ├── uvm_reg_access_seq.svh    # Read/write access
│       ├── uvm_mem_walk_seq.svh      # Memory walk
│       ├── uvm_mem_access_seq.svh    # Memory access
│       └── uvm_reg_mem_shared_access_seq.svh
│
├── macros/                 # All UVM macros
│   ├── uvm_object_defines.svh      # `uvm_object_utils, `uvm_field_*
│   ├── uvm_message_defines.svh     # `uvm_info, `uvm_warning, `uvm_error, `uvm_fatal
│   ├── uvm_sequence_defines.svh    # `uvm_do, `uvm_create, `uvm_send
│   ├── uvm_tlm_defines.svh         # TLM connection macros
│   ├── uvm_reg_defines.svh         # RAL macros
│   ├── uvm_callback_defines.svh    # Callback macros
│   └── uvm_global_defines.svh      # `UVM_REG_DATA_WIDTH, timeouts, etc.
│
├── dpi/                    # C/C++ DPI integration
│   ├── uvm_dpi.cc / .h             # DPI functions
│   ├── uvm_hdl.c / .svh            # HDL backdoor: read/force/release/deposit
│   ├── uvm_regex.cc / .svh         # Regex matching
│   └── uvm_svcmd_dpi.c / .svh      # Simulator command interface
│
└── dap/                    # Data Access Policies (advanced)
    ├── uvm_get_to_lock_dap.svh     # Lock after first get
    ├── uvm_set_before_get_dap.svh  # Must set before get
    └── uvm_simple_lock_dap.svh     # Simple lock
```

---

## 🏗️ Core Classes — Key Methods & Tasks

### `uvm_void` (base of all)
- Empty base class. All UVM classes derive from this.

### `uvm_object` (extends uvm_void)
**Data object base — no hierarchy**

```systemverilog
// Identification
virtual function string get_name();
virtual function void   set_name(string name);
virtual function string get_full_name();    // same as get_name for objects
virtual function string get_type_name();    // must override
static  function int    get_inst_count();
virtual function int    get_inst_id();
static  function uvm_object_wrapper get_type();  // for factory
virtual function uvm_object_wrapper get_object_type();

// Creation
virtual function uvm_object create(string name="");  // must override for clone
virtual function uvm_object clone();                  // calls create() then copy()

// Printing
function        void   print(uvm_printer printer=null);
function        string sprint(uvm_printer printer=null);
virtual function void  do_print(uvm_printer printer);   // override to customize

// Copy
function        void   copy(uvm_object rhs, uvm_copier copier=null);
virtual function void  do_copy(uvm_object rhs);          // override to implement

// Compare
function        bit    compare(uvm_object rhs, uvm_comparer comparer=null);
virtual function bit   do_compare(uvm_object rhs, uvm_comparer comparer); // override

// Packing
function int pack(ref bit bitstream[], input uvm_packer packer=null);
function int pack_bytes(ref byte unsigned bytestream[], input uvm_packer packer=null);
function int pack_ints(ref int unsigned intstream[], input uvm_packer packer=null);
virtual function void do_pack(uvm_packer packer);

// Unpacking
function int unpack(ref bit bitstream[], input uvm_packer packer=null);
function int unpack_bytes(ref byte unsigned bytestream[], input uvm_packer packer=null);
virtual function void do_unpack(uvm_packer packer);

// Recording
function void record(uvm_recorder recorder=null);
virtual function void do_record(uvm_recorder recorder);

// String
virtual function string convert2string();  // user-definable debug string
```

**Key macros for uvm_object:**
```systemverilog
`uvm_object_utils(MyClass)           // registers type, implements get_type/get_type_name/create
`uvm_object_param_utils(MyClass#(T)) // for parameterized classes
`uvm_field_int(field, flags)         // auto copy/compare/print for int field
`uvm_field_string(field, flags)
`uvm_field_object(field, flags)
`uvm_field_array_int(field, flags)
```

---

### `uvm_report_object` (extends uvm_object)
**Adds reporting to any class**

```systemverilog
// Reporting (also available as global functions)
function void uvm_report_info(string id, string message, int verbosity=UVM_MEDIUM, ...);
function void uvm_report_warning(string id, string message, ...);
function void uvm_report_error(string id, string message, ...);
function void uvm_report_fatal(string id, string message, ...);

// Configuration
function void set_report_verbosity_level(int verbosity_level);
function void set_report_verbosity_level_hier(int verbosity_level); // recursive
function void set_report_id_verbosity(string id, int verbosity);
function void set_report_severity_action(uvm_severity severity, uvm_action action);
function void set_report_id_action(string id, uvm_action action);
function void set_report_file(UVM_FILE file);
```

**UVM Actions:** `UVM_NO_ACTION`, `UVM_DISPLAY`, `UVM_LOG`, `UVM_COUNT`, `UVM_EXIT`, `UVM_CALL_HOOK`, `UVM_STOP`  
**UVM Verbosity:** `UVM_NONE=0`, `UVM_LOW=100`, `UVM_MEDIUM=200`, `UVM_HIGH=300`, `UVM_FULL=400`, `UVM_DEBUG=500`

**Macros:**
```systemverilog
`uvm_info("ID", "message", UVM_LOW)
`uvm_warning("ID", "message")
`uvm_error("ID", "message")
`uvm_fatal("ID", "message")
```

---

### `uvm_component` (extends uvm_report_object)
**Hierarchical component base — 3780 lines in source!**

```systemverilog
// Constructor (REQUIRED)
function new(string name, uvm_component parent);

// Hierarchy
virtual function uvm_component get_parent();
virtual function string        get_full_name();
        function void          get_children(ref uvm_component children[$]);
        function uvm_component get_child(string name);
        function int           get_first_child(ref string name);
        function int           get_next_child(ref string name);
        function int           get_num_children();
        function int           has_child(string name);
        function uvm_component lookup(string name);
        function int unsigned  get_depth();

// Standard Phases (all take uvm_phase arg!)
virtual function void build_phase(uvm_phase phase);          // create children
virtual function void connect_phase(uvm_phase phase);        // connect TLM
virtual function void end_of_elaboration_phase(uvm_phase phase);
virtual function void start_of_simulation_phase(uvm_phase phase);
virtual task          run_phase(uvm_phase phase);            // continuous execution
// Runtime sub-phases (parallel to run_phase):
virtual task pre_reset_phase(uvm_phase phase);
virtual task reset_phase(uvm_phase phase);
virtual task post_reset_phase(uvm_phase phase);
virtual task pre_configure_phase(uvm_phase phase);
virtual task configure_phase(uvm_phase phase);
virtual task post_configure_phase(uvm_phase phase);
virtual task pre_main_phase(uvm_phase phase);
virtual task main_phase(uvm_phase phase);
virtual task post_main_phase(uvm_phase phase);
virtual task pre_shutdown_phase(uvm_phase phase);
virtual task shutdown_phase(uvm_phase phase);
virtual task post_shutdown_phase(uvm_phase phase);
// Cleanup phases:
virtual function void extract_phase(uvm_phase phase);
virtual function void check_phase(uvm_phase phase);
virtual function void report_phase(uvm_phase phase);
virtual function void final_phase(uvm_phase phase);

// Phase callbacks
virtual function void phase_started(uvm_phase phase);
virtual function void phase_ready_to_end(uvm_phase phase);
virtual function void phase_ended(uvm_phase phase);

// Phase domains (power domains)
        function void       set_domain(uvm_domain domain, int hier=1);
        function uvm_domain get_domain();

// Configuration (legacy set/get — use uvm_config_db instead)
virtual function void set_config_int(string inst_name, string field_name, uvm_bitstream_t value);
virtual function void set_config_string(string inst_name, string field_name, string value);
virtual function void set_config_object(string inst_name, string field_name, uvm_object value, bit clone=1);
virtual function bit  get_config_int(string field_name, inout uvm_bitstream_t value);
virtual function bit  get_config_string(string field_name, inout string value);
virtual function bit  get_config_object(string field_name, inout uvm_object value, input bit clone=1);
virtual function void apply_config_settings(bit verbose=0);  // called in build_phase
virtual function bit  use_automatic_config();

// Objection callbacks (called when objections change under this component)
virtual function void raised(uvm_objection obj, uvm_object src, string desc, int count);
virtual function void dropped(uvm_objection obj, uvm_object src, string desc, int count);
virtual task          all_dropped(uvm_objection obj, uvm_object src, string desc, int count);

// Misc
virtual task   suspend();
virtual task   resume();
        function void resolve_bindings();
        function void print_config(bit recurse=0, bit audit=0);
```

**Key macro:**
```systemverilog
`uvm_component_utils(MyClass)
`uvm_component_param_utils(MyClass#(T))
```

---

### `uvm_root` (extends uvm_component)
**Singleton — top-level phase controller and implicit parent**

```systemverilog
// Access
static function uvm_root get();   // also: uvm_top global variable

// Run test (called from tb_top initial block)
virtual task run_test(string test_name="");  // reads +UVM_TESTNAME

// Simulation control
virtual function void die();
        function void set_timeout(time timeout, bit overridable=1);
        bit finish_on_completion = 1;

// Topology
        function uvm_component find(string comp_match);
        function void find_all(string comp_match, ref uvm_component comps[$], input uvm_component comp=null);
        function void print_topology(uvm_printer printer=null);
        bit enable_print_topology = 0;
```

---

### `uvm_factory` (abstract) / `uvm_default_factory`
**Type substitution system — the "secret sauce" of UVM reuse**

```systemverilog
// Access
static function uvm_factory get();
static function void        set(uvm_factory f);

// Registration (called by `uvm_*_utils macros)
pure virtual function void register(uvm_object_wrapper obj);

// Overrides
pure virtual function void set_type_override_by_type(uvm_object_wrapper orig, uvm_object_wrapper ovr, bit replace=1);
pure virtual function void set_type_override_by_name(string orig_name, string ovr_name, bit replace=1);
pure virtual function void set_inst_override_by_type(uvm_object_wrapper orig, uvm_object_wrapper ovr, string full_inst_path);
pure virtual function void set_inst_override_by_name(string orig, string ovr, string full_inst_path);
pure virtual function void set_type_alias(string alias_name, uvm_object_wrapper orig_type);

// Creation
pure virtual function uvm_object    create_object_by_type(uvm_object_wrapper req_type, string parent_path="", string name="");
pure virtual function uvm_component create_component_by_type(uvm_object_wrapper req_type, string parent_path="", string name, uvm_component parent);
pure virtual function uvm_object    create_object_by_name(string req_type_name, string parent_path="", string name="");
pure virtual function uvm_component create_component_by_name(string req_type_name, string parent_path="", string name, uvm_component parent);

// Debug
pure virtual function void debug_create_by_type(uvm_object_wrapper req_type, ...);
pure virtual function void debug_create_by_name(string req_type_name, ...);
pure virtual function void print(int all_types=1);
```

---

### `uvm_config_db` (key UVM mechanism)
**Hierarchical configuration database — replaces EVM's direct VIF passing**

```systemverilog
// Set configuration anywhere in the hierarchy
uvm_config_db#(virtual my_if)::set(this, "env.agent.*", "vif", my_vif);
uvm_config_db#(int)::set(this, "env.agent", "is_active", UVM_ACTIVE);

// Get configuration (called from build_phase)
if (!uvm_config_db#(virtual my_if)::get(this, "", "vif", vif))
    `uvm_fatal("NOVIF", "No VIF found in config_db")
```

---

### `uvm_phase`
**Phase object passed to every phase method**

```systemverilog
// Objection mechanism (primary way to control test duration)
phase.raise_objection(this, "starting test");
phase.drop_objection(this, "test done");
phase.get_objection_count(this);

// Phase info
function string    get_name();
function uvm_phase get_parent();
function bit       is_before(uvm_phase phase);
function bit       is_after(uvm_phase phase);

// Phase jumping (advanced)
function void jump(uvm_phase phase);  // jump to a different phase
```

---

### Components: `uvm_agent`, `uvm_driver`, `uvm_monitor`, `uvm_sequencer`

```systemverilog
// uvm_agent (virtual class, extends uvm_component)
uvm_active_passive_enum is_active = UVM_ACTIVE;  // set via config_db
virtual function uvm_active_passive_enum get_is_active();

// uvm_driver #(REQ, RSP=REQ) (extends uvm_component)
uvm_seq_item_pull_port #(REQ,RSP) seq_item_port;  // connect to sequencer
// In run_phase:
//   seq_item_port.get_next_item(req);    // blocking get
//   seq_item_port.try_next_item(req);    // non-blocking
//   seq_item_port.item_done();           // signal done
//   seq_item_port.item_done(rsp);        // with response
//   seq_item_port.put(rsp);              // send response
//   seq_item_port.peek(req);             // look without consuming

// uvm_monitor (virtual class, extends uvm_component)
uvm_analysis_port #(T) ap;  // broadcasts collected transactions

// uvm_sequencer #(REQ, RSP=REQ) (extends uvm_sequencer_param_base)
uvm_seq_item_pull_export #(REQ,RSP) seq_item_export;  // driver connects here
// Methods:
task wait_for_grant(uvm_sequence_base sequence_ptr, int item_priority=-1, bit lock_request=0);
task wait_for_item_done(uvm_sequence_base sequence_ptr, int transaction_id);
task wait_for_sequences();
function uvm_sequence_base get_current_grabber();
function bit               is_grabbed();
task                       lock(uvm_sequence_base seq);
task                       unlock(uvm_sequence_base seq);
task                       grab(uvm_sequence_base seq);
task                       ungrab(uvm_sequence_base seq);
```

---

## 📜 Sequences

```systemverilog
// uvm_sequence_item (extends uvm_transaction)
int unsigned  get_transaction_id();
function void set_id_info(uvm_sequence_item item);
function void set_sequencer(uvm_sequencer_base sequencer);

// uvm_sequence #(REQ, RSP=REQ) (extends uvm_sequence_base)
// Key task — override body() in derived sequences
virtual task body();   // implement test stimulus here

// Run a sequence on a sequencer:
my_seq.start(sequencer_handle);  // blocking call

// Inside body():
`uvm_do(req)                  // create + randomize + start_item + finish_item
`uvm_do_with(req, {req.addr == 32'h0;})
`uvm_create(req)              // just create
`uvm_send(req)                // just send (no randomize)
req = REQ::type_id::create("req");
start_item(req);              // manual flow
if (!req.randomize()) `uvm_fatal(...)
finish_item(req);

// Get response
get_response(rsp);           // blocking

// Sequence properties
uvm_sequencer_base p_sequencer;  // set by start()
uvm_sequence_base  parent_sequence;
int                get_priority();
function string    get_full_name();  // includes sequencer path
```

---

## 📊 Register Abstraction Layer (RAL)

### Classes

```
uvm_reg_block       — Block of registers; has a uvm_reg_map; lock_model() to use
uvm_reg             — One register; has fields
uvm_reg_field       — One field within a register
uvm_reg_map         — Address map; maps registers to bus addresses
uvm_mem             — Memory model (large arrays)
uvm_vreg            — Virtual register (reg spread across memory)
uvm_reg_file        — Named group of registers (sub-block without address)
uvm_reg_item        — Transaction passed to adapter
uvm_reg_adapter     — Converts uvm_reg_item ↔ bus-specific sequence_item
uvm_reg_predictor   — Observes bus and auto-updates mirror
uvm_reg_backdoor    — Direct HDL signal access for read/write
uvm_reg_cbs         — Pre/post read/write callbacks
```

### Key `uvm_reg_block` methods
```systemverilog
// Setup
virtual function void build();   // override to create regs + maps
function void lock_model();       // must call before use!
function uvm_reg_map create_map(string name, uvm_reg_addr_t base_addr, int n_bytes, uvm_endianness_e endian, bit byte_addressing=1);
function void add_reg(uvm_reg rg, uvm_reg_addr_t offset, string rights="RW", ...);

// Access
function uvm_reg    get_reg_by_name(string name);
function uvm_reg_map get_map_by_name(string name);
function void get_regs(ref uvm_reg regs[$]);
function void get_maps(ref uvm_reg_map maps[$]);

// Operations
virtual task reset(string kind="HARD");
virtual task do_check();
function uvm_status_e predict(uvm_reg_data_t value, ...);
function void set_hdl_path_root(string path, string kind="RTL");
```

### Key `uvm_reg` methods
```systemverilog
// Setup
function void configure(uvm_reg_block parent, uvm_reg_file regfile=null, string hdl_path="");
function uvm_reg_field add_field(string name, int size, int lsb_pos, string access, uvm_reg_data_t reset=0, bit has_reset=1, ...);

// Frontdoor read/write (via bus)
virtual task write(output uvm_status_e status, input uvm_reg_data_t value, input uvm_door_e path=UVM_DEFAULT_DOOR, uvm_reg_map map=null, ...);
virtual task read(output uvm_status_e status, output uvm_reg_data_t value, input uvm_door_e path=UVM_DEFAULT_DOOR, uvm_reg_map map=null, ...);
virtual task update(output uvm_status_e status, input uvm_door_e path=UVM_DEFAULT_DOOR, ...);  // write if mirror != desired
virtual task mirror(output uvm_status_e status, input uvm_check_e check=UVM_NO_CHECK, ...);  // read and optionally check

// Backdoor access
virtual task poke(output uvm_status_e status, input uvm_reg_data_t value, ...);   // direct HDL write
virtual task peek(output uvm_status_e status, output uvm_reg_data_t value, ...);  // direct HDL read

// Mirror management
function uvm_reg_data_t get();        // get mirrored value
function void           set(uvm_reg_data_t value);  // set desired value (not bus)
virtual function bit    predict(uvm_reg_data_t value, uvm_reg_byte_en_t be=-1, uvm_predict_e kind=UVM_PREDICT_DIRECT, ...);
virtual function void   reset(string kind="HARD");
function uvm_reg_data_t get_reset(string kind="HARD");
function uvm_reg_field  get_field_by_name(string name);

// Status
function bit needs_update();  // mirror != desired
```

### Key `uvm_reg_field` access types
```
RO, RW, RC, RS, WRC, WRS, WC, WS, W1C, W1S, W1T, W0C, W0S, W0T, W1SRC, W1CRS, W0SRC, W0CRS, WSRC, WCRS, WO, WOC, WOS, WO1, NOACCESS
```

### `uvm_reg_adapter` — Bridge between RAL and bus protocol
```systemverilog
// Must implement both (one converts RAL→bus, other converts bus→RAL)
pure virtual function uvm_sequence_item reg2bus(const ref uvm_reg_bus_op rw);
pure virtual function void              bus2reg(uvm_sequence_item bus_item, ref uvm_reg_bus_op rw);
bit supports_byte_enable;   // set in constructor if protocol supports byte enables
bit provides_responses;     // set if driver provides responses (not just write)
```

### `uvm_reg_predictor #(BUSTYPE)` — Auto-mirror updater
```systemverilog
// Connect in env connect_phase:
predictor.map = reg_map_handle;
predictor.adapter = adapter_handle;
bus_agent.monitor.analysis_port.connect(predictor.bus_in);
// Predictor receives every observed bus transaction and updates the register mirror
```

### Built-in RAL test sequences
```systemverilog
uvm_reg_hw_reset_seq    — reads all regs, checks reset values
uvm_reg_bit_bash_seq    — walks each bit for writable fields
uvm_reg_access_seq      — writes/reads all regs via frontdoor
uvm_mem_walk_seq        — walks every memory location
uvm_mem_access_seq      — reads/writes memory via frontdoor
uvm_reg_mem_shared_access_seq — tests shared regs between maps
```

---

## 📡 TLM 1.0

### Analysis (1-to-many broadcast)
```systemverilog
uvm_analysis_port #(T) ap;      // in monitor — broadcasts to subscribers
uvm_analysis_imp  #(T, IMP) ai; // in subscriber — receives via write() callback
uvm_analysis_export #(T) ae;    // in components that have internal imps
// Connection:
monitor.ap.connect(scoreboard.ai);

// In monitor: ap.write(txn);
// In scoreboard: function void write(T t); ... endfunction  // auto-called!
```

**Multiple analysis ports in one component:**
```systemverilog
`uvm_analysis_imp_decl(_wr)  // generates uvm_analysis_imp_wr #(T, IMP)
`uvm_analysis_imp_decl(_rd)  // generates uvm_analysis_imp_rd #(T, IMP)
// Then:
uvm_analysis_imp_wr #(write_txn, my_scoreboard) wr_imp;
uvm_analysis_imp_rd #(read_txn, my_scoreboard)  rd_imp;
// Callbacks: write_wr(write_txn t) and write_rd(read_txn t)
```

### Put/Get (point-to-point)
```systemverilog
// Ports (master side):
uvm_put_port  #(T) pp;   // put(), try_put(), can_put()
uvm_get_port  #(T) gp;   // get(), try_get(), can_get(), peek(), try_peek()
uvm_blocking_get_port #(T) bgp;

// Exports/Imps (slave side):
uvm_put_export  #(T) pe;
uvm_get_export  #(T) ge;
uvm_put_imp     #(T, IMP) pi;  // IMP implements put() task

// TLM FIFOs
uvm_tlm_fifo #(T) fifo;
```

### Sequencer-Driver connection (TLM)
```systemverilog
// Driver connects to sequencer via:
seq_item_port.connect(sequencer.seq_item_export);
// This creates a pull channel: driver calls get_next_item/item_done
// while sequencer manages sequence execution and arbitration
```

---

## 📡 TLM 2.0 (Generic Payload)

```systemverilog
// Generic payload (AMBA-inspired)
uvm_tlm_generic_payload gp;
gp.set_address(addr);         gp.get_address()
gp.set_command(UVM_TLM_WRITE_COMMAND); // UVM_TLM_READ_COMMAND
gp.set_data_ptr(data_array);  gp.get_data_ptr(data_array)
gp.set_byte_enable_ptr(be_array);
gp.set_response_status(UVM_TLM_OK_RESPONSE); // GENERIC_ERROR, ADDRESS_ERROR, COMMAND_ERROR
gp.is_response_ok();

// Sockets (connect initiator to target)
uvm_tlm_b_initiator_socket  #(GP) isock;  // blocking transport
uvm_tlm_b_target_socket     #(GP) tsock;
isock.connect(tsock);  // direct or via interconnect

// Transport
task b_transport(uvm_tlm_generic_payload t, uvm_tlm_time delay); // blocking
```

---

## 🎛️ Key UVM Macros Reference

```systemverilog
// Registration (go in class body)
`uvm_object_utils(ClassName)
`uvm_component_utils(ClassName)
`uvm_object_param_utils(ParamClass#(T))
`uvm_component_param_utils(ParamClass#(T))

// Field automation (in `uvm_object_utils_begin/end block)
`uvm_field_int(field_name, UVM_ALL_ON)
`uvm_field_string(field_name, UVM_ALL_ON)
`uvm_field_object(field_name, UVM_ALL_ON)
`uvm_field_enum(enum_type, field_name, UVM_ALL_ON)
`uvm_field_array_int(field_name, UVM_ALL_ON)
`uvm_field_da_int(field_name, UVM_ALL_ON)
// Flags: UVM_ALL_ON, UVM_COPY, UVM_COMPARE, UVM_PRINT, UVM_NOPRINT, etc.

// Messaging
`uvm_info("MYTAG", $sformatf("value=%0d", x), UVM_LOW)
`uvm_warning("MYTAG", "warning text")
`uvm_error("MYTAG", "error text")
`uvm_fatal("MYTAG", "fatal text")

// Sequence execution (inside body())
`uvm_create(item)                      // create item
`uvm_do(item)                          // create+randomize+send
`uvm_do_with(item, { item.x == 5; })  // with constraints
`uvm_send(item)                        // send without randomize

// Analysis imp for multiple ports in same component
`uvm_analysis_imp_decl(_SUFFIX)
```

---

## 🔄 UVM Phase Execution Order

```
                    ┌─ build_phase (top-down, function)
                    ├─ connect_phase (bottom-up, function)
                    ├─ end_of_elaboration_phase (bottom-up, function)
                    ├─ start_of_simulation_phase (bottom-up, function)
                    │
                    │   [run_phase runs in PARALLEL with all runtime sub-phases]
                    │
 [run_phase] ───────┤─ pre_reset_phase (task, needs objection)
                    ├─ reset_phase (task, needs objection)
                    ├─ post_reset_phase (task, needs objection)
                    ├─ pre_configure_phase (task, needs objection)
                    ├─ configure_phase (task, needs objection)
                    ├─ post_configure_phase (task, needs objection)
                    ├─ pre_main_phase (task, needs objection)
                    ├─ main_phase (task, needs objection)
                    ├─ post_main_phase (task, needs objection)
                    ├─ pre_shutdown_phase (task, needs objection)
                    ├─ shutdown_phase (task, needs objection)
                    └─ post_shutdown_phase (task, needs objection)
                    │
                    ├─ extract_phase (bottom-up, function)
                    ├─ check_phase (bottom-up, function)
                    ├─ report_phase (bottom-up, function)
                    └─ final_phase (top-down, function)
```

**Key:** All task phases end when all objections are dropped.  
run_phase ends when all other runtime sub-phases end.

---

## 🆚 UVM vs EVM — Why EVM Is Simpler

| UVM Pain Point | EVM Solution |
|----------------|--------------|
| Must call `phase.raise_objection(this)` in every component that has activity | `evm_root::get().raise_objection()` once in test, or use `evm_qc` for automatic |
| Virtual interfaces need `uvm_config_db::set/get` through hierarchy | `agent.set_vif(vif)` directly in tb_top |
| Every object needs `\`uvm_object_utils` + `\`uvm_field_*` macros for factory/copy/compare | Direct `new()`, override `do_copy/do_compare` only if needed |
| Factory overrides require proxy classes and type registration | Simple `evm_test_registry` with `EVM_REGISTER_TEST` macro |
| `uvm_config_db` wildcard scoping causes subtle bugs | No config_db — explicit wiring |
| Multiple analysis imps need `\`uvm_analysis_imp_decl` macro | Multiple `evm_analysis_imp#(T)` ports named separately |
| Callbacks require 3 classes (`uvm_callback`, `\`uvm_register_cb`, override) | Not needed for embedded verification |
| TLM2 sockets for high-speed interconnects | Not implemented — embedded uses simple AXI/APB |
| `uvm_reg_adapter` requires bus-specific implementation | `evm_reg_predictor` is directly parameterized by TXN type |
| Compile time: minutes (large simulator overhead) | Compile time: seconds |
