# Operating Systems - Exam-Oriented Study Guide
## Pages 601-800: Threads (Pthreads) & Synchronization ⭐⭐⭐⭐

**IMPORTANT EXAM MATERIAL - Threads and Synchronization!**

**Previous Sections:** Pages 1-600 covered processes, fork(), exec(), signals, pipes

---

## ⚠️ EXAM IMPORTANCE ⚠️

**This section represents approximately 15-20% of exam content:**
- **Pthread library** ⭐⭐⭐⭐ - Thread creation and management
- **Synchronization concepts** ⭐⭐⭐⭐ - Critical sections
- **Race conditions** ⭐⭐⭐ - Concurrent access problems

**Note:** This guide covers pages 601-800. Exam weight may vary by course - threads are more heavily tested in courses with dedicated parallel programming focus.

---

## SECTION 37: Thread Models ⭐⭐⭐
**Slides 16-31 | Exam Relevance: Medium (Conceptual)**

### 37.1 Kernel-Level Threads ⭐⭐⭐

**Characteristics:**
- Managed by kernel
- OS aware of threads
- Thread table in kernel
- Thread Control Block (TCB) for each thread

**Advantages:**
- Global scheduling across all processes
- If one thread blocks, others can run (even in same process)
- True parallelism on multiprocessor
- Effective for I/O-bound applications

**Disadvantages:**
- Slow (system call overhead)
- Expensive context switching
- Limited number of threads
- Large memory overhead

### 37.2 User-Level Threads ⭐⭐⭐

**Characteristics:**
- Managed by user library
- Kernel unaware (sees only processes)
- Thread table per process
- No system calls for thread operations

**Advantages:**
- Can run on any OS
- Very fast (100x faster than kernel threads)
- Fast context switch
- Unlimited threads
- Custom scheduling possible

**Disadvantages:**
- If one thread blocks, entire process blocks
- No true parallelism (only one kernel thread)
- OS makes poor scheduling decisions

### 37.3 Hybrid Model ⭐⭐

**Modern Approach:**
- M user threads mapped to N kernel threads
- Typically N < M
- Combines advantages of both
- Used by Windows, Linux, Solaris

**Linux NPTL:**
- 1:1 threading model
- One user thread = one kernel thread

### 37.4 fork() and exec() with Threads ⭐⭐⭐

**fork() Behavior:**
- Duplicates only the calling thread
- Other threads not copied
- Use pthread_atfork for proper handling

**exec() Behavior:**
- Replaces entire process (all threads)
- Not just the calling thread

**Signal Delivery:**
- Delivered to single thread in process
- Specific signals → specific thread (pthread_sigmask)
- Generic signals → arbitrary thread

### What to Know (Brief)

- Kernel threads: slow, true parallelism
- User threads: fast, no parallelism
- Hybrid: modern approach
- fork() copies only calling thread
- exec() replaces all threads
- **Low-medium exam impact** - conceptual understanding

---

## SECTION 38: Pthread Library ⭐⭐⭐⭐⭐
**Slides 2-48 (Pthread chapter) | Exam Relevance: HIGH**

### 38.1 Pthread Basics ⭐⭐⭐⭐

**What is Pthread:**
- POSIX 1003.1c (1995) standard
- Standard UNIX thread library
- Available on Linux, Windows, Mac OS, BSD
- 60+ functions with pthread_* prefix

**Compilation:**
```bash
gcc -Wall -g -o program file.c -lpthread
```

**Include:**
```c
#include <pthread.h>
```

### 38.2 Thread Identifier ⭐⭐⭐⭐

**pthread_t Type:**
```c
pthread_t tid;  // Opaque type (implementation dependent)
```

**Functions:**

```c
pthread_t pthread_self(void);
// Returns: thread ID of calling thread

int pthread_equal(pthread_t tid1, pthread_t tid2);
// Returns: nonzero if equal, 0 otherwise
```

**IMPORTANT:**
- pthread_t is opaque - cannot directly compare or print
- Only meaningful within process (not global like PID)

### 38.3 pthread_create() ⭐⭐⭐⭐⭐

