# Operating Systems - Exam-Oriented Study Guide
## Pages 401-500: wait(), exec(), and Signals ⭐⭐⭐⭐⭐

**CRITICAL EXAM MATERIAL - Heavily Tested!**

**Previous Sections:** Pages 1-400 covered commands, file I/O, filters, and fork()

---

## ⚠️ EXAM IMPORTANCE ⚠️

**This section represents approximately 25% of exam content:**
- **wait() and synchronization** - Frequently tested
- **exec() family** - Critical for process replacement understanding
- **Signals** - Important for async communication
- **Precedence graphs** - Common exam pattern

**Combined with fork() (pages 301-400), process management topics account for 60%+ of exam points!**

---

## SECTION 23: wait() and waitpid() Details ⭐⭐⭐⭐⭐
**Slides 50-59 | Exam Relevance: CRITICAL**

### 23.1 wait() with Status ⭐⭐⭐⭐⭐

**System Call:**
```c
#include <sys/wait.h>

pid_t wait(int *statLoc);

// Returns:
//   - PID of terminated child on success
//   - -1 on error (no children)
```

**Status Information (MEMORIZE!):**

The `statLoc` parameter is a pointer to an integer that receives the exit status of the child process.

**Status Macros:**
```c
WIFEXITED(statLoc)        // True if child terminated normally
WEXITSTATUS(statLoc)      // Gets 8 LSBs of exit() value
WIFSIGNALED(statLoc)      // True if child terminated by signal
WTERMSIG(statLoc)         // Gets signal number that caused termination
```

**Example:**
```c
pid_t pid, childPid;
int statVal;

pid = fork();
if (pid == 0) {
    // Child
    sleep(5);
    exit(6);  // Exit with value 6
} else {
    // Parent
    childPid = wait(&statVal);
    printf("Child terminated: PID = %d\n", childPid);
    
    if (WIFEXITED(statVal)) {
        // Child exited normally
        printf("Exit value: %d\n", WEXITSTATUS(statVal));  // Prints 6
    } else {
        printf("Abnormal termination\n");
    }
}
```

**Retrieving Exit Status in Shell:**
```bash
$ ./program
$ echo $?      # Prints exit status of last command
```

### 23.2 Zombie Processes Revisited ⭐⭐⭐⭐⭐

**Definition (CRITICAL):**
- Child process that has terminated
- Parent is still running
- Parent has NOT executed wait()

**Why Zombies Exist:**
- Child's exit status must be preserved for parent
- Entry remains in process table
- Resources not fully released
- Only removed when parent calls wait()

**Problem:**
```c
for (i=0; i<100; i++) {
    if (fork() == 0) {
        // Child
        exit(0);
    }
    // Parent doesn't call wait()
}
// Creates 100 zombie processes!
```

**Solution:**
```c
for (i=0; i<100; i++) {
    if (fork() == 0) {
        // Child
        exit(0);
    } else {
        wait(NULL);  // Reap zombie
    }
}
```

### 23.3 Orphan Processes ⭐⭐⭐⭐

**Definition:**
- Child process whose parent terminated before it
- Automatically adopted by init (PID 1) or custom init-user
- Will NOT become zombie (init automatically reaps)

**Example:**
```c
pid = fork();
if (pid == 0) {
    // Child
    sleep(5);
    printf("Child: PPID=%d\n", getppid());  // Will print 1 or custom init PID
} else {
    // Parent
    printf("Parent: Terminating\n");
    exit(0);  // Parent dies immediately
}
```

**Output:**
```
Parent: Terminating
(5 seconds later)
Child: PPID=1    ← init is now parent!
```

**Key Points:**
- Orphans are adopted by init
- Init automatically reaps them when they terminate
- No zombie problem with orphans

### 23.4 waitpid() System Call ⭐⭐⭐⭐

**More Control Than wait():**

```c
#include <sys/wait.h>

pid_t waitpid(pid_t pid, int *statLoc, int options);

// Returns:
//   - PID of terminated child on success
//   - 0 if WNOHANG and child still running
//   - -1 on error
```

**pid Parameter (MEMORIZE):**
```c
pid > 0     // Wait for specific child with PID = pid
pid == -1   // Wait for any child (same as wait())
pid == 0    // Wait for any child with same GID as caller
pid < -1    // Wait for any child with GID = abs(pid)
```

**options Parameter:**
```c
0           // Default: blocking wait
WNOHANG     // Non-blocking: return immediately if no child terminated
WCONTINUED  // Also return for stopped children that continued
WUNTRACED   // Also return for stopped children
```

