# Operating Systems - Exam-Oriented Study Guide
## Pages 801-1041: Advanced Synchronization ⭐⭐⭐⭐⭐

**CRITICAL EXAM MATERIAL - Heavily Tested Synchronization!**

**Previous Sections:** Pages 1-800 covered processes, threads, basic synchronization concepts

---

## ⚠️ EXAM IMPORTANCE ⚠️

**This section represents approximately 20-25% of exam content:**
- **Semaphores** ⭐⭐⭐⭐⭐ - **MOST TESTED** synchronization primitive
- **Producer-Consumer** ⭐⭐⭐⭐⭐ - Classic problem, always on exam
- **Readers-Writers** ⭐⭐⭐⭐ - Common exam pattern
- **Peterson's Algorithm** ⭐⭐⭐ - Software solution
- **pthread mutexes** ⭐⭐⭐⭐ - Practical implementation

**Semaphores and classic problems alone account for 25-30 exam questions!**

---

## SECTION 41: Peterson's Algorithm ⭐⭐⭐⭐
**Slides 10-19 | Exam Relevance: High**

### 41.1 Software Solutions Review ⭐⭐⭐

**Solution 1: Test-then-Set (FAILS)**
```c
int flag[2] = {FALSE, FALSE};

// Thread i
while (flag[j]);    // Test
flag[i] = TRUE;     // Set
CS
flag[i] = FALSE;
```
**Problem:** ✗ Mutual exclusion violated (both can enter)

**Solution 2: Set-then-Test (FAILS)**
```c
// Thread i
flag[i] = TRUE;     // Set
while (flag[j]);    // Test
CS
flag[i] = FALSE;
```
**Problem:** ✗ Deadlock (both can set and wait forever)

**Solution 3: Turn Variable (FAILS)**
```c
int turn = i;  // or j

// Thread i
while (turn != i);
CS
turn = j;
```
**Problem:** ✗ Starvation (forces strict alternation)

### 41.2 Peterson's Algorithm (CORRECT!) ⭐⭐⭐⭐⭐

**The First Correct Software Solution [Peterson, 1981]**

```c
int flag[2] = {FALSE, FALSE};
int turn = 0;  // or 1

// Thread i
while (TRUE) {
    flag[i] = TRUE;           // I want to enter
    turn = j;                 // Give priority to j
    while (flag[j] && turn == j);  // Wait if j wants AND has priority
    
    // CRITICAL SECTION
    
    flag[i] = FALSE;          // I'm done
    // Non-critical section
}

// Thread j
while (TRUE) {
    flag[j] = TRUE;           // I want to enter
    turn = i;                 // Give priority to i
    while (flag[i] && turn == i);  // Wait if i wants AND has priority
    
    // CRITICAL SECTION
    
    flag[j] = FALSE;          // I'm done
    // Non-critical section
}
```

### 41.3 Peterson's Algorithm Analysis ⭐⭐⭐⭐⭐

**Proof of Correctness:**

**1. Mutual Exclusion: ✓**
```
Ti in CS iff: flag[j] == FALSE OR turn == i
Tj in CS iff: flag[i] == FALSE OR turn == j

Both in CS requires:
  (flag[j] == FALSE OR turn == i) AND
  (flag[i] == FALSE OR turn == j)
  
But turn can only be i or j, not both!
Therefore, mutual exclusion guaranteed.
```

**2. Progress (No Deadlock): ✓**
```
If Ti waiting: flag[i] == TRUE AND flag[j] == TRUE AND turn == j

If Tj not interested (flag[j] == FALSE):
  → Ti enters immediately
  
If both waiting:
  → turn has single value
  → One will enter (whoever doesn't have turn)
  
No deadlock possible.
```

**3. Bounded Waiting (No Starvation): ✓**
```
If Tj fast and tries to re-enter:
  Tj sets flag[j] = TRUE
  Tj sets turn = i  ← Gives priority to Ti!
  → Ti can now enter
  
Even if Tj very fast, it always sets turn = i,
allowing Ti to proceed.
```

**4. Symmetry: ✓**
```
Code is symmetric for both threads.
```

### 41.4 Peterson's Limitations ⭐⭐⭐

**Disadvantages:**
- Still uses **busy waiting** (spin lock)
- Wastes CPU cycles
- Complex to extend beyond 2 threads
- Not practical for real systems