**System Call:**
```c
int pthread_create(
    pthread_t *tid,                    // OUT: thread ID
    const pthread_attr_t *attr,        // Attributes (NULL = default)
    void *(*startRoutine)(void *),    // Thread function
    void *arg                          // Argument to function
);

// Returns: 0 on success, error code on failure
```

**Thread Function Signature:**
```c
void *thread_function(void *arg) {
    // Thread code here
    return NULL;  // or pthread_exit(NULL);
}
```

**Basic Example:**
```c
void *myThread(void *arg) {
    printf("Hello from thread\n");
    pthread_exit(NULL);
}

int main() {
    pthread_t tid;
    int rc;
    
    rc = pthread_create(&tid, NULL, myThread, NULL);
    if (rc != 0) {
        fprintf(stderr, "pthread_create failed\n");
        exit(1);
    }
    
    pthread_exit(NULL);  // Let thread finish
    return 0;
}
```

### 38.4 Passing Arguments ⭐⭐⭐⭐⭐

**WRONG - Race Condition:**
```c
// BUGGY CODE!
for (int t = 0; t < N; t++) {
    pthread_create(&th[t], NULL, func, &t);  // ERROR!
}
// Problem: &t is same address for all threads
// t changes while threads read it
```

**Method 1: Array of Values (CORRECT)**
```c
int tA[N];
for (int t = 0; t < N; t++) {
    tA[t] = t;
    pthread_create(&th[t], NULL, func, &tA[t]);  // OK!
}

void *func(void *arg) {
    int id = *((int *)arg);
    // Use id...
}
```

**Method 2: Cast Integer as Pointer (TRICKY but OK)**
```c
for (long t = 0; t < N; t++) {
    pthread_create(&th[t], NULL, func, (void *)t);  // OK!
}

void *func(void *arg) {
    long id = (long)arg;  // Cast back
    // Use id...
}
```

**Method 3: Struct (CORRECT)**
```c
struct thread_data {
    int id;
    char name[100];
};

struct thread_data data[N];
for (int t = 0; t < N; t++) {
    data[t].id = t;
    strcpy(data[t].name, "thread");
    pthread_create(&th[t], NULL, func, &data[t]);
}

void *func(void *arg) {
    struct thread_data *d = (struct thread_data *)arg;
    int id = d->id;
    // Use id...
}
```

### 38.5 pthread_exit() and pthread_join() ⭐⭐⭐⭐⭐

**pthread_exit():**
```c
void pthread_exit(void *valuePtr);

// Terminates calling thread
// Returns valuePtr to joiner
```

**Process vs Thread Termination:**

**Process Terminates (all threads):**
- Any thread calls exit() or _exit()
- main() returns
- Receives termination signal

**Single Thread Terminates:**
- return from thread function
- pthread_exit()
- pthread_cancel() from another thread

**pthread_join():**
```c
int pthread_join(
    pthread_t tid,      // Thread to wait for
    void **valuePtr     // Receives return value (or NULL)
);

// Returns: 0 on success, error code on failure
// BLOCKS until specified thread terminates
```

**Complete Example:**
```c
void *thread_func(void *arg) {
    long id = (long)arg;
    printf("Thread %ld running\n", id);
    pthread_exit((void *)id);  // Return id
}

int main() {
    pthread_t threads[3];
    void *status;
    long t;
    
    // Create threads
    for (t = 0; t < 3; t++) {
        pthread_create(&threads[t], NULL, thread_func, (void *)t);
    }
    
    // Wait for all threads
    for (t = 0; t < 3; t++) {
        pthread_join(threads[t], &status);
        long result = (long)status;
        printf("Thread %ld returned %ld\n", t, result);
    }
    
    return 0;
}
```

**CRITICAL:** 
- Main thread should call pthread_exit(NULL), NOT exit(0)!
- exit(0) terminates ALL threads
- pthread_exit(NULL) allows other threads to continue

### 38.6 pthread_cancel() and pthread_detach() ⭐⭐⭐

**pthread_cancel():**
```c
int pthread_cancel(pthread_t tid);

// Terminates target thread
// Effect: like pthread_exit(PTHREAD_CANCELED)
// Does NOT wait for termination
```