**Examples:**
```c
// Wait for specific child
childPid = waitpid(pid, &status, 0);

// Non-blocking check
childPid = waitpid(-1, &status, WNOHANG);
if (childPid == 0) {
    printf("No child terminated yet\n");
}

// Wait for any child in same group
childPid = waitpid(0, &status, 0);
```

**Difference from wait():**
- wait(): blocks until ANY child terminates
- waitpid(): can wait for SPECIFIC child
- waitpid(): can be NON-BLOCKING with WNOHANG

### What to Memorize

**wait() Behavior:**
- [ ] Returns -1 if no children
- [ ] Blocks if children running
- [ ] Returns immediately if child already terminated
- [ ] Reaps zombies

**Status Macros:**
- [ ] WIFEXITED() - normal termination?
- [ ] WEXITSTATUS() - get exit value
- [ ] WIFSIGNALED() - terminated by signal?

**Zombie:**
- [ ] Terminated but not reaped
- [ ] Parent must call wait()
- [ ] Remains in process table

**Orphan:**
- [ ] Parent terminated first
- [ ] Adopted by init (PID 1)
- [ ] Won't become zombie

**waitpid():**
- [ ] pid > 0: specific child
- [ ] pid == -1: any child
- [ ] WNOHANG: non-blocking

---

## SECTION 24: Precedence Graphs ⭐⭐⭐⭐⭐
**Slides 60-76 | Exam Relevance: VERY HIGH**

### 24.1 Precedence Graph Concept ⭐⭐⭐⭐⭐

**Definition:**
A directed graph showing execution order constraints between statements.

**Notation:**
```
S1 → S2    means S2 can only execute after S1 completes
```

**Implementation:**
- Use fork() to create concurrent processes
- Use wait() to enforce ordering

### 24.2 Basic Patterns ⭐⭐⭐⭐⭐

**Pattern 1: Simple Sequential**
```
S1
↓
S2
↓
S3
```

**Implementation:**
```c
printf("S1\n");
printf("S2\n");
printf("S3\n");
```
No fork needed - already sequential!

**Pattern 2: Fork and Join**
```
S1
↓
S2 → S4
↓
S3 →
```

**Implementation:**
```c
printf("S1\n");
pid = fork();
if (pid == 0) {
    // Child
    printf("S3\n");
    exit(0);
} else {
    // Parent
    printf("S2\n");
    wait(NULL);
    printf("S4\n");
}
```

**Execution Order:**
- S1 first (before fork)
- S2 and S3 concurrent (may interleave)
- S4 after both S2 and S3 (wait ensures this)

### 24.3 Complex Precedence Graphs ⭐⭐⭐⭐⭐

**EXAM PATTERN: Multi-level fork and wait**

**Example:**
```
S1
↓
S2 ──→ S4
↓      ↓
S3 ──→ S7
↓
S5 ──→
↓
S6 ──→
```

**Solution Strategy:**
1. **Identify levels** of parallelism
2. **Use fork()** for branching
3. **Use wait()** for joining

**Implementation:**
```c
printf("S1\n");

pid = fork();
if (pid == 0) {
    // Child path: S3, S5, S6
    printf("S3\n");
    pid2 = fork();
    if (pid2 == 0) {
        printf("S6\n");
        exit(0);
    } else {
        printf("S5\n");
        wait(NULL);
    }
    exit(0);
} else {
    // Parent path: S2, S4
    printf("S2\n");
    wait(NULL);  // Wait for child to finish S3, S5, S6
    printf("S4\n");
}

printf("S7\n");
```

### 24.4 Unfeasible Graphs ⭐⭐⭐⭐

**Some precedence graphs CANNOT be implemented** with fork/wait alone!

**Example of Unfeasible Graph:**
```
S1 ──→ S2
↓      ↓
S3 ←── S4
```

**Why Unfeasible:**
- S3 needs to come AFTER S1
- S3 needs to come AFTER S4
- But S4 comes AFTER S2
- And S2 comes AFTER S1
- Creates a cycle: impossible with fork/wait!

**Exam Tip:** If you see crossing dependencies, it's likely unfeasible.

### 24.5 Exam Strategies ⭐⭐⭐⭐⭐

**Step-by-Step Method:**

1. **Identify independent branches**
   - Statements that can execute concurrently
   - These require fork()

2. **Identify synchronization points**
   - Where branches must join
   - These require wait()

3. **Draw the process tree**
   - Shows which process executes which statements

4. **Trace execution paths**
   - Parent path
   - Child path(s)

**Example Problem:**
"Implement this precedence graph with fork/wait:"
```
S1
↓
S2 ──→ S5
↓      
S3 ──→
↓
S4 ──→
```