**Modern systems use semaphores/mutexes instead!**

### What to Memorize

**Peterson's Algorithm:**
- [ ] Combines flag[] and turn variable
- [ ] flag[i] = TRUE (I want to enter)
- [ ] turn = j (give priority to other)
- [ ] while (flag[j] && turn == j) (wait)
- [ ] Satisfies all 4 conditions

**Why Previous Solutions Failed:**
- [ ] Solution 1: test-then-set not atomic
- [ ] Solution 2: both set, then deadlock
- [ ] Solution 3: forces alternation (starvation)

---

## SECTION 42: Hardware Solutions ⭐⭐⭐
**Slides 2-19 | Exam Relevance: Medium**

### 42.1 Approaches ⭐⭐

**Three Hardware Approaches:**

**1. Disable Interrupts:**
```c
while (TRUE) {
    disable_interrupts();
    // CRITICAL SECTION
    enable_interrupts();
    // Non-CS
}
```
**Problems:**
- Only works for kernel
- Unsafe (what if never re-enable?)
- Multiprocessor: must disable ALL CPUs

**2. Test-and-Set Instruction:**
```c
char TestAndSet(char *lock) {
    char val = *lock;  // Read
    *lock = TRUE;      // Write
    return val;        // Return old value
}
// ATOMIC - executes in single memory cycle
```

**3. Swap Instruction:**
```c
void Swap(char *a, char *b) {
    char temp = *a;
    *a = *b;
    *b = temp;
}
// ATOMIC - executes in single memory cycle
```

### 42.2 Test-and-Set Usage ⭐⭐⭐

**Basic Mutual Exclusion:**
```c
char lock = FALSE;  // Shared

while (TRUE) {
    while (TestAndSet(&lock));  // Spin until lock == FALSE
    // CRITICAL SECTION
    lock = FALSE;               // Release lock
    // Non-CS
}
```

**How it Works:**
```
If lock == FALSE:
  TestAndSet returns FALSE (old value)
  Sets lock = TRUE
  → Exit while loop, enter CS
  
If lock == TRUE:
  TestAndSet returns TRUE (old value)
  Keeps lock = TRUE
  → Continue spinning
```

### 42.3 Hardware Solutions Summary ⭐⭐

**Advantages:**
- ✓ Works on multiprocessor
- ✓ Easy to extend to N threads
- ✓ Simple to use
- ✓ Symmetric

**Disadvantages:**
- ✗ Still **busy waiting** (wastes CPU)
- ✗ Possible **starvation** (arbitrary selection)
- ✗ **Priority inversion** problem

**Priority Inversion:**
```
High priority H: waiting for lock
Low priority L: holds lock
Medium priority M: preempts L
→ H blocked by M (medium priority), even though H > M!
```

### What to Know (Brief)

- Disable interrupts: kernel only
- Test-and-Set: atomic read-modify-write
- Still busy waiting (waste CPU)
- **Low-medium exam impact** - understand concept

---

## SECTION 43: Semaphores ⭐⭐⭐⭐⭐
**Slides 2-23 | Exam Relevance: MAXIMUM CRITICAL**

**THIS IS THE MOST PRACTICAL AND TESTED SYNCHRONIZATION TOPIC!**

### 43.1 Semaphore Definition ⭐⭐⭐⭐⭐

**What is a Semaphore:**
- Shared integer variable
- Protected by OS
- Atomic operations
- NO busy waiting (kernel manages queue)

**Structure:**
```c
typedef struct {
    char lock;        // Protects operations
    int cnt;          // Counter
    process_t *head;  // Waiting queue
} semaphore_t;
```

**Introduced by Dijkstra (1965)**

### 43.2 Semaphore Operations ⭐⭐⭐⭐⭐

**Four Operations:**

**1. init(S, k):**
```c
init(S, k);  // Initialize semaphore S to value k
```

**Two Types:**
- **Binary semaphore:** k ∈ {0, 1} (mutex lock)
- **Counting semaphore:** k ≥ 0

**2. wait(S) - Also called P(S):**
```c
wait(S) {
    if (S == 0)
        block();     // Kernel suspends thread
    else
        S--;
}
```
- Decrements counter
- Blocks if S ≤ 0
- "probeer te verlagen" (Dutch: try to decrease)

