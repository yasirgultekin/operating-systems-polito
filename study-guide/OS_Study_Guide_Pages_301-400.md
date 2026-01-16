# Operating Systems - Exam-Oriented Study Guide
## Pages 301-400: Process Management with fork() ⭐⭐⭐⭐⭐

**THIS IS THE MOST HEAVILY TESTED SECTION IN THE ENTIRE EXAM!**
**41 Questions, 111 Points - Maximum Priority!**

**Previous Sections:** Pages 1-300 covered commands, file I/O, filters

---

## ⚠️ CRITICAL EXAM WARNING ⚠️

**This section (Process Management with fork) accounts for:**
- **41 exam questions** (most of any topic)
- **111 total points** (highest point value)
- **Appears in EVERY exam**
- **Most complex calculations required**

**Study Time Allocation:**
- **Minimum 40% of total OS exam preparation time should be on this section**
- Master fork() behavior completely
- Practice process tree drawing extensively
- Understand output prediction patterns

---

## SECTION 18: File System Allocation (Brief) ⭐⭐
**Slides 10-43 | Exam Relevance: Low-Medium**

### 18.1 Directory Structures ⭐

**Tree Directories:**
- Hierarchical organization
- Absolute and relative paths
- Working directory concept
- Efficient for searching and grouping

**Acyclic Graph Directories:**
- Allows sharing via links
- Hard links vs soft links distinction
- Cycle prevention important

**Not heavily tested** - Background knowledge only

### 18.2 Allocation Methods ⭐⭐⭐

**Three Main Techniques:**

**1. Contiguous Allocation:**
- File stored in contiguous blocks
- Directory stores: start block (b) and length (n)
- File uses blocks: b, b+1, b+2, ..., b+n-1
- **Advantages:** Simple, fast direct access
- **Drawbacks:** External fragmentation, allocation algorithms needed (first-fit, best-fit, worst-fit)

**2. Linked Allocation:**
- File as linked list of blocks
- Directory contains pointer to first and last block
- Each block points to next
- **Advantages:** No external fragmentation, dynamic growth
- **Drawbacks:** Only efficient for sequential access, space for pointers

**3. FAT (File Allocation Table):**
- Variant of linked allocation
- Pointers in separate table (FAT), not in blocks
- **Advantages:** Easier management
- **Drawbacks:** Two disk accesses per block, FAT loss catastrophic

**4. Indexed Allocation (UNIX/Linux):** ⭐⭐⭐⭐
- Each file has index block (i-node)
- **Combined schema in UNIX:**
  - 15 pointers in i-node
  - **First 12:** Direct pointers to data blocks
  - **Pointer 13:** Single indirect (points to block of pointers)
  - **Pointer 14:** Double indirect (points to block of pointers to blocks of pointers)
  - **Pointer 15:** Triple indirect
  - With 64-bit pointers: files up to 2^60 bytes (exabyte)

**Hard Links:**
- Directory entry pointing to i-node
- Same i-node can have multiple hard links
- Link counter in i-node
- File deleted only when counter reaches 0
- **Cannot:** Link directories, cross filesystems

**Soft Links (Symbolic):**
- Data block contains pathname of target file
- Can cross filesystems
- Can link directories
- Becomes dangling if target deleted

### What to Know (Brief)

- Indexed allocation with i-nodes (UNIX system)
- 12 direct + 3 levels of indirect pointers
- Hard vs soft link differences
- **Exam Impact:** Low-medium, conceptual questions only

---

## SECTION 19: File System Functions ⭐⭐⭐
**Slides 47-60 | Exam Relevance: Medium**

### 19.1 stat() Function ⭐⭐⭐

**Purpose:** Get file/directory information

```c
#include <sys/types.h>
#include <sys/stat.h>

int stat(const char *path, struct stat *sb);
int lstat(const char *path, struct stat *sb);   // For symbolic links
int fstat(int fd, struct stat *sb);             // For open files

// Returns: 0 on success, -1 on error
```

**struct stat Fields:**
```c
struct stat {
    mode_t st_mode;     // File type & mode
    ino_t st_ino;       // i-node number
    dev_t st_dev;       // Device number
    ...
};
```

**Type Check Macros (MEMORIZE):**
```c
S_ISREG(m)      // Regular file
S_ISDIR(m)      // Directory
S_ISCHR(m)      // Character special file
S_ISBLK(m)      // Block special file
S_ISFIFO(m)     // FIFO/pipe
S_ISLNK(m)      // Symbolic link
S_ISSOCK(m)     // Socket
```