**Solution:**
```c
int main() {
    pid_t pid;
    
    printf("S1\n");
    
    pid = fork();
    if (pid == 0) {
        // Child: S3, S4
        printf("S3\n");
        printf("S4\n");
        exit(0);
    } else {
        // Parent: S2, wait, S5
        printf("S2\n");
        wait(NULL);
        printf("S5\n");
    }
    
    return 0;
}
```

**Execution:**
- S1 (parent before fork)
- S2 (parent) and S3 (child) concurrent
- S4 (child continues)
- Child exits
- S5 (parent after wait)

### What to Memorize

**Precedence Graph Implementation:**
- [ ] Fork creates parallel execution
- [ ] Wait synchronizes (join point)
- [ ] Draw process tree to verify
- [ ] Check if feasible first

**Common Patterns:**
- [ ] Sequential: no fork needed
- [ ] Fork-Join: fork + wait
- [ ] Multiple children: multiple forks in parent
- [ ] Nested parallelism: fork in child

**Exam Strategy:**
- [ ] Identify branches first
- [ ] Identify synchronization points
- [ ] Draw tree, trace paths
- [ ] Verify all constraints satisfied

---

## SECTION 25: exec() Family ⭐⭐⭐⭐⭐
**Slides 2-34 (Advanced Control chapter) | Exam Relevance: CRITICAL**

### 25.1 exec() Concept ⭐⭐⭐⭐⭐

**What exec() Does:**
- **Replaces** the current process image with a new program
- Does NOT create new process
- PID remains the same
- Starts execution from new program's main()

**Key Difference:**
```
fork()  → Creates NEW process (duplicate)
exec()  → Replaces CURRENT process (new program)
```

**Address Space Transformation:**
```
Before exec():          After exec():
┌──────────┐           ┌──────────┐
│ Old Code │           │ New Code │  ← Replaced
├──────────┤           ├──────────┤
│ Old Data │           │ New Data │  ← Replaced
├──────────┤           ├──────────┤
│ Old Stack│           │New Stack │  ← Replaced
├──────────┤           ├──────────┤
│ Old Heap │           │ New Heap │  ← Replaced
├──────────┤           ├──────────┤
│   PCB    │    →      │   PCB    │  ← PID SAME!
└──────────┘           └──────────┘
```

**CRITICAL Concept:**
- exec() only returns on ERROR
- If successful, old program is GONE
- Code after exec() NEVER executes (unless exec fails)

### 25.2 exec() Family ⭐⭐⭐⭐⭐

**Six Versions (MEMORIZE!):**

```c
#include <unistd.h>

int execl(char *path, char *arg0, ..., (char *)0);
int execlp(char *name, char *arg0, ..., (char *)0);
int execle(char *path, char *arg0, ..., (char *)0, char *envp[]);

int execv(char *path, char *argv[]);
int execvp(char *name, char *argv[]);
int execve(char *path, char *argv[], char *envp[]);

// Returns: -1 on error, does NOT return on success
```

**Naming Convention (MEMORIZE!):**

```
exec + [l|v] + [p] + [e]
       │ │     │     │
       │ │     │     └─ e: environment vector provided
       │ │     └─────── p: search PATH for executable
       │ └───────────── v: argv[] vector
       └─────────────── l: list of arguments
```

**Letter Meanings:**
- **l** (list): Arguments passed as list
- **v** (vector): Arguments passed as array (char *argv[])
- **p** (path): Search PATH environment variable for executable
- **e** (environment): Custom environment variables

### 25.3 Detailed exec() Variants ⭐⭐⭐⭐⭐

**execl() - List, Full Path:**
```c
execl("/bin/ls", "ls", "-l", "-a", NULL);
       │          │     │    │     └─ Must end with NULL!
       │          │     └──────────── Arguments
       │          └────────────────── argv[0] (program name)
       └───────────────────────────── Full path required
```

**execlp() - List, Search PATH:**
```c
execlp("ls", "ls", "-l", "-a", NULL);
        │     │     │    │     └─ Must end with NULL
        │     │     └──────────── Arguments
        │     └────────────────── argv[0]
        └──────────────────────── Name only (searches PATH)
```

**execv() - Vector, Full Path:**
```c
char *args[] = {"ls", "-l", "-a", NULL};
execv("/bin/ls", args);
       │          └───── argv[] array
       └────────────── Full path
```

**execvp() - Vector, Search PATH:**
```c
char *args[] = {"ls", "-l", "-a", NULL};
execvp("ls", args);
        │     └───── argv[] array (NULL-terminated!)
        └─────────── Name only
```