**3. signal(S) - Also called V(S):**
```c
signal(S) {
    if (blocked())
        wakeup();    // Kernel wakes a waiting thread
    else
        S++;
}
```
- Increments counter
- Wakes blocked thread if any
- "verhogen" (Dutch: to increment)

**4. destroy(S):**
```c
destroy(S);  // Free semaphore memory
```

**CRITICAL:** 
- **NOT** the same as wait() system call for processes!
- **NOT** the same as signal() for signal handlers!
- All operations are **ATOMIC** (OS guarantees)

### 43.3 Mutual Exclusion with Semaphores ⭐⭐⭐⭐⭐

**Basic Pattern:**
```c
semaphore_t S;
init(S, 1);  // Binary semaphore (mutex)

// Thread i
while (TRUE) {
    wait(S);          // P(S) - Lock
    // CRITICAL SECTION
    signal(S);        // V(S) - Unlock
    // Non-critical section
}

// Thread j
while (TRUE) {
    wait(S);          // P(S) - Lock
    // CRITICAL SECTION
    signal(S);        // V(S) - Unlock
    // Non-critical section
}
```

**Execution Trace:**
```
Time  S   T1          T2          T3
0     1   
1     0   wait(S) ✓   
2     0   [in CS]     wait(S) →   wait(S) →
                      blocked     blocked
3     1   signal(S)   
4     0               [in CS] ✓   blocked
5     1               signal(S)
6     0                           [in CS] ✓
7     1                           signal(S)
```

**Semaphore as Counter:**
- S > 0: Available resources
- S == 0: No resources, but no waiting
- S < 0: |S| = number of blocked threads

### 43.4 Allowing N Concurrent Threads ⭐⭐⭐⭐

**Pattern:**
```c
init(S, N);  // Allow N threads in CS

while (TRUE) {
    wait(S);
    // UP TO N THREADS HERE SIMULTANEOUSLY
    signal(S);
}
```

**Example with N=2:**
```
Time  S   T1        T2        T3
0     2   
1     1   wait ✓    
2     0   [in CS]   wait ✓    
3     -1  [in CS]   [in CS]   wait → blocked
4     0   signal    [in CS]   [in CS] ✓
5     1             signal    [in CS]
6     2                       signal
```

**Use Cases:**
- Database connections (limit concurrent users)
- Thread pool (limit active threads)
- Resource pool (limit resource usage)

### 43.5 Pure Synchronization ⭐⭐⭐⭐⭐

**Pattern 1: Enforce Order (A before B)**
```c
semaphore_t S;
init(S, 0);  // Start blocked!

// Thread i
A;            // Execute A
signal(S);    // Signal completion

// Thread j
wait(S);      // Wait for A to complete
B;            // Execute B
```

**Why init(S, 0):**
- Thread j calls wait(S)
- S == 0 → blocks!
- Thread i completes A, calls signal(S)
- S becomes 1, wakes thread j
- Thread j proceeds with B

**Pattern 2: Client-Server**
```c
semaphore_t S1, S2;
init(S1, 0);
init(S2, 0);

// Client (Ti)
while (TRUE) {
    prepare_data;
    signal(S1);    // Data ready
    wait(S2);      // Wait for result
    get_result;
}

// Server (Tj)
while (TRUE) {
    wait(S1);      // Wait for data
    process_data;
    signal(S2);    // Result ready
}
```

**Pattern 3: Precedence Graph**
```
A
↓
B → C
```

**Implementation:**
```c
semaphore_t S;
init(S, 0);

// Thread i
A;
signal(S);

// Thread j  
wait(S);
B;
signal(S);  // If C needs B

// Thread k
wait(S);
C;
```

**Pattern 4: Fork-Join (cobegin-coend)**
```
     P0
    / | \
   P1 P2 P3 ... Pn
    \ | /
     Pn+1
```

**Implementation:**
```c
semaphore_t S1, S2;
init(S1, 0);
init(S2, 0);

// P0
for (i = 1; i <= n; i++)
    signal(S1);  // Release all Pi

// Pi (for i = 1 to n)
wait(S1);        // Wait for P0
// Do work
signal(S2);      // Signal completion

// Pn+1
for (i = 1; i <= n; i++)
    wait(S2);    // Wait for all Pi
// Continue
```

### 43.6 Common Semaphore Errors ⭐⭐⭐⭐⭐