**pthread_detach():**
```c
int pthread_detach(pthread_t tid);

// Makes thread detached
// Status not retained after termination
// Cannot join with detached thread
```

**Joinable vs Detached:**

**Joinable (default):**
- Status retained until joined
- Another thread must call pthread_join()
- Like zombie if not joined

**Detached:**
- Status released immediately on termination
- Cannot join
- Useful for fire-and-forget threads

**Example:**
```c
pthread_t tid;
pthread_create(&tid, NULL, func, NULL);
pthread_detach(tid);  // Now detached

// This will fail:
pthread_join(tid, NULL);  // Returns error
```

**Creating Detached Thread:**
```c
pthread_attr_t attr;
pthread_attr_init(&attr);
pthread_attr_setdetachstate(&attr, PTHREAD_CREATE_DETACHED);
pthread_create(&tid, &attr, func, NULL);
pthread_attr_destroy(&attr);
```

### 38.7 Race Condition Example ⭐⭐⭐⭐⭐

**The Problem:**
```c
int myglobal = 0;

void *thread_func(void *arg) {
    int i, j;
    for (i = 0; i < 20; i++) {
        j = myglobal;    // Read
        j = j + 1;       // Increment
        myglobal = j;    // Write
        printf("t");
    }
    return NULL;
}

int main() {
    pthread_t tid;
    pthread_create(&tid, NULL, thread_func, NULL);
    
    for (int i = 0; i < 20; i++) {
        myglobal = myglobal + 1;
        printf("m");
        sleep(1);
    }
    
    pthread_join(tid, NULL);
    printf("Final: %d\n", myglobal);  // Should be 40
    return 0;
}
```

**Possible Outputs:**
```
Case 1: Thread executes immediately (no sleep)
ttttttttttttttttttttmmmmmmmmmmmmmmmmmmmm
Final: 40  ✓ (no race)

Case 2: Thread with sleep(1), alternating
mtmtmtmtmtmtmtmtmtmtmtmtmtmtmtmtmtmtmtmt
Final: 21  ✗ (lost updates!)

Case 3: Unpredictable interleaving
mttmttmttmttmttmttmttmttmttmttm
Final: 30  ✗ (some lost, some OK)
```

**Why Updates Lost:**
```
Time  Main              Thread
1     read global=0
2                       read global=0
3     global=1
4                       global=1  ← overwrites main's update!
5     read global=1
6                       read global=1
7     global=2
8                       global=2  ← lost again!
```

**Solution: Synchronization (next section)**

### 38.8 Precedence Graphs with Threads ⭐⭐⭐⭐

**Example:**
```
A
↓
B → D → G
↓       ↑
C → E ──┘
    ↓
    F
```

**Implementation:**
```c
void *thread_CF(void *arg) {
    printf("C\n");
    sleep(rand() % 5);
    printf("F\n");
    return NULL;
}

void *thread_E(void *arg) {
    printf("E\n");
    return NULL;
}

int main() {
    pthread_t t_cf, t_e;
    
    printf("A\n");
    sleep(rand() % 5);
    
    pthread_create(&t_cf, NULL, thread_CF, NULL);
    
    printf("B\n");
    sleep(rand() % 5);
    
    pthread_create(&t_e, NULL, thread_E, NULL);
    
    printf("D\n");
    
    pthread_join(t_e, NULL);   // Wait for E
    pthread_join(t_cf, NULL);  // Wait for C, F
    
    printf("G\n");
    return 0;
}
```

**Key Pattern:**
- Create threads for parallel branches
- Use pthread_join() for synchronization points
- Main thread continues for main path

### What to Memorize

**pthread_create():**
- [ ] 4 parameters: tid, attr, function, arg
- [ ] Returns 0 on success
- [ ] Function signature: void *(*)(void *)

**Passing Arguments:**
- [ ] NEVER pass &loopVar (race condition!)
- [ ] Use array of values
- [ ] Or cast integer as pointer
- [ ] Or struct for multiple values

**pthread_exit() vs exit():**
- [ ] pthread_exit(): terminates thread only
- [ ] exit(): terminates entire process
- [ ] Main should use pthread_exit() to let threads finish