**execle() - List, Environment:**
```c
char *env[] = {"PATH=/tmp", "USER=test", NULL};
execle("/bin/ls", "ls", "-l", NULL, env);
        │          │     │     │     └── Environment array
        │          │     │     └──────── NULL terminator
        │          │     └────────────── Arguments
        │          └──────────────────── argv[0]
        └─────────────────────────────── Full path
```

**execve() - Vector, Environment:**
```c
char *args[] = {"ls", "-l", NULL};
char *env[] = {"PATH=/tmp", NULL};
execve("/bin/ls", args, env);
```

### 25.4 exec() Practical Examples ⭐⭐⭐⭐⭐

**Example 1: Copy Command**
```c
// Execute: cp ./file1 ./file2

// Using execl (list, full path):
execl("/bin/cp", "mycp", "./file1", "./file2", NULL);

// Using execlp (list, search PATH):
execlp("cp", "mycp", "./file1", "./file2", NULL);

// Using execv (vector, full path):
char *cmd[] = {"mycp", "./file1", "./file2", NULL};
execv("/bin/cp", cmd);

// Using execvp (vector, search PATH):
char *cmd[] = {"mycp", "./file1", "./file2", NULL};
execvp("cp", cmd);
```

**Example 2: Self-Replacement**
```c
int main(int argc, char **argv) {
    int n = atoi(argv[1]);
    
    printf("n = %d, PID = %d\n", n, getpid());
    
    if (n > 0) {
        char str[10];
        sprintf(str, "%d", n-1);
        execl(argv[0], argv[0], str, NULL);
        // This line NEVER executes (unless exec fails)
        printf("exec failed!\n");
    }
    
    printf("End!\n");
    return 0;
}
```

**Execution with argv[1] = "5":**
```
n = 4, PID = 1234
n = 3, PID = 1234  ← Same PID!
n = 2, PID = 1234
n = 1, PID = 1234
n = 0, PID = 1234
End!
```

**Output: 4 3 2 1 0 End!**

Only ONE process! PID never changes!

**Example 3: fork() + exec() Pattern**
```c
pid = fork();
if (pid == 0) {
    // Child: run different program
    execl("/bin/ls", "ls", "-l", NULL);
    // If we get here, exec failed
    fprintf(stderr, "exec failed\n");
    exit(1);
} else {
    // Parent: continue with original program
    wait(NULL);
    printf("Child finished\n");
}
```

This is the **standard pattern** for running external programs!

### 25.5 exec() Important Details ⭐⭐⭐⭐

**What is Preserved Across exec():**
- PID (process ID)
- PPID (parent process ID)
- Open file descriptors (including stdin, stdout, stderr)
- UID, GID
- Working directory
- Environment variables (unless 'e' version used)

**What is Replaced:**
- Code (text segment)
- Data segment
- Stack
- Heap

**argv[0] Significance:**
```c
execl("/bin/cp", "mycp", "./file1", "./file2", NULL);
                 ^^^^^^
                 This is argv[0] in the new program!
```
- First argument becomes argv[0]
- Allows program to behave differently based on invocation name

**NULL Terminator (CRITICAL!):**
```c
execl("/bin/ls", "ls", "-l", NULL);  ✓ Correct
execl("/bin/ls", "ls", "-l", 0);    ✓ Also correct
execl("/bin/ls", "ls", "-l");       ✗ WRONG! Undefined behavior
```

**Vector Must Be NULL-Terminated:**
```c
char *args[] = {"ls", "-l", NULL};  ✓ Correct
char *args[] = {"ls", "-l"};       ✗ WRONG!
```

### What to Memorize

**exec() Basics:**
- [ ] Replaces current process
- [ ] Does NOT create new process
- [ ] PID stays same
- [ ] Only returns on error (-1)
- [ ] Success = old program gone

**Naming:**
- [ ] l = list of arguments
- [ ] v = vector (argv[])
- [ ] p = search PATH
- [ ] e = environment variables

**Common Patterns:**
- [ ] execl() - list, full path
- [ ] execlp() - list, search PATH
- [ ] execv() - vector, full path
- [ ] execvp() - vector, search PATH

**Critical Details:**
- [ ] Must end with NULL
- [ ] argv[0] = program name
- [ ] File descriptors preserved
- [ ] fork() + exec() pattern

---

## SECTION 26: system() System Call ⭐⭐⭐⭐
**Slides 24-34 | Exam Relevance: High**

### 26.1 system() Concept ⭐⭐⭐⭐

**Purpose:**
Execute a shell command from within a C program.

