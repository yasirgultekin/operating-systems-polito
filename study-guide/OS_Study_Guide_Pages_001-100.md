# Operating Systems - Exam-Oriented Study Guide
## Pages 1-100: Introduction, OS Concepts, and Classification

**Exam Priority Legend:**
- ⭐⭐⭐⭐⭐ CRITICAL (frequently tested, high points)
- ⭐⭐⭐⭐ HIGH PRIORITY
- ⭐⭐⭐ MEDIUM PRIORITY  
- ⭐⭐ LOWER PRIORITY
- ⭐ MINIMAL (rarely tested)

---

## SECTION 1: Course Introduction and Logistics ⭐
**Slides 1-17 | Exam Relevance: Minimal**

### What You Need to Know (Brief)

**Course Structure:**
- 6 credits, 60 hours
- Topics covered: Processes, Threads, Synchronization, Deadlock, Linux environment
- Laboratory: Important practical component (bash scripting)

**Exam Format (IMPORTANT):**
- Platform: "Exam" platform in classroom
- Duration: 120 minutes
- Questions: 6-18 open or closed questions
- Scoring: 36/30 points (≥32 or 33 → 30 with honors)
- **Materials**: Books/notes NOT allowed
- **Cheat sheets provided**: Shell commands, BASH, threads
- **Penalties exist for wrong answers** (critical to remember!)

**What to Memorize:**
- Exam is closed-book but provides reference cheat sheets
- Wrong answers carry penalties → be strategic, don't guess

**Exam Impact:** While logistics aren't directly tested, understanding the exam format (especially penalties) is crucial for strategy.

---

## SECTION 2: Computer System Components ⭐⭐
**Slides 2-6 (Introduction to OS chapter) | Exam Relevance: Low (conceptual foundation)**

### Core Concepts

**Computer System Layers (Top to Bottom):**
1. **Users** - People, machines, other computers
2. **Applications** - User programs (compilers, databases, games, etc.)
3. **System calls** - Interface between applications and OS
4. **Operating System** - Controls and coordinates hardware use
   - Shell (command interpreter)
   - Kernel (core OS)
5. **Hardware** - CPU, memory, I/O devices

### Operating System Definition

**What is an OS?**
- Software interface between user/application and hardware
- **Goals:**
  - Execute commands and programs
  - Make system user-friendly
  - Use and share hardware efficiently

**Two Perspectives:**

1. **Top-Down View (Virtual Machine):**
   - Abstracts and hides hardware complexity
   - Provides simplified resource management
   - Interface between users and hardware

2. **Bottom-Up View (Resource Manager):**
   - Controls devices and their operations
   - Controls execution of user programs
   - Set of modules providing services

### What to Understand vs. Memorize

**UNDERSTAND:**
- OS sits between applications and hardware
- Acts as both abstraction layer and resource manager

**MEMORIZE:**
- Layer order: Users → Applications → System Calls → OS (Shell/Kernel) → Hardware
- Two views: Virtual Machine (top-down) vs Resource Manager (bottom-up)

### Exam Patterns
- **True/False**: "OS controls execution of user programs" (TRUE)
- **True/False**: "OS provides interface between user and hardware" (TRUE)
- Rarely directly tested but provides foundation for other concepts

---

## SECTION 3: OS Modules and Services ⭐⭐
**Slides 7-15 | Exam Relevance: Low-Medium (tested as true/false)**

### Core OS Modules

The OS provides multiple service modules (analyzed in this course):

**1. Command Interpreter (Shell)**
- User-OS communication interface (textual or graphical)
- User performs tasks through command interpreter
- Allows user to manage processes, memory, protection, network

**2. Process Management** ⭐⭐⭐⭐⭐
- **Process** = program in execution (active unit)
- **Program** = executable on disk (passive unit)
- Requires resources: CPU, memory, devices
- OS supports:
  - Creating, suspending, deleting processes
  - Communication and synchronization among processes