**Error 1: signal() before wait()**
```c
init(S, 1);

// Thread 1
signal(S);  // ✗ WRONG ORDER!
CS
wait(S);
```
**Problem:** CS not protected! S becomes 2, multiple threads can enter.

**Error 2: wait() twice**
```c
// Thread 1
wait(S);
CS
wait(S);  // ✗ DEADLOCK!
```
**Problem:** Thread 1 blocks itself. All threads deadlock.

**Error 3: signal() twice**
```c
// Thread 1
wait(S);
CS
signal(S);
signal(S);  // ✗ WRONG!
```
**Problem:** S becomes 2, breaks mutual exclusion.

**Error 4: Forgetting signal()**
```c
wait(S);
CS
// ✗ MISSING signal(S)!
```
**Problem:** Lock never released. All other threads block forever.

### What to Memorize

**Semaphore Basics:**
- [ ] Integer counter + queue
- [ ] init(S, k) - initialize
- [ ] wait(S) - decrement, block if 0
- [ ] signal(S) - increment, wake if blocked
- [ ] All operations ATOMIC

**Mutual Exclusion Pattern:**
- [ ] init(S, 1) for binary semaphore
- [ ] wait(S) before CS
- [ ] signal(S) after CS
- [ ] init(S, N) for N concurrent threads

**Synchronization Pattern:**
- [ ] init(S, 0) to enforce order
- [ ] Thread A: do work, signal(S)
- [ ] Thread B: wait(S), do work

**Common Errors:**
- [ ] signal before wait (no protection)
- [ ] wait twice (deadlock)
- [ ] signal twice (breaks exclusion)
- [ ] Forget signal (permanent block)

---

## SECTION 44: pthread Mutexes ⭐⭐⭐⭐
**Slides 55-64 | Exam Relevance: High**

### 44.1 pthread Mutex Functions ⭐⭐⭐⭐

**Mutex = Mutual Exclusion Lock**

**Declaration and Initialization:**
```c
#include <pthread.h>

pthread_mutex_t mutex;

// Method 1: Static
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;

// Method 2: Dynamic
pthread_mutex_init(&mutex, NULL);
```

**Lock and Unlock:**
```c
pthread_mutex_lock(&mutex);      // Blocks if locked
// CRITICAL SECTION
pthread_mutex_unlock(&mutex);

// Non-blocking version:
if (pthread_mutex_trylock(&mutex) == 0) {
    // Got lock!
    // CS
    pthread_mutex_unlock(&mutex);
} else {
    // Lock busy (EBUSY)
}
```

**Destroy:**
```c
pthread_mutex_destroy(&mutex);
```

### 44.2 Mutex Example ⭐⭐⭐⭐

**Protecting Shared Counter:**
```c
int counter = 0;
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;

void *thread_func(void *arg) {
    for (int i = 0; i < 1000; i++) {
        pthread_mutex_lock(&mutex);
        counter++;  // Protected!
        pthread_mutex_unlock(&mutex);
    }
    return NULL;
}

int main() {
    pthread_t threads[10];
    
    for (int i = 0; i < 10; i++)
        pthread_create(&threads[i], NULL, thread_func, NULL);
    
    for (int i = 0; i < 10; i++)
        pthread_join(threads[i], NULL);
    
    printf("Counter: %d\n", counter);  // Should be 10000
    return 0;
}
```

### 44.3 Mutex vs Semaphore ⭐⭐⭐

**Mutex (pthread_mutex_t):**
- Binary lock (locked/unlocked)
- **Ownership:** Only locker can unlock
- For mutual exclusion only
- Simpler, faster
- pthread-specific

**Semaphore (semaphore_t):**
- Counter (can be > 1)
- **No ownership:** Any thread can signal
- For both mutual exclusion AND synchronization
- More flexible
- POSIX standard

**When to Use:**
- **Mutex:** Protecting shared data
- **Semaphore:** Synchronization, resource counting

### What to Memorize

**pthread Mutex:**
- [ ] pthread_mutex_init(&m, NULL)
- [ ] pthread_mutex_lock(&m) - blocks
- [ ] pthread_mutex_trylock(&m) - non-blocking
- [ ] pthread_mutex_unlock(&m)
- [ ] pthread_mutex_destroy(&m)