**System Call:**
```c
#include <stdlib.h>

int system(const char *string);

// Returns:
//   -1 if fork() or waitpid() fails
//   127 if exec() fails
//   Exit status of shell command
```

**What it Does:**
1. Forks a shell (/bin/sh)
2. Shell executes the command string
3. Parent waits for shell to complete
4. Returns shell's exit status

**Example:**
```c
system("ls -la");              // List files
system("date");                 // Show date
system("date > file");          // Redirect output
system("echo Hello");           // Simple command

char cmd[100];
strcpy(cmd, "ls -la");
system(cmd);                    // From string variable
```

### 26.2 system() Implementation ⭐⭐⭐⭐

**How system() Works Internally:**
```c
int system(const char *cmd) {
    pid_t pid;
    int status;
    
    if (cmd == NULL)
        return 1;
    
    if ((pid = fork()) < 0) {
        status = -1;  // Fork failed
    } else if (pid == 0) {
        // Child: execute shell
        execl("/bin/sh", "sh", "-c", cmd, (char *)0);
        _exit(127);   // exec failed
    } else {
        // Parent: wait for shell
        while (waitpid(pid, &status, 0) < 0) {
            if (errno != EINTR) {
                status = -1;
                break;
            }
        }
    }
    
    return status;
}
```

**Key Points:**
- Uses fork() to create child
- Child executes shell with command
- Parent waits with waitpid()
- Returns shell exit status

### 26.3 system() vs exec() ⭐⭐⭐⭐

**Differences:**

```c
// exec() - replaces current process
execl("/bin/ls", "ls", "-l", NULL);
printf("This never prints\n");  // Never reached

// system() - creates new process, returns
system("ls -l");
printf("This prints!\n");       // Executes!
```

**When to Use:**
- **exec()**: Want to replace current process entirely
- **system()**: Want to run command and continue

**Example with exec():**
```c
int main(int argc, char **argv) {
    int n = atoi(argv[1]);
    
    if (n > 0) {
        printf("%d\n", n);
        char str[10];
        sprintf(str, "%d", n-1);
        execl(argv[0], argv[0], str, NULL);
    }
    printf("End!\n");
    return 0;
}
// Input: 4
// Output: 4 3 2 1 0 End!
// Only ONE "End!" - no parent survives
```

**Example with system():**
```c
int main(int argc, char **argv) {
    int n = atoi(argv[1]);
    
    if (n > 0) {
        printf("%d\n", n);
        char str[100];
        sprintf(str, "%s %d", argv[0], n-1);
        system(str);
    }
    printf("End!\n");
    return 0;
}
// Input: 4
// Output: 4 3 2 1 0 End! End! End! End! End!
// FIVE "End!" - each call returns!
```

**Process Tree Difference:**

**exec():**
```
P(4) → exec → P(3) → exec → P(2) → exec → P(1) → exec → P(0)
  │                                                        │
  └────────────────────────────────────────────────────────┘
                 Same process throughout!
```

**system():**
```
P(4)
 ├─ shell → P(3)
 │           ├─ shell → P(2)
 │           │           ├─ shell → P(1)
 │           │           │           ├─ shell → P(0)
 │           │           │           │
 │           │           │           └─ End!
 │           │           │
 │           │           └─ End!
 │           │
 │           └─ End!
 │
 └─ End!
End!
```

Multiple processes! Each survives!

### What to Memorize

**system():**
- [ ] Forks shell to run command
- [ ] Returns after command completes
- [ ] Can use shell features (redirection, pipes)
- [ ] Returns shell exit status

**system() vs exec():**
- [ ] exec(): replaces process, never returns
- [ ] system(): creates child, returns
- [ ] exec(): one "End!" in example
- [ ] system(): multiple "End!" in example

---

## SECTION 27: Shell Skeleton ⭐⭐⭐
**Slides 22-23 | Exam Relevance: Medium**

### 27.1 Shell Implementation ⭐⭐⭐

**Basic Shell Loop:**

**Foreground Command:**
```c
while (TRUE) {
    write_prompt();
    read_command(command, parameters);
    
    if (fork() == 0) {
        // Child: Execute command
        execve(command, parameters);
    } else {
        // Parent: Wait for child
        wait(&status);
    }
}
```

**Background Command (&):**
```c
while (TRUE) {
    write_prompt();
    read_command(command, parameters);
    
    if (fork() == 0) {
        // Child: Execute command
        execve(command, parameters);
    }
    // Parent: DOES NOT wait!
    // Continues immediately to next prompt
}
```