**3. Main Memory Management**
- Program data/instructions must be in main memory for execution
- Memory logically organized as vector of words
- OS responsibilities:
  - Track used/free memory regions
  - Decide which processes to allocate/deallocate
  - Optimize CPU memory access

**4. Secondary Memory Management**
- Main memory is volatile and small → need permanent storage
- OS responsibilities:
  - Organize information in available space
  - Allocate/deallocate space
  - Manage free space
  - Optimize read/write scheduling

**5. I/O Device Management**
- Users cannot manage I/O devices directly (complexity, sharing)
- OS responsibilities:
  - Hide device details (uniform interface)
  - Provide read/write/control operations

**6. File System Management** ⭐⭐⭐⭐⭐
- Data organized in file systems (directories and files)
- OS responsibilities:
  - Create, read, write, remove files/directories
  - Establish access protection mechanisms
  - Optimize read/write operations

**7. Protection Mechanisms**
- Control access to system resources
- OS responsibilities:
  - Define access rights for users and resources
  - Distinguish authorized/unauthorized use
  - Track resource usage

**8. Network and Distributed Systems**
- Network = processors without shared memory/clock
- OS responsibilities:
  - Grant resource access
  - Increase performance and reliability

### What to Memorize

**Module List** (for multiple choice questions):
1. Command interpreter
2. Process management ⭐⭐⭐⭐⭐
3. Main memory management
4. Secondary memory management
5. I/O device management
6. File system management ⭐⭐⭐⭐⭐
7. Protection mechanisms
8. Network management

### Exam Patterns

**Typical Question Format:**
"Which of the following are OS responsibilities?" (Select multiple)
- Creating and deleting processes ✓
- Establishing process synchronization ✓
- Managing free memory space ✓
- Writing application code ✗

**Common Mistakes:**
- Confusing what OS does vs. what applications do
- OS manages resources, doesn't write application logic

---

## SECTION 4: Terminology and Basic Concepts ⭐⭐⭐
**Slides 16-49 | Exam Relevance: Medium (definitions tested)**

### 4.1 Kernel Concepts ⭐⭐

**Kernel:**
- Core of the OS
- Manages all system resources (memory, processors)
- Program always in execution
- All other programs are system programs or applications

**Types of Kernels:**

**1. Monolithic Kernel** (Most Common)
- Services through system calls from tightly integrated modules
- **Advantages:**
  - Extremely efficient due to tight integration
- **Drawbacks:**
  - Module problem can block entire system
  - Adding hardware requires kernel recompilation (modern: runtime loading)
- **Examples:** UNIX, Linux, OpenVMS, XTS-400

**2. Microkernel**
- Very small, simple modules on hardware
- Minimal services implementation
- Separates core services from operational structures
- **Advantages:**
  - Very stable
- **Drawbacks:**
  - Slow (many context switches)
- **Examples:** Mach, L4, AmigaOS, Minix

**3. Hybrid Kernel**
- Intermediate approach
- Microkernel + additional "non-essential" kernel-level code
- Compromise adopted by many developers
- **Advantages:**
  - Integrates benefits of both monolithic and microkernel
- **Drawbacks:**
  - Slightly worse performance than monolithic
- **Examples:** Windows NT, Netware, macOS XNU, Dragonfly BSD

**4. Exokernel**
- "Vertical operating systems"
- Radically different: more direct hardware access
- Separates "protection from management"
- Extremely small and compact
- **Examples:** Nemesis, ExOS

### 4.2 Bootstrap ⭐

**Bootstrap (Booting):**
- Initialization program
- Executes at power-on
- Performs hardware check and initialization
- Loads kernel into main memory
- Usually stored in ROM/EEPROM (firmware)

### 4.3 Kernel Protection ⭐⭐⭐

**Mode Bit:**
- Hardware feature indicating current mode
- **Kernel mode (0):** Can execute all instructions
- **User mode (1):** Restricted instructions
- When interrupt/exception/fault occurs → switches to kernel mode

