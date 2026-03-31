# RISC-V Processor Design with EVM

**Educational Course: Building a RISC-V Processor from Scratch with Modern Verification**

This repository contains a comprehensive course for learning processor design and verification by building a RISC-V processor from the ground up, verified using the Embedded Verification Methodology (EVM).

## 🎯 Course Overview

Over 12-15 weeks, you'll progressively build a complete RISC-V processor while learning professional verification techniques. Each week builds upon the previous, culminating in a fully functional pipelined processor with a complete verification environment.

### What You'll Build:
- ✅ Complete RV32I RISC-V processor
- ✅ 5-stage pipeline with hazard detection
- ✅ Professional EVM-based testbench
- ✅ Constrained random verification
- ✅ Coverage-driven verification
- ✅ Real-world verification infrastructure

### What You'll Learn:
- SystemVerilog RTL design
- Pipeline microarchitecture
- EVM verification methodology
- Constrained random testing
- Functional coverage
- Assertions and formal properties
- Industry-standard verification flows

## 📚 Course Structure

### Phase 1: Fundamentals (Weeks 1-3)
- **Week 1**: SystemVerilog Basics + EVM Introduction
- **Week 2**: ALU Design + Basic EVM Testing
- **Week 3**: Register File + Monitor/Scoreboard Pattern

### Phase 2: Core Components (Weeks 4-7)
- **Week 4**: Instruction Decoder + Sequence-Based Testing
- **Week 5**: Program Counter + Control Flow
- **Week 6**: Memory Interface + AXI-Lite Integration
- **Week 7**: Pipeline Basics + Coverage

### Phase 3: Integration (Weeks 8-11)
- **Week 8**: 2-Stage Pipeline Integration
- **Week 9**: Hazard Detection + Assertions
- **Week 10**: Branch Prediction + Random Testing
- **Week 11**: Exception Handling + Reset Scenarios

### Phase 4: Advanced (Weeks 12-15)
- **Week 12**: Data Cache + Layered Sequences
- **Week 13**: Complete 5-Stage Pipeline
- **Week 14**: Performance Optimization
- **Week 15**: Final Project + Presentation

## 🚀 Getting Started

### Prerequisites
- SystemVerilog simulator (VCS, Questa, or Xcelium recommended)
- Git
- Basic knowledge of digital design
- Familiarity with command-line tools

### Setup

1. **Clone this repository:**
   ```bash
   git clone <your-repo-url>
   cd riscv-evm-course
   ```

2. **Initialize EVM submodule:**
   ```bash
   git submodule init
   git submodule update
   ```

3. **Verify setup:**
   ```bash
   cd evm-sv/vkit/src
   ./compile_check.sh  # Checks if EVM compiles with your simulator
   ```

### Directory Structure

```
riscv-evm-course/
├── README.md              # This file
├── LICENSE                # Course license
├── .gitignore            
│
├── evm-sv/               # EVM framework (git submodule)
│
├── lectures/             # Weekly lecture materials
│   ├── week01_intro/
│   ├── week02_alu/
│   └── ...
│
├── labs/                 # Progressive labs with solutions
│   ├── lab01_alu/
│   │   ├── README.md     # Lab instructions
│   │   ├── rtl/          # Starter RTL code
│   │   ├── tb/           # Starter testbench
│   │   └── solution/     # Complete solution
│   └── ...
│
├── reference/            # Reference implementations
│   ├── minimal_core/     # Minimal working core
│   └── complete_core/    # Full-featured core
│
├── common/               # Shared utilities
│   ├── tb_utils/        # Common testbench components
│   └── scripts/         # Build and simulation scripts
│
└── docs/
    ├── COURSE_OUTLINE.md # Detailed course outline
    ├── SETUP_GUIDE.md    # Detailed setup instructions
    ├── RISC-V_PRIMER.md  # RISC-V ISA primer
    └── EVM_GUIDE.md      # EVM quick reference
```

## 📖 Course Materials

### Each Lab Includes:
- **README.md** - Lab objectives, background, and instructions
- **Starter Code** - RTL and testbench templates to fill in
- **Reference Solution** - Complete, working implementation
- **Grading Rubric** - Clear evaluation criteria
- **Extra Credit** - Advanced challenges

### Learning Path:
1. Read the lecture slides
2. Watch the video tutorial (if available)
3. Complete the lab exercises
4. Compare with reference solution
5. Complete extra credit challenges

## 🎓 Learning Objectives

By the end of this course, you will be able to:

✅ Design a complete RISC-V processor from scratch  
✅ Implement pipelining with hazard detection  
✅ Write professional verification environments  
✅ Use constrained random testing effectively  
✅ Apply coverage-driven verification  
✅ Debug complex hardware issues  
✅ Follow industry-standard design flows  

## 🛠️ Tools and Technologies

- **RTL Design**: SystemVerilog
- **Verification**: EVM (Embedded Verification Methodology)
- **Simulators**: VCS, Questa, Xcelium, or Vivado
- **Waveform Viewing**: Verdi, DVE, or Vivado
- **Version Control**: Git
- **Documentation**: Markdown

## 📝 Assessment

- **Labs (60%)**: 12-15 weekly labs
- **Quizzes (10%)**: Short weekly quizzes
- **Midterm (15%)**: Written + coding exam
- **Final Project (15%)**: Complete processor design

## 🤝 Contributing

This is an educational repository. Students should:
- Complete labs individually
- Ask questions via Issues
- Contribute improvements via Pull Requests (after completing course)

Instructors can:
- Fork for their own courses
- Submit improvements
- Share additional materials

## 📄 License

This course material is provided for educational purposes.
- **Course Materials**: [Your License]
- **EVM Framework**: MIT License (see evm-sv/LICENSE)
- **RISC-V ISA**: Open Standard (RISC-V International)

## 🌟 Acknowledgments

- **RISC-V International** - For the open RISC-V ISA specification
- **EVM Contributors** - For the verification framework
- **Students** - For feedback and improvements

## 📧 Contact

- **Instructor**: [Your Name]
- **Course Website**: [URL]
- **Issues**: Use GitHub Issues for questions
- **Discussions**: Use GitHub Discussions for general topics

---

**Ready to build a processor? Start with Week 1!**

📂 [Go to lectures/week01_intro →](lectures/week01_intro/README.md)