**Example:**
```c
struct stat buf;
if (lstat(path, &buf) < 0) {
    fprintf(stderr, "lstat error\n");
    exit(1);
}

if (S_ISDIR(buf.st_mode))
    printf("directory\n");
else if (S_ISREG(buf.st_mode))
    printf("regular file\n");
```

### 19.2 Directory Functions ⭐⭐

**getcwd() and chdir():**
```c
#include <unistd.h>

char *getcwd(char *buf, int size);   // Get current working directory
int chdir(char *path);                // Change directory

// Returns: buf/0 on success, NULL/-1 on error
```

**mkdir() and rmdir():**
```c
#include <sys/stat.h>

int mkdir(const char *path, mode_t mode);
int rmdir(const char *path);          // Only if empty

// Returns: 0 on success, -1 on error
```

### What to Know

- stat() for file type checking
- S_ISDIR, S_ISREG macros
- getcwd/chdir for navigation
- mkdir/rmdir for creation/deletion
- **Exam Impact:** Medium - appears in code reading questions

---

## SECTION 20: Process Concepts ⭐⭐⭐⭐⭐
**Slides 2-11 | Exam Relevance: CRITICAL (Foundation for fork!)**

### 20.1 Definitions ⭐⭐⭐⭐⭐

**MUST MEMORIZE:**

**Algorithm:**
- Logical process solving a problem in finite steps

**Program:**
- Formalization of algorithm in programming language
- **Passive entity** - executable file on disk
- Static, stored permanently

**Process:**
- Abstraction of running program
- **Active entity** - program in execution
- Dynamic, exists during execution
- Sequence of operations on given data

### 20.2 Sequential vs Concurrent ⭐⭐⭐⭐

**Sequential Execution:**
- Actions executed one after another
- New action begins only after previous completes
- Totally ordered
- **Deterministic behavior:**
  - Same input → same output
  - Independent of execution time, speed, or number of processes

**Concurrent Execution:**
- Multiple actions at same time
- No order relation
- **Non-deterministic behavior**
- **Real concurrency:** Multi-processor/multi-core systems
- **Pseudo-concurrency:** Mono-processor with time-sharing

### 20.3 Process Identity ⭐⭐⭐⭐⭐

**Process Identifier (PID):**
- Non-negative integer
- Unique for each process
- UNIX reuses PIDs of terminated processes
- Can be used for unique object names

**System Calls:**
```c
#include <unistd.h>

pid_t getpid();      // Current process ID
pid_t getppid();     // Parent process ID
uid_t getuid();      // User ID
gid_t getgid();      // Group ID
```

**CRITICAL:** No system call to get child PID!
- Parent receives child PID from fork()
- Child must communicate its PID if needed

**Reserved PIDs:**
- **PID 0:** Swapper (scheduler)
  - Memory management and process scheduling
  - Kernel level execution
- **PID 1:** init daemon
  - First user-level process
  - Super-user privileges
  - Becomes parent of orphan processes

### What to Memorize

- [ ] Program = passive, Process = active
- [ ] Sequential = deterministic, Concurrent = non-deterministic
- [ ] getpid() = current PID, getppid() = parent PID
- [ ] PID 0 = swapper, PID 1 = init
- [ ] No system call to get child PID

---

## SECTION 21: fork() System Call ⭐⭐⭐⭐⭐
**Slides 12-42 | Exam Relevance: MAXIMUM CRITICAL!**

**THIS IS THE MOST TESTED MATERIAL IN THE ENTIRE COURSE!**

### 21.1 fork() Basics ⭐⭐⭐⭐⭐

**System Call:**
```c
#include <unistd.h>

pid_t fork(void);

// Returns:
//   - Child PID to parent process
//   - 0 to child process
//   - -1 on error
```

**CRITICAL CONCEPT:**
- **fork() is called ONCE**
- **fork() returns TWICE** (in two different processes!)
- Creates exact copy of calling process
- Child is duplicate of parent

**Return Values (MEMORIZE!):**
```c
pid_t pid = fork();

if (pid < 0) {
    // Error (fork failed)
    // Usually: process limit reached
}
else if (pid == 0) {
    // Child process
    // pid variable contains 0
}
else {  // pid > 0
    // Parent process
    // pid variable contains child's PID
}
```

### 21.2 Process Duplication ⭐⭐⭐⭐⭐