**Privileged Instructions** (only in kernel mode):
- All I/O instructions
- Changing system register contents
- Loading memory protection registers
- Loading timer

**Protection Mechanisms:**
- **Dual mode** ensures user program can't gain kernel control
- **Memory protection** prevents user writing to kernel memory
- **Timer** used for time-sharing (privileged instruction)

**Key Question:** How does user program perform I/O if I/O instructions are privileged?
**Answer:** Through **system calls**!

### 4.4 System Calls ⭐⭐⭐⭐

**Definition:**
- Interface with OS services
- Entry point of the OS
- Often implemented in assembler
- Accessible via high-level APIs:
  - **Win32/64 API** (Windows)
  - **POSIX API** (UNIX, Linux, macOS)
  - **Java API** (JVM)

**How Many System Calls?**
- UNIX 4.4 BSD: ~110
- Linux: 240-260
- UNIX FreeBSD: ~320

**System Call vs. Function:**
- Both provide services to users
- **Functions:**
  - Written in high-level language (e.g., C)
  - Can be substituted/modified
  - Provide elaborate functionalities
- **System Calls:**
  - Kept stable over years
  - Provide basic functionalities
  - Simple prototypes

**Example - File Reading:**
```c
// UNIX/POSIX
int read(int fd, void *buffer, size_t nbytes);

// Windows
BOOL ReadFile(
    HANDLE fileHandle,
    LPVOID dataBuffer,
    DWORD numberOfBytesToRead,
    LPDWORD numberOfBytesRead,
    LPOVERLAPPED overlappedDataStructure
);
```

**How System Calls Work:**
1. System call causes exception (software interrupt/trap)
2. CPU switches to kernel mode (mode bit = 0)
3. Exception activates corresponding service routine
4. OS verifies parameters are correct and legal
5. Executes request
6. Returns control to instruction following system call

**Common Linux System Calls** (MEMORIZE):
- **Process management:** fork, wait, exec, exit, kill
- **File management:** open, close, read, write, lseek, stat
- **Directory management:** mkdir, rmdir, link, unlink, mount, umount, chdir, chmod

### 4.5 Login and Shell ⭐⭐

**Login:**
- Requires username and password
- Passwords traditionally stored in /etc/passwd

**Shell:**
- Command line interpreter (typical UNIX interface)
- **Not directly part of OS** (important distinction!)
- Set of user-space commands included in OS distribution
- Reads and executes user commands
- Commands from terminal or script file
- **Types:** Bourne shell (sh), Bash (bash), tcsh, ksh

### 4.6 File System Terminology ⭐⭐⭐

**Filename:**
- Few composition rules
- Length limit: typically 255 bytes
- **In UNIX, cannot use:**
  - Slash "/" (directory separator)
  - Null character

**Pathname:**
- Sequence of names separated by slashes '/'
- Examples: /usr/bin, /home/scanzio
- **Special directories:**
  - `.` = current directory
  - `..` = parent directory
- **Types:**
  - **Absolute path:** From root directory (starts with /)
  - **Relative path:** From current working directory

**Home Directory:**
- Accessed after login
- Contains user's files and directories
- Identified by tilde `~` in UNIX-like systems
- User foo's home: /home/foo = ~ for that user

**Root Directory:**
- Main directory
- Root of directory tree
- Origin point for absolute pathnames
- Denoted by `/`

**Working Directory:**
- Origin for relative pathnames
- Initially equals home directory (~)
- Can be changed by navigating file system
- **Each process has its own working directory** (important!)
- Implicit reference: `directory/file1` = `./directory/file1`

### 4.7 Program vs. Process ⭐⭐⭐⭐⭐

**Program:**
- Executable file residing on disk
- **Passive entity**
- Specifies set of operations to execute defined task

**Sequential Program:**
- Operations performed in sequence
- New instruction starts at end of previous one
- Fetch - Decode - Execute cycle