**Mutex vs Semaphore:**
- [ ] Mutex: binary, ownership
- [ ] Semaphore: counter, no ownership
- [ ] Mutex: protection only
- [ ] Semaphore: sync + protection

---

## SECTION 45: Producer-Consumer ⭐⭐⭐⭐⭐
**Slides 2-9 | Exam Relevance: MAXIMUM CRITICAL**

**ALWAYS ON EXAM! Must know perfectly!**

### 45.1 Problem Statement ⭐⭐⭐⭐⭐

**Scenario:**
- Producer(s) generate data
- Consumer(s) process data
- Shared bounded buffer (circular queue)
- Buffer size = SIZE

**Constraints:**
1. Producer must wait if buffer full
2. Consumer must wait if buffer empty
3. Mutual exclusion on buffer access

### 45.2 Solution 1: Single Producer, Single Consumer ⭐⭐⭐⭐⭐

**Semaphores:**
```c
semaphore_t empty;  // Count empty slots
semaphore_t full;   // Count filled slots

init(empty, SIZE);  // Initially all empty
init(full, 0);      // Initially none full
```

**Code:**
```c
#define SIZE 10
int queue[SIZE];
int head = 0, tail = 0;

void enqueue(int val) {
    queue[tail] = val;
    tail = (tail + 1) % SIZE;
}

int dequeue() {
    int val = queue[head];
    head = (head + 1) % SIZE;
    return val;
}

void producer() {
    int val;
    while (TRUE) {
        produce(&val);     // Generate data
        
        wait(empty);       // Wait for empty slot
        enqueue(val);      // Add to buffer
        signal(full);      // Signal data available
    }
}

void consumer() {
    int val;
    while (TRUE) {
        wait(full);        // Wait for data
        val = dequeue();   // Remove from buffer
        signal(empty);     // Signal slot available
        
        consume(val);      // Process data
    }
}
```

**Why This Works:**
- `empty` tracks free space (starts at SIZE)
- `full` tracks available data (starts at 0)
- Producer: wait(empty), signal(full)
- Consumer: wait(full), signal(empty)
- Symmetric!
- No explicit mutex needed (single P, single C)

### 45.3 Solution 2: Multiple Producers, Multiple Consumers ⭐⭐⭐⭐⭐

**Problem with Solution 1:**
- Two producers can call enqueue() simultaneously
- Two consumers can call dequeue() simultaneously
- Race condition on head/tail!

**Solution: Add Mutexes**
```c
semaphore_t empty, full;
semaphore_t MEp, MEc;  // Mutual exclusion

init(empty, SIZE);
init(full, 0);
init(MEp, 1);   // Mutex for producers
init(MEc, 1);   // Mutex for consumers

void producer() {
    int val;
    while (TRUE) {
        produce(&val);
        
        wait(empty);      // Wait for space
        wait(MEp);        // Lock enqueue
        enqueue(val);
        signal(MEp);      // Unlock enqueue
        signal(full);     // Signal data
    }
}

void consumer() {
    int val;
    while (TRUE) {
        wait(full);       // Wait for data
        wait(MEc);        // Lock dequeue
        val = dequeue();
        signal(MEc);      // Unlock dequeue
        signal(empty);    // Signal space
        
        consume(val);
    }
}
```

**CRITICAL ORDER:**
```c
// CORRECT:
wait(empty);     // First: check space
wait(MEp);       // Then: lock

// WRONG (DEADLOCK!):
wait(MEp);       // Lock first
wait(empty);     // Then check - DEADLOCK if buffer full!
```

**Why Order Matters:**
```
If wait(MEp) first:
  Producer 1: wait(MEp) ✓, buffer full, wait(empty) → blocks with lock!
  Producer 2: wait(MEp) → blocked
  Consumer: can't run (needs data) → DEADLOCK!
```

### 45.4 Producer-Consumer Variations ⭐⭐⭐⭐

**Variation 1: Unbounded Buffer**
```c
semaphore_t full;
semaphore_t ME;

init(full, 0);
init(ME, 1);
// No 'empty' semaphore! (unlimited space)

void producer() {
    wait(ME);
    enqueue(val);
    signal(ME);
    signal(full);
}

void consumer() {
    wait(full);
    wait(ME);
    val = dequeue();
    signal(ME);
}
```

**Variation 2: Multiple Buffers**
```c
// Different buffer for each producer-consumer pair
// Each pair has own empty/full semaphores
```