**pthread_join():**
- [ ] Blocks until thread terminates
- [ ] Retrieves return value
- [ ] Like wait() for threads

**Detached Threads:**
- [ ] pthread_detach() or attribute
- [ ] Cannot join
- [ ] Status released immediately

---

## SECTION 39: Synchronization Concepts ⭐⭐⭐⭐⭐
**Slides 2-12 (Synchronization chapter) | Exam Relevance: CRITICAL**

### 39.1 Race Conditions ⭐⭐⭐⭐⭐

**Definition:**
Result depends on timing/order of concurrent operations.

**Classic Example: "Too Much Milk"**
```
Time    Person A              Person B
10:00   Look in fridge (no milk)
10:05   Leave for store
10:10   At store              Look in fridge (no milk)
10:15   Buy milk              Leave for store
10:20   Return home           At store
10:25   Put milk away         Buy milk
10:30                         Return home
10:35                         Put milk away ← TWO MILKS!
```

**Stack Example:**
```c
// Shared: int top = 0; int stack[SIZE];

// Thread i: push
stack[top] = val;
top++;

// Thread j: pop
top--;
*val = stack[top];
```

**Race Scenario:**
```
Thread i          Thread j
stack[0] = 5
                  top-- (top=0)
top++ (top=1)
                  *val = stack[0]  ← gets 5 instead of empty!
```

**Result:** Corrupted stack state!

### 39.2 Critical Sections ⭐⭐⭐⭐⭐

**Definition:**
- Section of code accessing shared resources
- Multiple processes/threads compete for access
- Must execute with mutual exclusion

**Example:**
```c
// Shared: int n = 0; int queue[SIZE];

void enqueue(int val) {
    queue[tail] = val;
    tail = (tail + 1) % SIZE;
    n++;  ← CRITICAL SECTION (shared variable n)
}

void dequeue(int *val) {
    *val = queue[head];
    head = (head + 1) % SIZE;
    n--;  ← CRITICAL SECTION (shared variable n)
}
```

**Problem:**
```c
// Assembly for n++:
register = n         // Read
register = register + 1
n = register        // Write

// Assembly for n--:
register = n        // Read
register = register - 1
n = register       // Write
```

**Race:**
```
Thread 1          Thread 2          n
reg1 = n (=5)                       5
                  reg2 = n (=5)     5
reg1 = 6
n = 6                               6
                  reg2 = 4
                  n = 4             4  ← Lost the increment!
```

### 39.3 Mutual Exclusion ⭐⭐⭐⭐⭐

**Goal:**
Only ONE thread in critical section at a time.

**Access Protocol:**
```c
Thread i                Thread j
while (TRUE) {          while (TRUE) {
    // Non-CS               // Non-CS
    
    entry_code();          entry_code();
    CRITICAL_SECTION;      CRITICAL_SECTION;
    exit_code();           exit_code();
    
    // Non-CS               // Non-CS
}                       }
```

**Required Properties:**

**1. Mutual Exclusion:**
- Only one thread in CS at a time
- **MANDATORY**

**2. Progress:**
- If no thread in CS, a waiting thread must enter in bounded time
- Only threads trying to enter can participate in selection
- No thread outside CS can block others
- Avoids **deadlock**

**3. Bounded Waiting:**
- Limit on how many times others can enter before a waiting thread
- Avoids **starvation**

**4. Symmetry:**
- Selection doesn't depend on relative speed or priority
- Fair scheduling

### 39.4 Solution Approaches ⭐⭐⭐

**Three Approaches:**

**1. Software Solutions:**
- Pure algorithmic logic
- No special hardware
- Based on shared variables
- Complex, limited to few threads

**2. Hardware Solutions:**
- Special CPU instructions
- Atomic operations (test-and-set, compare-and-swap)
- Simpler algorithms
- Still busy-waiting (spin locks)

**3. System Calls:**
- Kernel provides synchronization primitives
- **Semaphores** (Dijkstra, 1965)
- **Mutexes, Condition Variables**
- Most practical approach