**Concurrent/Parallel Program:**
- Several statements executed in parallel
- Operation can be performed without waiting for previous one

**Atomicity Concept:** ⭐⭐⭐
- Operation is **atomic** if it cannot be interrupted
- Example: `x++;` is NOT atomic!
  ```assembly
  // x++ becomes:
  add BYTE PTR [0x20], 1
  // Which actually involves:
  1. Copy value from memory to CPU register
  2. Increment register
  3. Store register back to memory
  ```
- If another processor executes same operation concurrently, one increment can be lost
- **Why not always guarantee atomicity?**
  - Expensive to guarantee
  - Slows down entire computer

**Process:** ⭐⭐⭐⭐⭐
- Running program (includes PC, registers, variables, etc.)
- **Active entity**
- Each process has unique positive integer identifier (PID)

**Process Tree:**
- Processes can create child processes
- Forms hierarchical tree structure
- Example:
  ```
  P1
  ├── P2
  │   ├── P5
  │   └── P6
  ├── P3
  └── P4
  ```
  P1 creates P2, P3, P4
  P2 creates P5, P6

### 4.8 Threads ⭐⭐⭐⭐

**Thread (Light Process):**
- Process uses set of resources
- Process can have one or more control streams (threads)
- Each thread:
  - Belongs to a process
  - Shares process resources
  - Has its own identifier and "life"
  - Is scheduled separately

### 4.9 Pipe ⭐⭐⭐⭐

**Pipe:**
- Allows communication data flow between two processes
- Typically **half-duplex** (monodirectional)
  - Communication in ONE direction: P1→P2 or P2→P1
  - NOT both simultaneously
- **Full-duplex:** Bidirectional communication

### 4.10 Deadlock, Livelock, Starvation ⭐⭐⭐⭐

**Deadlock:** ⭐⭐⭐⭐
- Entities (processes) sharing resources wait indefinitely for event caused by other entities
- Results in one or more entities blocked **forever**
- Nobody makes progress
- Classic example: Two cars at narrow bridge, both want to cross
  ```
  P1 holds R1, waits for R2
  P2 holds R2, waits for R1
  → Both blocked forever (deadlock)
  ```

**Livelock (Active Deadlock):**
- Similar to deadlock but entities not actually blocked
- Entities too busy responding to each other to make progress
- Example: Two people in corridor repeatedly moving to same side
- Both doing work (polling) but no progress made