**What is Shared:**
- Code (program instructions)
- File descriptors (including stdin, stdout, stderr)
- User ID, Group ID
- Working directory, root directory
- System resource limits
- Signal table

**What is Different:**
- Return value from fork()
- Process ID (PID)
  - Parent keeps its PID
  - Child gets new PID
- Parent PID (PPID)
  - Child's PPID = parent's PID
- Data, heap, stack spaces
  - Initial values inherited
  - Separate memory spaces
  - Copy-on-write optimization

**CRITICAL for Exam:**
```c
char c = 'X';
if (fork()) {
    // Parent
    c = 'F';
} else {
    // Child
    c = 'C';
}
printf("c=%c\n", c);

// Output:
// c=F  (from parent)
// c=C  (from child)
// Variables are INDEPENDENT after fork!
```

### 21.3 Counting Processes ⭐⭐⭐⭐⭐

**EXAM PATTERN #1: Sequential forks**

**Formula:** n sequential fork() calls → 2^n total processes

**Example 1:**
```c
fork();   // #1
fork();   // #2
fork();   // #3

// How many processes? 2^3 = 8 processes
```

**Process Tree:**
```
P
├── C1
│   ├── C11
│   │   └── C111
│   └── C12
├── C2
│   └── C21
└── C3
```

**Control Flow Graph (CFG):**
```
P → fork1 → fork2 → fork3
    ↓       ↓       ↓
    C1 →   fork2 → fork3
            ↓       ↓
            C2 →   fork3
                    ↓
                    C3 (and so on...)
```

**EXAM PATTERN #2: Conditional forks**

**Example 2:**
```c
pid = fork();    // #1
if (pid != 0)
    fork();      // #2
fork();          // #3

// How many processes?
```

**Step-by-step:**
1. After fork #1: 2 processes (P, C1)
2. fork #2 executed by: Only P (pid != 0)
   - Creates C2
   - Now: P, C1, C2