**Key Difference:**
- Foreground: Parent waits → user gets prompt after command finishes
- Background: Parent doesn't wait → user gets prompt immediately

### What to Know (Brief)

- Shell = loop of fork/exec/wait
- Foreground: shell waits
- Background: shell continues
- **Not heavily tested** - conceptual understanding sufficient

---

## SECTION 28: Process Theory (Brief) ⭐⭐
**Slides 2-12 (Theoretical Aspects) | Exam Relevance: Low-Medium**

### 28.1 Process Concepts ⭐⭐

**Program vs Process (Repeated for Emphasis):**
- **Program:** Passive entity, code on disk
- **Process:** Active entity, program in execution

**Process Memory Layout:**
```
┌──────────────┐ ← High address
│    Stack     │   Function params, local vars
├──────────────┤   ↓ Grows downward
│      ↓       │
│              │
│      ↑       │
├──────────────┤   ↑ Grows upward
│     Heap     │   Dynamic allocation (malloc)
├──────────────┤
│     Data     │   Global variables
├──────────────┤
│     Text     │   Program code
└──────────────┘ ← Low address
```

### 28.2 Process Control Block (PCB) ⭐⭐

**PCB Contains:**
- Process state (New, Ready, Running, Waiting, Terminated)
- Program counter
- CPU registers
- CPU scheduling info (priority, queue pointers)
- Memory management info (page tables, segments)
- I/O status (open files, devices)
- Signal table

### 28.3 Process States ⭐⭐

**State Diagram:**
```
        New
         ↓
     ┌───────┐
     │ Ready │ ←────┐
     └───────┘      │
         ↓          │
    ┌─────────┐    │
    │ Running │    │
    └─────────┘    │
         ↓          │
     ┌─────────┐   │
     │ Waiting │ ──┘
     └─────────┘
         ↓
     Terminated
```

**Transitions:**
- Ready → Running: CPU scheduler selects process
- Running → Ready: Time slice expired, higher priority arrived
- Running → Waiting: I/O request, wait for event
- Waiting → Ready: I/O complete, event occurred

### 28.4 Context Switching ⭐⭐

**Context Switch:**
1. Save state of current process (registers, PC to PCB)
2. Load state of new process (from PCB to registers, PC)
3. Switch memory context (page tables)

**Overhead:**
- Time spent in context switching is pure overhead
- No useful work done during switch
- Hardware-dependent (faster with hardware support)

### What to Know (Brief)

- Process = running program
- PCB stores process state
- Five states: New, Ready, Running, Waiting, Terminated
- Context switch = save/restore process state
- **Minimal exam impact** - background concepts

---

## SECTION 29: Signals ⭐⭐⭐⭐⭐
**Slides 2-29 (Signals chapter) | Exam Relevance: HIGH**

### 29.1 Signal Concepts ⭐⭐⭐⭐⭐

**Definition:**
- Software interrupt
- Asynchronous notification to a process
- Sent by kernel or another process
- Notifies occurrence of an event

**Characteristics:**
- Asynchronous (can arrive anytime)
- Limited form of IPC (Inter-Process Communication)
- Standard across POSIX systems

**Signal Lifecycle:**
1. **Generation:** Event occurs, signal created
2. **Delivery:** Signal delivered to process
3. **Handling:** Process reacts to signal

### 29.2 Common Signals ⭐⭐⭐⭐⭐

**MUST MEMORIZE:**

```c
SIGINT     // Ctrl-C, interrupt from keyboard
           // Default: terminate process

SIGTERM    // Termination signal
           // Default: terminate process

SIGKILL    // Kill signal (cannot be caught or ignored!)
           // Default: terminate process

SIGCHLD    // Child process terminated or stopped
           // Default: ignore

SIGALRM    // Timer signal from alarm()
           // Default: terminate process

SIGUSR1    // User-defined signal 1
SIGUSR2    // User-defined signal 2
           // Default: terminate process

SIGTSTP    // Ctrl-Z, stop signal from terminal
           // Default: stop process

SIGQUIT    // Ctrl-\, quit from keyboard
           // Default: terminate with core dump

SIGPIPE    // Broken pipe (write to pipe with no reader)
           // Default: terminate process

SIGSEGV    // Invalid memory access (segmentation fault)
           // Default: terminate with core dump

SIGFPE     // Floating point exception
           // Default: terminate with core dump

SIGILL     // Illegal instruction
           // Default: terminate with core dump
```

**View All Signals:**
```bash
$ kill -l
```

### 29.3 signal() System Call ⭐⭐⭐⭐⭐

