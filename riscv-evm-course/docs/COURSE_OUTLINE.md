# RISC-V EVM Course - Detailed Outline

## Course Information

**Duration:** 12-15 weeks  
**Level:** Intermediate to Advanced  
**Prerequisites:** Basic digital design, SystemVerilog basics  
**Target Audience:** Hardware engineers, verification engineers, students

---

## Week-by-Week Breakdown

### Week 1: Introduction & Setup
**Topics:**
- Course overview and objectives
- RISC-V ISA introduction (RV32I)
- SystemVerilog refresher
- EVM framework introduction
- Development environment setup

**Lab 1: Environment Setup**
- Install simulator
- Clone repositories
- Run first EVM example
- Understand basic EVM components

**Deliverables:**
- Working development environment
- Successfully run minimal_test example

---

### Week 2: ALU Design
**Topics:**
- Arithmetic and logic operations
- RTL coding best practices
- Basic testbench structure
- EVM test basics

**Lab 2: ALU Implementation**
- Design 32-bit ALU
- Support ADD, SUB, AND, OR, XOR, SLT, SLL, SRL, SRA
- Write basic EVM test
- Verify all operations

**Deliverables:**
- Working ALU RTL
- Basic testbench with 100% coverage

---

### Week 3: Register File
**Topics:**
- Memory elements in RTL
- Dual-port register file design
- Monitor/Scoreboard pattern
- Transaction-level verification

**Lab 3: Register File + Verification**
- Design 32x32 register file
- Implement x0 hardwired to zero
- Write monitor to capture transactions
- Implement scoreboard for checking

**Deliverables:**
- Register file RTL
- Monitor + scoreboard testbench
- Coverage of all read/write combinations

---

### Week 4: Instruction Decoder
**Topics:**
- RISC-V instruction formats (R, I, S, B, U, J)
- Decoding logic
- Control signals
- Sequence-based testing

**Lab 4: Decoder + Sequences**
- Decode RV32I instructions
- Generate control signals
- Write sequences to generate instruction patterns
- Verify all instruction types

**Deliverables:**
- Complete instruction decoder
- Sequence library
- 100% instruction format coverage

---

### Week 5: Program Counter & Control Flow
**Topics:**
- PC logic and branching
- Jump instructions
- Hazard introduction
- Virtual sequences

**Lab 5: Control Flow Unit**
- Implement PC with increment/branch/jump
- Branch target calculation
- Virtual sequences for control flow
- Test all branch conditions

**Deliverables:**
- PC and control flow logic
- Virtual sequence testbench
- Branch coverage

---

### Week 6: Memory Interface
**Topics:**
- Load/Store instructions
- AXI-Lite protocol
- Memory agent
- TLM connections

**Lab 6: Memory System**
- Implement load/store unit
- Integrate AXI-Lite agent from EVM vkit
- Memory model
- Test all addressing modes

**Deliverables:**
- Memory interface RTL
- AXI-Lite integration
- Memory transaction verification

---

### Week 7: Single-Cycle Core
**Topics:**
- Integrating all components
- Datapath design
- Pipeline introduction
- Functional coverage

**Lab 7: Single-Cycle Integration**
- Connect ALU, RegFile, Decoder, PC, Memory
- Run actual RISC-V programs
- Implement functional coverage
- Profile performance

**Deliverables:**
- Working single-cycle core
- Can execute simple programs
- Coverage report

---

### Week 8: 2-Stage Pipeline
**Topics:**
- Pipeline basics
- IF/EX stages
- Pipeline registers
- Structural hazards

**Lab 8: Basic Pipeline**
- Split into IF and EX stages
- Add pipeline registers
- Handle structural hazards
- Performance analysis

**Deliverables:**
- 2-stage pipelined core
- Hazard handling
- Performance improvements measured

---

### Week 9: Hazard Detection
**Topics:**
- Data hazards (RAW, WAR, WAW)
- Control hazards
- Forwarding/bypassing
- Stalling
- Assertions

**Lab 9: Hazard Unit + Assertions**
- Implement hazard detection
- Add forwarding paths
- Stall logic
- SVA assertions for hazards

**Deliverables:**
- Complete hazard unit
- Assertion suite
- Hazard coverage

---

### Week 10: Branch Prediction
**Topics:**
- Branch prediction basics
- Static prediction
- Dynamic prediction (optional)
- Constrained random testing