3. fork #3 executed by: P, C1, C2
   - Each creates child
   - Creates: C3 (P's child), C11 (C1's child), C21 (C2's child)
   - Total: P, C1, C2, C3, C11, C21 = **6 processes**

**Process Tree:**
```
P
├── C1
│   └── C11
├── C2
│   └── C21
└── C3
```

**EXAM PATTERN #3: Loop with forks**

**Example 3:**
```c
for (i=0; i<2; i++) {
    printf("i: %d\n", i);
    if (fork())      // #1
        fork();      // #2
}

// CRITICAL: Use setbuf(stdout, 0) or fflush(stdout)!
```

**Iteration 1 (i=0):**
- P prints "0"
- P forks → creates C1
- P (fork returned >0) executes fork #2 → creates C2
- C1 (fork returned 0) skips fork #2
- After iteration 1: P, C1, C2 (all have i=1 next)

**Iteration 2 (i=1):**
- P prints "1", forks C3, forks C4
- C1 prints "1", forks C11, doesn't fork (returned 0)
- C2 prints "1", forks C21, forks C22
- After iteration 2: 8 processes total

**Output:** 0 1 1 1 (without setbuf/fflush, may be duplicated!)

### 21.4 setbuf() and Output Prediction ⭐⭐⭐⭐⭐

**CRITICAL FOR EXAM!**

**The Problem:**
```c
printf("i: %d\n", i);
fork();
```

Without setbuf(stdout, 0):
- printf() output buffered
- Buffer copied to child at fork()
- Both processes flush buffer → duplicate output!

**The Solution:**
```c
setbuf(stdout, 0);  // Disable buffering
// OR
fflush(stdout);     // Flush before each fork
```

**EXAM QUESTIONS:**
- "Given this code, predict output"
- **Always check if setbuf or fflush is present!**
- Without it: printed values may appear multiple times

### 21.5 Process Tree Drawing ⭐⭐⭐⭐⭐

**Step-by-Step Method:**

**Example:**
```c
pid = fork();    // #1
fork();          // #2
if (pid != 0)
    fork();      // #3
```

**Steps:**
1. **Label each fork** (#1, #2, #3)
2. **Track which processes exist** at each point
3. **Check conditions** (if statements)
4. **Draw tree** showing parent-child relationships

**Process Tree:**
```
P
├── C1 (from #1)
│   ├── C11 (from #2)
│   └── C12 (from #3, if C1 is pid!=0... NO! C1 has pid==0)
├── C2 (from #2)
│   └── C21 (from #3)
└── C3 (from #3)
```

**Wait, let me recalculate:**

After fork #1: P (pid=child's PID), C1 (pid=0)
After fork #2: P, C1, C2 (P's child from #2), C11 (C1's child from #2)
Fork #3 executed by: P and C2 (both have pid != 0)
- Creates C3 (P's child), C21 (C2's child)

Final: **P, C1, C2, C3, C11, C21 = 6 processes**

### 21.6 Complex Fork Examples ⭐⭐⭐⭐⭐

**EXAM PATTERN #4: Nested functions with fork**

```c
int main() {
    int a, b=5, c;
    a = fork(); /* #1 */
    if (a) {
        a = b; 
        c = split(a, b++);
    } else {
        fork(); /* #2 */
        c = a++; 
        b += c;
    }
    if (b > c) {
        fork(); /* #3 */
    }
    printf("%3d", a+b+c);
    return 0;
}

int split(int a, int b) {
    a++;
    a = fork(); /* #4 */
    if (a) {
        a = b;
    } else {
        if (fork()) /* #5 */ {
            a--;
            b += a;
        }
    }
    return a+b;
}
```

**Solution Strategy:**
1. **Trace each process path** through code
2. **Track variable values** for each process
3. **Note which forks each process executes**
4. **Calculate final a+b+c** for each process

**Process Tree:**
```
P
├── C1 (from #1, main else branch)
│   ├── C11 (from #2)
│   ├── C12 (from #3 if b>c)
│   └── ...
├── C2 (from #1→split→#4)
│   └── ...
└── ...
```

**Variable Tracking Table:**
```
Process | a   | b   | c   | a+b+c | Notes
--------|-----|-----|-----|-------|------
P       | 5   | 6   | 10  | 21    | 
C1      | 1   | 5   | 0   | 6     |
C2      | 5   | 6   | 3   | 14    |
...
```

### 21.7 Common Mistakes (EXAM TRAPS!) ⭐⭐⭐⭐⭐

**TRAP #1: Counting fork() calls instead of processes**
```c
fork();
fork();
fork();

// Students think: 3 fork calls = 3 children
// WRONG! 2^3 = 8 total processes (including parent)
```

**TRAP #2: Forgetting child doesn't execute parent code**
```c
for (i=0; i<n; i++) {
    fork();
}

// Students think: n children created
// WRONG! 2^n total processes created!
// Child processes also execute loop!
```

**TRAP #3: Missing setbuf/fflush implications**
```c
printf("Hello\n");
fork();

// If no setbuf(stdout, 0):
// "Hello" may print twice! (buffered, copied to child)
```

**TRAP #4: Confusing return values**
```c
pid = fork();
if (pid > 0) {
    // Students might think: child code
    // WRONG! This is PARENT (pid = child's PID > 0)
}
```

**TRAP #5: Variable independence**
```c
int x = 5;
fork();
x = 10;  // Only in one process!

// Students think: x becomes 10 in both
// WRONG! Separate memory spaces after fork!
```

### 21.8 Generating N Children ⭐⭐⭐⭐⭐

**EXAM PATTERN: "Write code to create exactly N children"**

**WRONG Solution:**
```c
for (i=0; i<n; i++) {
    fork();
}

// Creates 2^n processes, not n children!
```

**CORRECT Solution:**
```c
for (i=0; i<n; i++) {
    if (fork() == 0) {
        // Child code
        printf("Child %d: PID=%d\n", i, getpid());
        break;  // CRITICAL! Child exits loop
    }
}
// Parent continues here
printf("Parent: PID=%d\n", getpid());
```

**Why break is critical:**
- Without break: child continues loop
- Child would fork more children
- Creates exponential explosion!

**Process Tree (with n=3):**
```
P
├── C1 (i=0, breaks)
├── C2 (i=1, breaks)
└── C3 (i=2, breaks)

Total: 1 parent + 3 children = 4 processes ✓
```

### What to Memorize

**fork() Return Values:**
- [ ] Parent: child's PID (positive)
- [ ] Child: 0
- [ ] Error: -1

**Process Count:**
- [ ] n sequential forks → 2^n processes
- [ ] Check conditions carefully
- [ ] Draw process tree step by step

**Shared vs Different:**
- [ ] Shared: code, file descriptors
- [ ] Different: PID, return value, data/stack

**Output Prediction:**
- [ ] setbuf(stdout, 0) disables buffering
- [ ] Without it: duplicate output possible
- [ ] fflush(stdout) before each fork

**Common Patterns:**
- [ ] Creating N children: fork in loop with break
- [ ] Conditional forks: track which processes execute
- [ ] Nested forks: draw CFG and process tree

### Exam Patterns

**MOST COMMON EXAM QUESTIONS:**

**Type 1: Count processes**
"How many processes are created by this code?"
- Draw process tree
- Count carefully
- Don't forget parent!

**Type 2: Predict output**
"What is the output of this program?"
- Check for setbuf/fflush
- Track variable values
- Consider execution order

**Type 3: Draw process tree**
"Draw the process generation tree"
- Label each fork
- Show parent-child relationships
- Include all processes

**Type 4: Write code**
"Write a program to create N children"
- Use fork() in loop
- Child must break
- Parent continues

**Type 5: Variable values**
"What are the final values of variables a, b, c?"
- Track each process separately
- Remember copy-on-write
- Calculate for each process

---

## SECTION 22: Process Termination and wait() ⭐⭐⭐⭐⭐
**Slides 43-49 | Exam Relevance: CRITICAL**

### 22.1 Process Termination ⭐⭐⭐⭐

**Normal Termination (5 methods):**
1. **return from main()**
2. **exit() system call**
3. **_exit() or _Exit()**
   - Similar to exit() but different stdio handling
4. **return from last thread**
5. **pthread_exit() from last thread**

**Abnormal Termination (3 methods):**
1. **abort()** function
   - Generates SIGABRT signal
2. **Uncaught signal**
   - Signal causes termination
3. **Last thread cancelled**

### 22.2 wait() System Call ⭐⭐⭐⭐⭐

**Purpose:** Parent waits for child termination

```c
#include <sys/wait.h>

pid_t wait(int *statLoc);

// Returns:
//   - PID of terminated child on success
//   - -1 on error (no children)
```

**Behavior:**

**Case 1: No children**
- Returns -1 immediately
- Process without children shouldn't call wait

**Case 2: All children running**
- **Blocks** calling process
- Returns when **first** child terminates
- Returns that child's PID

**Case 3: Child already terminated**
- Returns immediately
- Returns terminated child's PID
- Retrieves exit status

**CRITICAL Concept: Zombie Processes**
- When child terminates, parent receives SIGCHLD signal
- If parent doesn't call wait():
  - Child becomes **zombie**
  - Termination status remains pending
  - Resources not fully released
  - Entry remains in process table
- **wait() reaps zombie**, fully releases resources

**Example:**
```c
pid_t pid, child_pid;
int status;

pid = fork();
if (pid == 0) {
    // Child
    printf("Child: PID=%d\n", getpid());
    exit(0);
} else {
    // Parent
    child_pid = wait(&status);
    printf("Parent: Child %d terminated\n", child_pid);
}
```

**Without wait():**
```c
pid = fork();
if (pid == 0) {
    exit(0);  // Child terminates
}
// Parent doesn't wait
// Child becomes zombie until parent terminates!
```

### 22.3 Orphan Processes ⭐⭐⭐⭐

**Orphan Process:**
- Parent terminates before child
- Child's PPID becomes 1 (init process)
- init adopts orphan
- init automatically reaps orphans when they terminate

**Example:**
```c
pid = fork();
if (pid == 0) {
    // Child
    sleep(5);  // Wait 5 seconds
    printf("Child: PPID=%d\n", getppid());  // Will print 1!
} else {
    // Parent
    printf("Parent: Terminating\n");
    exit(0);  // Parent terminates immediately
}
```

**Output:**
```
Parent: Terminating
(5 seconds pass)
Child: PPID=1       ← init is now parent!
```

### 22.4 Parent-Child Synchronization ⭐⭐⭐⭐⭐

**Pattern: Parent waits for all children**

```c
int i, n = 5;
pid_t pid;

// Create n children
for (i=0; i<n; i++) {
    if (fork() == 0) {
        // Child code
        printf("Child %d: PID=%d\n", i, getpid());
        exit(0);  // Child terminates
    }
}

// Parent waits for all children
for (i=0; i<n; i++) {
    pid = wait(NULL);
    printf("Child %d terminated\n", pid);
}
printf("All children done\n");
```

**Pattern: Child waits, parent continues**

```c
if (fork() == 0) {
    // Child
    wait(NULL);  // Wait for... nothing! (no children)
    // Returns -1 immediately
} else {
    // Parent continues immediately
}
```

### What to Memorize

- [ ] wait() blocks if children still running
- [ ] wait() returns immediately if child already terminated
- [ ] wait() returns -1 if no children
- [ ] Zombie process: terminated but not reaped
- [ ] Orphan process: adopted by init (PID 1)
- [ ] Always call wait() to prevent zombies
- [ ] init automatically reaps orphans

### Exam Patterns

**Type 1: Zombie identification**
"Does this code create zombies?"
```c
fork();
// No wait() → YES, creates zombie
```

**Type 2: Parent-child order**
"Which process terminates first?"
- Check sleep() calls
- Check wait() calls
- Determine execution order

**Type 3: Orphan detection**
"What is child's PPID when it prints?"
- If parent terminates first → PPID = 1
- Otherwise → PPID = parent's PID

---

## SUMMARY: Critical Exam Knowledge

### MOST IMPORTANT FORMULAS:

**1. Sequential Forks:**
```
n sequential fork() calls → 2^n total processes
```

**2. Fork Return Values:**
```
Parent: receives child PID (positive)
Child: receives 0
Error: receives -1
```

**3. Variable After Fork:**
```
Initial value inherited
Changes independent
Separate memory spaces
```

### MEMORIZATION CHECKLIST:

**fork() Fundamentals:**
- [ ] Called once, returns twice
- [ ] Parent gets child PID, child gets 0
- [ ] Creates exact duplicate
- [ ] Shared: code, file descriptors
- [ ] Different: PID, return value, data

**Process Counting:**
- [ ] 2^n formula for sequential forks
- [ ] Draw process tree for complex cases
- [ ] Check conditions (if statements)
- [ ] Count all processes (including parent)

**Output Prediction:**
- [ ] setbuf(stdout, 0) disables buffering
- [ ] fflush(stdout) forces immediate output
- [ ] Without these: duplicate output possible
- [ ] Check before every exam question!

**wait() Behavior:**
- [ ] Blocks if children running
- [ ] Returns immediately if child terminated
- [ ] Returns -1 if no children
- [ ] Prevents zombie processes

**Common Patterns:**
- [ ] N children: fork + break in loop
- [ ] Conditional forks: check who executes
- [ ] Nested forks: draw CFG
- [ ] Wait for all: loop with n wait() calls

### EXAM STRATEGY:

**Time Allocation (for fork questions):**
- Read carefully: 1 minute
- Draw process tree: 2-3 minutes
- Count/calculate: 1-2 minutes
- Verify: 1 minute
- **Total: 5-7 minutes per fork question**

**Step-by-Step Method:**
1. **Label each fork** with comment (#1, #2, etc.)
2. **Identify conditions** (if statements)
3. **Draw process tree** showing parent-child
4. **Track variables** for each process
5. **Check for setbuf/fflush**
6. **Calculate output** or count

**Common Traps to Avoid:**
❌ Counting fork calls instead of processes
❌ Forgetting child executes code too
❌ Ignoring setbuf/fflush presence
❌ Thinking variables shared after fork
❌ Missing break in N-children loop
❌ Confusing parent (pid>0) and child (pid==0)

### PRACTICE PRIORITIES:

**MUST PRACTICE (70% of study time):**
1. **Sequential forks** - Master 2^n formula
2. **Conditional forks** - Track execution paths
3. **Loop with forks** - Understand iteration behavior
4. **Output prediction** - Consider buffering
5. **N children creation** - Break pattern

**SHOULD PRACTICE (20%):**
6. **Nested function calls** with fork
7. **wait() synchronization**
8. **Zombie and orphan processes**

**OPTIONAL (10%):**
9. Complex variable tracking
10. Multiple conditionals

---

## FINAL EXAM TIPS:

**Before Answering Fork Questions:**
1. ✓ Check for setbuf or fflush
2. ✓ Label each fork
3. ✓ Draw process tree (even if not asked)
4. ✓ Verify your count
5. ✓ Double-check conditions

**If Stuck:**
- Start with simpler case (n=2 instead of n=3)
- Draw CFG and process tree
- Count processes systematically
- Eliminate obviously wrong answers

**Time Management:**
- Fork questions: 5-7 minutes each
- Don't spend > 10 minutes on one question
- Come back if stuck

---

**End of Pages 301-400 Study Guide**
**Total Coverage: Pages 1-400 Complete**
**Next: exec(), Signals, and Pipes (also heavily tested!)**

**🔥 THIS SECTION IS 40% OF YOUR EXAM SUCCESS! MASTER IT! 🔥**