**System Call:**
```c
#include <signal.h>

void (*signal(int sig, void (*func)(int)))(int);

// Simplified typedef:
typedef void (*sighandler_t)(int);
sighandler_t signal(int sig, sighandler_t func);

// Returns:
//   - Previous signal handler on success
//   - SIG_ERR on error
```

**Three Reactions to Signal:**

**1. Default Behavior:**
```c
signal(SIGINT, SIG_DFL);

// SIG_DFL = default handler (usually terminate)
```

**2. Ignore Signal:**
```c
signal(SIGINT, SIG_IGN);

// Process will ignore SIGINT (Ctrl-C does nothing)
// Cannot ignore: SIGKILL, SIGSTOP
```

**3. Catch Signal (Custom Handler):**
```c
void my_handler(int sig) {
    printf("Received signal %d\n", sig);
    // Handle signal
    return;
}

signal(SIGINT, my_handler);
```

### 29.4 Signal Handler Examples ⭐⭐⭐⭐⭐

**Example 1: Basic Handler**
```c
#include <signal.h>
#include <stdio.h>
#include <unistd.h>

void manager(int sig) {
    printf("Received signal %d\n", sig);
    return;
}

int main() {
    signal(SIGINT, manager);  // Install handler
    
    while (1) {
        printf("main: Hello!\n");
        sleep(1);
    }
    
    return 0;
}
```