### What to Memorize

**Single P/C:**
- [ ] init(empty, SIZE), init(full, 0)
- [ ] Producer: wait(empty), enqueue, signal(full)
- [ ] Consumer: wait(full), dequeue, signal(empty)
- [ ] No mutex needed

**Multiple P/C:**
- [ ] Add MEp and MEc mutexes
- [ ] Producer: wait(empty), wait(MEp), enqueue, signal(MEp), signal(full)
- [ ] Consumer: wait(full), wait(MEc), dequeue, signal(MEc), signal(empty)
- [ ] ORDER CRITICAL: space/data check BEFORE mutex!

**Common Errors:**
- [ ] Mutex before space check (deadlock!)
- [ ] Forgetting signal (blocked forever)
- [ ] Wrong semaphore order

---

## SECTION 46: Readers-Writers ⭐⭐⭐⭐
**Slides 10-18 | Exam Relevance: High**

### 46.1 Problem Statement ⭐⭐⭐⭐

**Scenario:**
- Multiple readers can read simultaneously
- Writers need exclusive access (no readers, no other writers)
- Shared database/file

**Two Variants:**
1. **Readers priority:** Readers never wait if another reader active
2. **Writers priority:** Writers get priority over readers

### 46.2 Readers Priority Solution ⭐⭐⭐⭐⭐

**Semaphores:**
```c
semaphore_t w;     // Writer lock
semaphore_t meR;   // Protects nR counter
int nR = 0;        // Number of active readers
```

**Initialization:**
```c
init(w, 1);
init(meR, 1);
nR = 0;
```

**Reader Code:**
```c
void reader() {
    // Entry protocol
    wait(meR);           // Lock nR
    nR++;
    if (nR == 1)
        wait(w);         // First reader locks writers
    signal(meR);         // Unlock nR
    
    // READING (multiple readers here!)
    
    // Exit protocol
    wait(meR);           // Lock nR
    nR--;
    if (nR == 0)
        signal(w);       // Last reader unlocks writers
    signal(meR);         // Unlock nR
}
```

**Writer Code:**
```c
void writer() {
    wait(w);            // Exclusive access
    
    // WRITING (only one writer!)
    
    signal(w);          // Release
}
```

**How It Works:**
```
Execution:
1. Reader 1: nR=0→1, wait(w) ✓, locks writers
2. Reader 2: nR=1→2, doesn't wait(w), enters
3. Reader 3: nR=2→3, enters
4. Writer: wait(w) → BLOCKED (readers active)
5. Reader 1 exits: nR=3→2
6. Reader 2 exits: nR=2→1
7. Reader 3 exits: nR=1→0, signal(w) → wakes writer
8. Writer: gets w, writes, signal(w)
```

**First Reader In, Last Reader Out:**
- **First reader** (nR: 0→1): locks writers with wait(w)
- **Middle readers** (nR > 1): just increment nR
- **Last reader** (nR: 1→0): unlocks writers with signal(w)

### 46.3 Readers Priority Analysis ⭐⭐⭐⭐

**Advantages:**
- ✓ Maximizes concurrency for reads
- ✓ Good for read-heavy workloads

**Disadvantages:**
- ✗ **Writers can starve!**
- If readers keep arriving, writers wait forever

**Starvation Scenario:**
```
Time  Readers  Writers
0     R1       W1 waiting
1     R1, R2   W1 waiting
2     R1, R2, R3   W1 waiting
3     R2, R3, R4   W1 waiting  ← R1 left, R4 arrived
4     R3, R4, R5   W1 waiting  ← R2 left, R5 arrived
...
∞     Rn       W1 STILL WAITING!
```

### 46.4 Alternative: Writers Priority ⭐⭐⭐

**Concept:**
- Once a writer waiting, no new readers start
- Prevents writer starvation
- Readers can starve instead

**Implementation** (more complex, less commonly tested):
```c
// Additional semaphores/variables needed
semaphore_t r, w, meR, meW;
int nR = 0, nW = 0, nWait = 0;

// Reader waits if writer waiting or writing
// Writer gets priority
```

### What to Memorize

**Readers Priority:**
- [ ] Semaphores: w (writer lock), meR (protect nR)
- [ ] Variable: nR (reader count)
- [ ] Reader entry: nR++, if (nR==1) wait(w)
- [ ] Reader exit: nR--, if (nR==0) signal(w)
- [ ] Writer: wait(w), write, signal(w)