### What to Memorize

**Race Condition:**
- [ ] Result depends on timing
- [ ] Concurrent access to shared data
- [ ] Non-atomic operations on shared variables

**Critical Section:**
- [ ] Code accessing shared resources
- [ ] Must have mutual exclusion
- [ ] Protect with entry/exit code

**Mutual Exclusion Requirements:**
- [ ] Only one in CS at a time (mandatory)
- [ ] Progress (no deadlock)
- [ ] Bounded waiting (no starvation)
- [ ] Symmetry (fairness)

**Solution Types:**
- [ ] Software (algorithmic)
- [ ] Hardware (atomic instructions)
- [ ] System calls (semaphores, mutexes)

---

## SECTION 40: Software Solutions ⭐⭐⭐
**Slides 2-8 (Software solutions chapter) | Exam Relevance: Medium**

### 40.1 Solution 1 - INCORRECT ⭐⭐⭐

**Approach:** Test then set flag

```c
int flag[2] = {FALSE, FALSE};

// Thread i
while (TRUE) {
    while (flag[j]);    // Wait while j busy
    flag[i] = TRUE;      // Set my flag
    // CRITICAL SECTION
    flag[i] = FALSE;     // Clear my flag
    // Non-CS
}

// Thread j
while (TRUE) {
    while (flag[i]);    // Wait while i busy
    flag[j] = TRUE;      // Set my flag
    // CRITICAL SECTION
    flag[j] = FALSE;     // Clear my flag
    // Non-CS
}
```

**Problem: Mutual Exclusion VIOLATED!**
```
Thread i          Thread j
while (flag[j])   
  FALSE ✓
                  while (flag[i])
                    FALSE ✓
flag[i] = TRUE
                  flag[j] = TRUE
Enter CS          Enter CS  ← BOTH IN CS!
```

**Analysis:**
- ✗ Mutual exclusion: FAILS
- ✓ Progress: OK
- ✓ Bounded waiting: OK
- ✓ Symmetry: OK

**Reason:** Test and set are NOT atomic!

### 40.2 Solution 2 - INCORRECT ⭐⭐⭐

**Approach:** Set then test flag

```c
int flag[2] = {FALSE, FALSE};

// Thread i
while (TRUE) {
    flag[i] = TRUE;      // Set my flag FIRST
    while (flag[j]);     // Wait while j busy
    // CRITICAL SECTION
    flag[i] = FALSE;
    // Non-CS
}

// Thread j
while (TRUE) {
    flag[j] = TRUE;      // Set my flag FIRST
    while (flag[i]);     // Wait while i busy
    // CRITICAL SECTION
    flag[j] = FALSE;
    // Non-CS
}
```

**Problem: DEADLOCK!**
```
Thread i          Thread j
flag[i] = TRUE
                  flag[j] = TRUE
while (flag[j])
  TRUE → wait
                  while (flag[i])
                    TRUE → wait
                  
← BOTH WAIT FOREVER! (Livelock)
```

**Analysis:**
- ✓ Mutual exclusion: OK
- ✗ Progress: FAILS (deadlock possible)
- ✗ Bounded waiting: FAILS
- ✓ Symmetry: OK

**Note:** Technically "livelock" (both busy-waiting) not deadlock (blocked), but same effect.

### 40.3 Spin Locks ⭐⭐

**Busy Waiting:**
- Thread continuously tests condition
- Wastes CPU cycles
- Called "spinning"

**Spin Lock:**
- Lock mechanism using busy waiting
- Acceptable ONLY if wait time very short
- Used in multiprocessor systems where thread on different CPU

**Problem:**
- Single CPU: spinning thread prevents lock holder from running!
- Inefficient use of resources

### What to Know (Brief)

- Solution 1: test-then-set (FAILS mutual exclusion)
- Solution 2: set-then-test (FAILS progress - deadlock)
- Spin locks: busy waiting (CPU waste)
- **Low-medium exam impact** - understand why simple solutions fail

---

## SUMMARY: Pages 601-800 Key Concepts

### CRITICAL MEMORIZATION CHECKLIST:

**Pthread Basics:**
- [ ] #include <pthread.h>, compile with -lpthread
- [ ] pthread_t tid (opaque type)
- [ ] pthread_self(), pthread_equal()

**pthread_create():**
- [ ] 4 params: tid, attr, function, arg
- [ ] Function: void *(*)(void *)
- [ ] Returns 0 on success
- [ ] NEVER pass &loopVar!

**Thread Termination:**
- [ ] pthread_exit() - thread only
- [ ] exit() - entire process
- [ ] return from function
- [ ] Main: pthread_exit(NULL) not exit(0)

**pthread_join():**
- [ ] Blocks until thread terminates
- [ ] Like wait() for threads
- [ ] Retrieves return value
- [ ] Can only join joinable threads

**Race Conditions:**
- [ ] Result depends on timing
- [ ] Non-atomic operations on shared data
- [ ] Lost updates common problem

**Critical Sections:**
- [ ] Code accessing shared resources
- [ ] Needs entry/exit protocol
- [ ] Mutual exclusion required

**Mutual Exclusion Properties:**
- [ ] Only one in CS (mandatory)
- [ ] Progress (no deadlock)
- [ ] Bounded waiting (no starvation)
- [ ] Symmetry (fairness)

### EXAM STRATEGY:

**Time Allocation:**
- Pthread questions: 5-7 minutes
- Race condition identification: 3-5 minutes
- Critical section analysis: 5-7 minutes
- Synchronization concepts: 2-3 minutes

**Common Traps:**
❌ Passing &loopVar to pthread_create
❌ Using exit(0) in main instead of pthread_exit(NULL)
❌ Forgetting to join threads (zombie threads)
❌ Not recognizing race conditions
❌ Confusing joinable vs detached
❌ Thinking test-then-set is atomic

**Pthread Problem Checklist:**
1. ✓ Argument passed correctly? (not &loopVar)
2. ✓ Return type void *?
3. ✓ pthread_exit() or return?
4. ✓ All threads joined?
5. ✓ Main uses pthread_exit(NULL)?

**Race Condition Checklist:**
1. ✓ Shared variable accessed?
2. ✓ Multiple threads?
3. ✓ Read-modify-write operations?
4. ✓ No synchronization?
→ RACE CONDITION!

---

## IMPORTANT NOTES:

**Pthread vs Process:**
```
fork()              pthread_create()
Parent/child        Equal threads
wait()              pthread_join()
PID (global)        pthread_t (local)
exit() kills all    pthread_exit() one thread
```

**Thread Safety:**
```
Thread-safe:        Not thread-safe:
  Local variables     Global variables
  Parameters          Static variables
  Const data          malloc/free (without sync)
  Reentrant funcs     printf (without sync)
```

**Typical Exam Patterns:**

**Pattern 1: Thread Creation**
"Create N threads, each printing its ID"
```c
for (long t = 0; t < N; t++) {
    pthread_create(&th[t], NULL, func, (void *)t);
}
for (long t = 0; t < N; t++) {
    pthread_join(th[t], NULL);
}
```

**Pattern 2: Race Condition**
"Does this code have a race condition?"
```c
// Shared: int counter = 0;
for (i = 0; i < 100; i++) {
    counter++;  // Thread A
}
for (i = 0; i < 100; i++) {
    counter--;  // Thread B
}
// Answer: YES! counter++ is not atomic
```

**Pattern 3: Precedence Graph**
"Implement with threads"
- Create thread for parallel branches
- pthread_join() at sync points
- Main thread for main path

---

**End of Pages 601-800 Study Guide**
**Total Coverage: Pages 1-800 Complete**

**🔥 With 800 pages covered, you have comprehensive coverage of OS fundamentals! 🔥**

**Key Achievement:**
- ✅ Processes (fork, exec, wait)
- ✅ Signals
- ✅ Pipes and IPC
- ✅ Threads (pthread)
- ✅ Synchronization concepts

**What's Typically Next (if exam covers more):**
- Mutexes and condition variables
- Semaphores (P/V operations)
- Classic synchronization problems
- Deadlock prevention
- Memory management
- File systems (detailed)
- CPU scheduling algorithms