**Behavior:**
- Pressing Ctrl-C sends SIGINT
- manager() is called
- Prints "Received signal 2"
- Program continues (doesn't terminate!)

**Example 2: Multiple Signals, One Handler**
```c
void manager(int sig) {
    if (sig == SIGUSR1)
        printf("Received SIGUSR1\n");
    else if (sig == SIGUSR2)
        printf("Received SIGUSR2\n");
    else
        printf("Received signal %d\n", sig);
    return;
}

int main() {
    signal(SIGUSR1, manager);
    signal(SIGUSR2, manager);
    
    // Both signal types use same handler
    pause();  // Wait for any signal
    
    return 0;
}
```

**Example 3: SIGCHLD Handler**

**Synchronous (with wait):**
```c
if (fork() == 0) {
    // Child
    sleep(1);
    exit(2);
} else {
    // Parent
    sleep(5);
    pid = wait(&code);
    printf("Child %d exited with %d\n", pid, WEXITSTATUS(code));
}
```

**Asynchronous (with signal handler):**
```c
void sigchld_handler(int signo) {
    if (signo == SIGCHLD)
        printf("Received SIGCHLD\n");
    return;
}

signal(SIGCHLD, sigchld_handler);

if (fork() == 0) {
    // Child
    exit(0);
} else {
    // Parent continues
    // Handler called automatically when child exits
    pause();  // Wait for signal
}
```

**SIGCHLD Special Behavior:**
```c
// Ignore SIGCHLD - prevents zombies!
signal(SIGCHLD, SIG_IGN);

if (fork() == 0) {
    exit(0);
}
// No wait() needed - child won't become zombie!
```

**CRITICAL:** `signal(SIGCHLD, SIG_IGN)` prevents zombies, but `signal(SIGCHLD, SIG_DFL)` does NOT!

### 29.5 kill() System Call ⭐⭐⭐⭐⭐

**System Call:**
```c
#include <signal.h>

int kill(pid_t pid, int sig);

// Returns:
//   0 on success
//   -1 on error
```

**pid Parameter (MEMORIZE!):**
```c
pid > 0    // Send signal to process with PID = pid
pid == 0   // Send to all processes in same group as sender
pid < -1   // Send to all processes with GID = abs(pid)
pid == -1  // Send to all processes (broadcast, superuser only)
```

**Examples:**
```c
// Send SIGTERM to process 1234
kill(1234, SIGTERM);

// Send SIGUSR1 to all processes in group
kill(0, SIGUSR1);

// Check if process exists (send NULL signal)
if (kill(pid, 0) == -1) {
    printf("Process %d does not exist\n", pid);
}

// Terminate all children in group
kill(0, SIGKILL);
```

**Permission Rules:**
- User process can only send signals to processes with same UID
- Superuser can send signals to any process
- Some signals (SIGKILL, SIGSTOP) cannot be caught or ignored

### 29.6 Other Signal Functions ⭐⭐⭐

**raise():**
```c
#include <signal.h>

int raise(int sig);

// Send signal to self
// Equivalent to: kill(getpid(), sig)
```

**pause():**
```c
#include <unistd.h>

int pause(void);

// Suspend process until signal received
// Returns: -1 (after signal handler completes)
```

**alarm():**
```c
#include <unistd.h>

unsigned int alarm(unsigned int seconds);

// Schedule SIGALRM signal after 'seconds'
// Returns: seconds remaining from previous alarm, or 0
```

**Example: Timeout with alarm:**
```c
void alarm_handler(int sig) {
    printf("Timeout!\n");
    exit(1);
}

signal(SIGALRM, alarm_handler);
alarm(5);  // Set 5-second timeout

// Do some work...
// If work takes > 5 seconds, SIGALRM sent

alarm(0);  // Cancel alarm if work completes
```

**sleep():**
```c
#include <unistd.h>

unsigned int sleep(unsigned int seconds);

// Suspend for 'seconds'
// Returns: 0 if full time elapsed
//          seconds remaining if interrupted by signal
```

### What to Memorize

**Common Signals:**
- [ ] SIGINT (Ctrl-C) - interrupt
- [ ] SIGTERM - terminate
- [ ] SIGKILL - kill (uncatchable!)
- [ ] SIGCHLD - child terminated
- [ ] SIGALRM - alarm timer
- [ ] SIGUSR1, SIGUSR2 - user signals

**signal() Usage:**
- [ ] SIG_DFL - default behavior
- [ ] SIG_IGN - ignore signal
- [ ] handler function - catch signal
- [ ] Returns previous handler

**kill() pid Values:**
- [ ] pid > 0 - specific process
- [ ] pid == 0 - process group
- [ ] pid == -1 - all processes
- [ ] pid < -1 - specific group

**SIGCHLD Special:**
- [ ] signal(SIGCHLD, SIG_IGN) prevents zombies
- [ ] SIG_DFL does NOT prevent zombies
- [ ] Can use wait() or signal handler

### Exam Patterns

**Type 1: Handler Installation**
"What happens when Ctrl-C is pressed?"
```c
signal(SIGINT, SIG_IGN);
// Answer: Nothing (signal ignored)

signal(SIGINT, handler);
// Answer: handler() is called
```

**Type 2: kill() Target**
"Which processes receive the signal?"
```c
kill(1234, SIGTERM);  // Only PID 1234
kill(0, SIGUSR1);     // All in same group
kill(-1, SIGKILL);    // All processes (superuser)
```

**Type 3: SIGCHLD Behavior**
```c
signal(SIGCHLD, SIG_IGN);
fork();
// Answer: Child won't become zombie (no wait needed)
```

---

## SUMMARY: Pages 401-500 Key Concepts

### CRITICAL MEMORIZATION CHECKLIST:

**wait() and waitpid():**
- [ ] wait() returns -1 if no children
- [ ] wait() blocks until child terminates
- [ ] WIFEXITED(), WEXITSTATUS() macros
- [ ] Zombie = terminated but not reaped
- [ ] Orphan = adopted by init
- [ ] waitpid() can wait for specific child
- [ ] WNOHANG = non-blocking

**exec() Family:**
- [ ] exec() replaces process (PID same)
- [ ] Returns only on error (-1)
- [ ] l = list, v = vector
- [ ] p = search PATH
- [ ] e = environment
- [ ] Must end with NULL
- [ ] Fork + exec pattern

**system():**
- [ ] Forks shell, runs command, returns
- [ ] Uses fork/exec/wait internally
- [ ] Returns shell exit status

**Signals:**
- [ ] SIGINT, SIGTERM, SIGKILL
- [ ] SIGCHLD, SIGALRM
- [ ] signal(sig, SIG_DFL/SIG_IGN/handler)
- [ ] kill(pid, sig) - pid values
- [ ] signal(SIGCHLD, SIG_IGN) prevents zombies

**Precedence Graphs:**
- [ ] Fork = parallel execution
- [ ] Wait = synchronization point
- [ ] Draw process tree to verify
- [ ] Check feasibility first

### EXAM STRATEGY:

**Time Allocation:**
- wait() questions: 3-5 minutes
- exec() questions: 5-7 minutes
- Precedence graphs: 7-10 minutes
- Signal questions: 3-5 minutes

**Common Traps:**
❌ Thinking exec() creates new process
❌ Forgetting exec() doesn't return on success
❌ Missing NULL terminator in exec()
❌ Confusing system() and exec() behavior
❌ Not checking WIFEXITED() before WEXITSTATUS()
❌ Thinking signal(SIGCHLD, SIG_DFL) prevents zombies
❌ Forgetting kill(0, sig) sends to group

---

**End of Pages 401-500 Study Guide**
**Total Coverage: Pages 1-500 Complete**

**🔥 Combined with fork() knowledge, you now have 60%+ of exam content mastered! 🔥**