**Lab 10: Branch Predictor + Random Testing**
- Implement branch predictor
- Constrained random instruction generator
- Branch prediction coverage
- Performance analysis

**Deliverables:**
- Branch predictor
- Random test generator
- Prediction accuracy metrics

---

### Week 11: Exception Handling
**Topics:**
- Exception types
- CSR registers
- Trap handling
- Reset scenarios
- Mid-sim reset with EVM

**Lab 11: Exceptions + Reset**
- Implement basic CSRs
- Exception detection and handling
- Use EVM reset infrastructure
- Test exception scenarios

**Deliverables:**
- Exception handling logic
- CSR implementation
- Reset test scenarios

---

### Week 12: Complete 5-Stage Pipeline
**Topics:**
- IF, ID, EX, MEM, WB stages
- Pipeline optimization
- Layered sequences
- Advanced coverage

**Lab 12: Full Pipeline**
- Implement all 5 stages
- Optimize datapath
- Layered sequence architecture
- Advanced functional coverage

**Deliverables:**
- Complete 5-stage core
- Optimized performance
- Full verification suite

---

### Week 13: Cache System (Optional)
**Topics:**
- Cache basics
- Direct-mapped cache
- Cache controller
- Miss handling

**Lab 13: Data Cache**
- Implement simple D-cache
- Cache hit/miss logic
- Performance improvement
- Cache coverage

**Deliverables:**
- Working data cache
- Performance analysis
- Cache coverage report

---

### Week 14: Optimization & Debug
**Topics:**
- Performance optimization
- Power considerations
- Debug methodology
- Industry best practices

**Lab 14: Optimization Project**
- Profile your core
- Identify bottlenecks
- Optimize critical paths
- Measure improvements

**Deliverables:**
- Optimized core
- Performance report
- Lessons learned document

---

### Week 15: Final Project
**Topics:**
- System integration
- Comprehensive testing
- Documentation
- Presentation

**Final Project:**
- Complete processor with verification environment
- Run real programs (sorting, fibonacci, etc.)
- Comprehensive test report
- Professional documentation
- Team presentation

**Deliverables:**
- Complete, documented processor
- Full verification environment
- Test report with coverage metrics
- Presentation slides
- Demo video

---

## Grading Rubric

### Labs (60% total - 4% each lab)
- **Functionality (50%)**: Does it work correctly?
- **Code Quality (20%)**: Clean, documented, follows best practices
- **Verification (20%)**: Good coverage, proper checking
- **Documentation (10%)**: Clear README and comments

### Quizzes (10% total)
- Weekly 10-minute quizzes
- Multiple choice + short answer
- Cover previous week's material

### Midterm (15%)
- Written exam on weeks 1-7
- Design problems
- Coding exercises

### Final Project (15%)
- Comprehensive processor design
- Verification quality
- Documentation quality
- Presentation quality

---

## Resources

### Required Reading:
- "Computer Organization and Design: RISC-V Edition" - Patterson & Hennessy
- RISC-V ISA Specification (free online)
- EVM Documentation (this repository)

### Recommended:
- "SystemVerilog for Verification" - Chris Spear
- "Digital Design and Computer Architecture: RISC-V Edition" - Harris & Harris

### Online Resources:
- RISC-V International website
- Venus RISC-V simulator
- RISC-V GNU toolchain

---

## Lab Policies

### Submission:
- Submit via Git (private branch)
- Due 11:59 PM each Friday
- Grace period: 48 hours (10% penalty per day)

### Academic Integrity:
- Labs must be individual work
- Collaboration on concepts is OK
- Copying code is not OK
- Solutions will be checked for similarity

### Late Policy:
- First late: Warning
- Second late: -10% final grade
- Third late: Must meet with instructor

### Extra Credit:
- Each lab has optional challenges
- Up to +5% per lab
- Can recover from late penalties

---

## Getting Help

### Office Hours:
- Instructor: Monday/Wednesday 2-3 PM
- TAs: Tuesday/Thursday 4-5 PM

### Communication:
- Use GitHub Issues for technical problems
- Use Discussions for general questions
- Email for private matters
- Slack for quick questions

### Tips for Success:
1. Start labs early
2. Ask questions when stuck
3. Review solutions after submission
4. Participate in discussions
5. Build incrementally, test often