**Starvation:**
- Access to needed resource repeatedly refused to entity
- **Starvation ≠ Deadlock**
- While one entity starves, others can progress
- **Deadlock → Starvation** (all entities starve)
- **Starvation ≠ Deadlock** (system progresses, one doesn't)

### What to Memorize

**System Call Examples:**
- Process: fork, wait, exec, exit, kill
- File: open, close, read, write, lseek, stat
- Directory: mkdir, rmdir, link, unlink, mount, umount, chdir, chmod

**Special Directory Symbols:**
- `.` = current directory
- `..` = parent directory
- `~` = home directory
- `/` = root directory

**Key Distinctions:**
- Program (passive) vs. Process (active)
- Deadlock (nobody progresses) vs. Starvation (one doesn't, others do)
- Half-duplex (one direction) vs. Full-duplex (both directions)
- Kernel mode (0) vs. User mode (1)

### Exam Patterns

**True/False Questions:**
1. "All I/O instructions are privileged" → TRUE
2. "System calls allow user programs to request OS services" → TRUE
3. "Shell is part of the OS kernel" → FALSE (it's user-space!)
4. "Each process has its own working directory" → TRUE
5. "x++ is an atomic operation" → FALSE
6. "Deadlock implies starvation" → TRUE
7. "Starvation implies deadlock" → FALSE
8. "Pipes are full-duplex by default" → FALSE (half-duplex)

**Common Mistakes:**
- Thinking shell is part of kernel (it's not!)
- Confusing deadlock with starvation
- Assuming simple operations like x++ are atomic
- Forgetting that each process has its own working directory

---

## SECTION 5: Operating Systems Classification ⭐
**Slides 2-23 (OS Classification chapter) | Exam Relevance: Minimal**

### OS Market Share (Brief Overview)

**All Devices (September 2021):**
- Android: 42.61%
- Windows: 30.66%
- iOS/macOS: 16.55%
- OS X: 6.50%
- Linux: 0.98%

**Desktop/Laptop:**
- Windows 10: 58.93%
- Windows 7: 23.35%
- macOS: ~8%
- Linux (Ubuntu): 2.57%

**Servers:**
- Windows: 85.23%
- Linux: 13.60%
- Unix: 5.60%

### Windows ⭐

**Microsoft:**
- Founded 1975 (Bill Gates, Paul Allen)
- MS-DOS introduced 1981
- Windows introduced 1985
- Graphical interface based on windows
- Intel processor oriented
- Controls 80%-90% desktop market

**Evolution:**
- 16-bit: Windows 1.0 (1985) - Windows 3.1 (1992)
- 16/32-bit: Windows 9x (1993-2000)
- 32/64-bit: Windows NT onwards

**Server Versions:** NT, 2000, Server 2003/2008/2012
**Desktop Versions:** XP (2001), Vista (2007), 7 (2009), 8/8.1 (2012), 10 (2015), 11 (2021)
**Embedded:** CE, Embedded, Phone, Mobile, RT

### macOS ⭐

**Apple:**
- 1984-2001: Classic Mac OS (graphical only)
- Reached structural limits (no preemptive multitasking, no protected memory)
- 2001: Mac OS X introduced
  - Based on UNIX BSD architecture
  - 100% POSIX compliant
  - Initially backward-compatible

**Architecture:**
- Initially micro-kernel structure (performance issues)
- Recent: Hybrid three-layer structure
- Includes UNIX utilities, shells, Java VM, scripting languages

**Characteristics:**
- Proprietary, not open source
- Can execute many GNU Linux programs
- Micro-kernel: easily extendable, high reliability
- High security
- Limited diffusion, expensive

### UNIX/Linux ⭐

**UNIX History:**
- Designed 1970 for PDP-11 minicomputer
- High portability but many versions in 1980s
- Standardization required by organizations

**Standards:**
- **ISO C:** ANSI C (1989), C90 (1990), C99 (1999), C11 (2011)
- **POSIX:** Portable Operating System Interface
  - Defines services for POSIX compliance
  - Includes ISO C standard
- **SUS (Single UNIX Specification):**
  - POSIX superset
  - Defines standards for "UNIX" trademark

**UNIX Implementations:**
- AIX (IBM), Digital Unix (Compaq), HP-UX (HP)
- IRIX (SGI), macOS (Apple)
- FreeBSD, Solaris (Sun)
- Android (Google)
- Linux distributions

**Linux:**
- Created 1991 by Linus Torvalds (Helsinki)
- Developed from Minix (Tanenbaum, 1987)
- Open source (GNU Public License)
- "Linux" = kernel name
- Many distributions share common kernel

**Major Distributions:**
- **Mint:** User friendly, various versions
- **Ubuntu:** Based on Debian, easy and complete (2002)
- **Debian:** Only open software, very old (1993)
- **Fedora:** Sponsored by Red Hat (1995)
- **CentOS:** For servers (2003)
- **Arch:** For "geeks"

**Linux Characteristics:**
- Developed globally
- 95% Hollywood effects on Linux
- Debian 4.0 (2007): 283M lines of code
  - Would require 73,000 man-years, $8.16B without open source
- Many consider it most advanced OS

**Android Note:**
- Uses modified Linux kernel
- NOT GNU/Linux (uses Bionic C library instead of GNU C)
- 75% of smartphones Linux-based (2018)

### Comparison ⭐

**Windows:**
- Price: ≥$100
- Ease: Easy
- Reliability: Average
- Security: Average
- Software: High quantity, ≥$200
- Hardware support: Very large
- Open source: No
- Support: Proprietary

**macOS:**
- Price: ≥$100
- Ease: Easy
- Reliability: Good
- Security: Good
- Software: High quantity, ≥$200
- Hardware support: Good
- Open source: No
- Support: Proprietary

**Linux:**
- Price: Free
- Ease: Average
- Reliability: Excellent
- Security: Excellent
- Software: Good quantity, Free
- Hardware support: Average
- Open source: Yes
- Support: Online

### Why Focus on UNIX/Linux? ⭐

**General Reasons:**
- Course needs specific reference (not too abstract)
- Limited time (6 credits)
- Strong link with C programming

**Historical Reasons:**
- Learned programs useful in everyday life
- Derived OSs used in embedded devices (e.g., Raspberry Pi)

**Limits of Alternatives:**
- Windows: Already known, not open, subject of master's courses
- macOS: Expensive, not available in labs, similar to UNIX/Linux anyway

### What to Know (Exam Relevance: Low)

**Key Takeaways:**
- Three major OS families: Windows, macOS, Linux
- Linux is open source, others proprietary
- macOS is POSIX-compliant (UNIX-based)
- Course focuses on Linux for practical reasons
- Android uses Linux kernel

**Exam Impact:**
- Rarely directly tested
- Background knowledge only
- Focus on UNIX/Linux practical skills

---

## SECTION 6: Linux Installation Options ⭐
**Slides 2-15 (UNIX & Linux commands chapter beginning) | Exam Relevance: Minimal**

### Installation Methods (Brief Overview)

**1. Linux LIVE:**
- Execute OS without installation
- From CD or USB key (.iso files)
- Cannot save configuration
- Reduced features

**2. Multi-boot Partition:**
- Multiple OSs on different partitions
- Boot loader (GRUB/GRUB2) chooses OS at startup
- Complex and potentially dangerous

**3. Virtual Machine (VM):** ⭐
- Hardware emulation applications
- **VirtualBox** (Oracle, most common)
  - For AMD64/Intel64
  - Available for Windows, Linux, macOS, Solaris
- **Others:** VMware, QEMU, Microsoft Virtual Machine
- Creates illusion of multiple PCs
- Requires virtualization enabled in BIOS
- Recommendation: Install "Guest Additions" after Linux installation

**4. Windows Subsystem for Linux (WSL):**
- Originally "Bash on Ubuntu on Windows"
- NOT virtualization
- Microsoft implemented Linux kernel API subsystem
- More efficient, fewer resources than VM
- Requirements: Windows 10 (from 1607), 64-bit
- Install software with: `sudo apt install <packageName>`

**5. Docker:**
- Open-source container technology
- Containers share OS kernel
- Embed dependencies and configurations
- **Advantages over VM:**
  - Fewer resource requirements
  - Faster
  - Containers include app + dependencies (not full OS)
- **Benefits:** Isolation, portability, agility, scalability

### Ubuntu Distributions ⭐

**Release Schedule:**
- New version every 6 months
- LTS (Long Term Support) every 24 months (supported 5 years)
- **Recent LTS versions:**
  - 14.04 LTS Trusty Tahr (2014)
  - 16.04 LTS Xenial Xerus (2016)
  - 18.04 LTS Bionic Beaver (2018)
  - 20.04 LTS Focal Fossa (2020)
  - 22.04 LTS Jammy Jellyfish (2022)

**Name Origin:**
- "Ubuntu" in Nguni Bantu = "humanity" or "I am because we are"
- GNU GPL (General Public Library) license

### What to Know

**Exam Impact:** Virtually zero
- Installation methods not tested
- Background information only
- Focus on using Linux, not installing it

**Practical Knowledge:**
- Lab exercises use Linux environment
- Can use VirtualBox if needed
- WSL convenient for Windows users

---

## SUMMARY: Key Takeaways from Pages 1-100

### CRITICAL CONCEPTS (Must Master):

1. **System Architecture:**
   - Layer order: Users → Apps → System Calls → OS → Hardware
   - OS as both Virtual Machine and Resource Manager

2. **System Calls:** ⭐⭐⭐⭐
   - Interface between user programs and OS
   - Enable privileged operations from user mode
   - Common calls: fork, wait, exec, open, close, read, write

3. **Process vs Program:** ⭐⭐⭐⭐⭐
   - Program = passive (on disk)
   - Process = active (in execution)
   - Process has unique PID

4. **Kernel Protection:**
   - Mode bit: kernel (0) vs user (1)
   - Privileged instructions only in kernel mode
   - I/O operations require system calls

5. **File System Terminology:** ⭐⭐⭐
   - Working directory vs home directory
   - Absolute vs relative paths
   - Special: `.` (current), `..` (parent), `~` (home), `/` (root)

6. **Deadlock vs Starvation:** ⭐⭐⭐⭐
   - Deadlock: Nobody progresses (circular wait)
   - Starvation: One doesn't progress, others do
   - Deadlock implies starvation, but not vice versa

### MEMORIZATION CHECKLIST:

**System Calls:**
- [ ] fork, wait, exec, exit, kill (process)
- [ ] open, close, read, write, lseek, stat (file)
- [ ] mkdir, rmdir, link, unlink, chdir, chmod (directory)

**Path Symbols:**
- [ ] `.` = current directory
- [ ] `..` = parent directory
- [ ] `~` = home directory
- [ ] `/` = root directory

**Key Distinctions:**
- [ ] Active vs passive (process vs program)
- [ ] Kernel mode (0) vs user mode (1)
- [ ] Deadlock vs starvation
- [ ] Half-duplex vs full-duplex

**Atomicity:**
- [ ] Simple operations like x++ are NOT atomic
- [ ] Can lose updates in concurrent execution

### EXAM STRATEGY FOR THIS MATERIAL:

1. **True/False Questions:**
   - Focus on definitions (program vs process, deadlock vs starvation)
   - Watch for kernel vs user-space distinctions
   - Shell is NOT part of kernel!

2. **Multiple Choice:**
   - OS module responsibilities
   - System call categories
   - Path types and special symbols

3. **Low Priority:**
   - OS market share statistics
   - Specific OS versions
   - Installation methods
   - Kernel type details

### COMMON EXAM TRAPS:

❌ "Shell is part of the OS kernel" → FALSE (user-space!)
❌ "x++ is atomic" → FALSE (three operations!)
❌ "Starvation implies deadlock" → FALSE (opposite only!)
❌ "Pipes are full-duplex by default" → FALSE (half-duplex!)
❌ "User programs can directly execute I/O" → FALSE (need system calls!)

### PRIORITY FOCUS FOR NEXT SECTIONS:

Based on exam analysis, upcoming topics will include:
- **HIGHEST PRIORITY:** Process creation (fork), File operations, Synchronization
- **HIGH PRIORITY:** Thread management, Scheduling, Pipes
- **MEDIUM PRIORITY:** Shell commands, Signals
- **LOWER PRIORITY:** Specific Linux distributions, Tools

---

## STUDY RECOMMENDATION:

**Time allocation for pages 1-100:**
- Quick review: 30 minutes
- Main focus: System calls, process concepts, file paths, deadlock/starvation
- Skip: OS market share, installation details, version numbers

**What's coming next (pages 101+):**
- Linux commands and file operations ⭐⭐⭐⭐⭐
- Process management and fork() ⭐⭐⭐⭐⭐ (MOST CRITICAL!)
- File system implementation ⭐⭐⭐⭐⭐
- These are HEAVILY tested → deserve deep focus!

---

**End of Pages 1-100 Study Guide**