**Key Insight:**
- [ ] First reader locks writers
- [ ] Last reader unlocks writers
- [ ] Writers can starve

**Pattern:**
- [ ] meR protects nR counter
- [ ] w provides mutual exclusion for writers
- [ ] Multiple readers allowed simultaneously

---

## SUMMARY: Pages 801-1041 Key Concepts

### CRITICAL MEMORIZATION CHECKLIST:

**Peterson's Algorithm:**
- [ ] flag[2] and turn variables
- [ ] flag[i] = TRUE, turn = j
- [ ] while (flag[j] && turn == j)
- [ ] Satisfies all 4 conditions
- [ ] Still busy waiting

**Semaphores (MOST IMPORTANT!):**
- [ ] init(S, k) - initialize
- [ ] wait(S) - P(S) - decrement, block if 0
- [ ] signal(S) - V(S) - increment, wake
- [ ] All atomic, no busy waiting

**Mutual Exclusion Pattern:**
- [ ] init(S, 1)
- [ ] wait(S), CS, signal(S)

**Synchronization Pattern:**
- [ ] init(S, 0) - start blocked
- [ ] Thread A: work, signal(S)
- [ ] Thread B: wait(S), work

**Producer-Consumer (CRITICAL!):**
- [ ] Single P/C: empty, full semaphores
- [ ] Multiple P/C: add MEp, MEc
- [ ] ORDER: space/data check BEFORE mutex!
- [ ] Producer: wait(empty), wait(MEp), enqueue, signal(MEp), signal(full)

**Readers-Writers:**
- [ ] Readers priority: w, meR, nR
- [ ] First reader: wait(w)
- [ ] Last reader: signal(w)
- [ ] Writer: wait(w), write, signal(w)

**pthread Mutexes:**
- [ ] pthread_mutex_lock(&m)
- [ ] CS
- [ ] pthread_mutex_unlock(&m)

### EXAM STRATEGY:

**Time Allocation:**
- Producer-Consumer: 10-15 minutes (most complex)
- Semaphore problems: 5-10 minutes each
- Readers-Writers: 7-10 minutes
- Peterson's: 3-5 minutes (analysis)

**Common Traps:**
❌ Producer-Consumer: mutex before space check (DEADLOCK!)
❌ Semaphores: signal before wait (no protection)
❌ wait() twice (deadlock)
❌ Forgetting signal() (permanent block)
❌ Readers-Writers: forgetting first/last reader checks
❌ Confusing wait(S) with wait() for processes
❌ Confusing signal(S) with signal() for handlers

**Problem-Solving Checklist:**
1. ✓ Identify what needs protection (CS)
2. ✓ Identify synchronization requirements (ordering)
3. ✓ Choose semaphores (binary vs counting)
4. ✓ Initialize correctly (0 for sync, 1 for mutex, N for resources)
5. ✓ Check wait/signal order carefully
6. ✓ Verify no deadlock (no circular wait)

---

## EXAM PATTERNS:

**Pattern 1: "Implement with semaphores"**
Given precedence graph or synchronization requirement, implement with semaphores.
- Identify dependencies
- Use init(S, 0) for ordering
- Use init(S, 1) for mutual exclusion

**Pattern 2: "Find the error"**
Code with semaphores, find bug.
- Check signal before wait
- Check wait twice
- Check missing signal
- Check order in producer-consumer

**Pattern 3: "Producer-Consumer variant"**
Modify producer-consumer for specific scenario.
- Identify if single or multiple P/C
- Add mutexes if multiple
- Verify order

**Pattern 4: "Will this deadlock?"**
Analyze code for deadlock.
- Check circular wait
- Check hold-and-wait
- Check resource ordering

---

**End of Pages 801-1041 Study Guide**
**Total Coverage: Pages 1-1041 Complete**

**🔥 With 1041 pages covered, you have mastered synchronization! 🔥**

**Complete Achievement:**
- ✅ Processes (fork, exec, wait, signals)
- ✅ Pipes and IPC
- ✅ Threads (pthread)
- ✅ **Semaphores and classic problems** ← YOU ARE HERE
- ✅ Mutexes

**This completes the core synchronization material that appears on virtually every OS exam!**
