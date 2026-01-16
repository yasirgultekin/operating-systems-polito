# Operating Systems - Exam-Oriented Study Guide
## Pages 201-300: find, Filters, File I/O, and Directories

**Previous Sections:** Pages 1-200 covered introduction, Linux commands, C tools, and regex

---

## SECTION 14: find Command ⭐⭐⭐⭐⭐
**Slides 13-29 | Exam Relevance: CRITICAL (heavily tested with regex!)**

### 14.1 find Basics ⭐⭐⭐⭐⭐

**Purpose:**
- Search and list files, directories, or links matching criteria
- Execute shell commands on each matching file

**Format:**
```bash
find directory options actions
```

**Key Characteristics:**
- Outputs **relative path** (not basename)
- Important for writing regex and actions
- Visits entire directory subtree

### 14.2 find Options ⭐⭐⭐⭐⭐

**Name-based Matching:**
```bash
-name pattern          # Match filename (case-sensitive)
-iname pattern         # Case-insensitive name match
-path pattern          # Match path+filename
-ipath pattern         # Case-insensitive path match
```

**Regex Matching (EXAM CRITICAL!):**
```bash
-regex expr            # Regular expression on full relative path
-iregex expr           # Case-insensitive regex
-regextype type        # Specify regex type (posix-extended, etc.)
                       # MUST come BEFORE -regex!
```

**Type and Size:**
```bash
-type f                # Regular file
-type d                # Directory
-type l                # Symbolic link
-type p                # Named pipe
-type s                # Socket

-size +500c            # Size > 500 bytes
-size -1M              # Size < 1 MB
-size 10k              # Size exactly 10 KB
                       # b=blocks(512B), c=bytes, k=KB, M=MB, G=GB
```

**Time-based:**
```bash
-atime n               # Last access time
-mtime n               # Last modification time
-ctime n               # Last status change time
                       # n=1 means 0-24 hours ago
                       # +n means ≥n, -n means ≤n
```

**Depth Control:**
```bash
-mindepth n            # Minimum depth
-maxdepth n            # Maximum depth
                       # -maxdepth 1 = current directory only
```

**Permissions and Ownership:**
```bash
-user name             # File owner
-group name            # File group
-readable              # Readable files
-writable              # Writable files
-executable            # Executable files
```

**Other:**
```bash
-quit                  # Exit after first match
```

### 14.3 find Actions ⭐⭐⭐⭐⭐

**Default Action:**
```bash
-print                 # Print matching filenames (default)
-print0                # Print without newline
-fprint file           # Print to file
```

**Execute Commands (HEAVILY TESTED!):**
```bash
-exec command \{\} \;      # Execute command on each match
-exec command '{}' ';'     # Alternative syntax
-execdir command \{\} \;   # Execute in file's directory
-delete                    # Delete matching files
```

**CRITICAL exec Syntax:**
- `\{\}` or `'{}'` = placeholder for current filename
- `\;` or `';'` = terminates the -exec command
- **exec**: runs in current directory
- **execdir**: runs in file's directory

### 14.4 find Examples ⭐⭐⭐⭐⭐

**Example 1: Basic Name Search**
```bash
find . -name "*.c"                    # All .c files
find . -regex ".*\.c"                 # Same with regex (must escape .)
```

**Example 2: Case-Insensitive**
```bash
find /usr/bin -iname "a.*"            # Files starting with a or A
```

**Example 3: Size Filter**
```bash
find . -size +500c                    # Files > 500 bytes
```

**Example 4: Complex Regex with Extended RE**
```bash
find . -readable -regex "\./a+b.*\..*"
# Readable files in current dir with name: a+b...extension
```

**Example 5: Palindrome (EXAM PATTERN!)**
```bash
# Basic RE (requires escaping)
find /usr/bin -regex ".*\/..\(.\)\(.\).\2\1.*"

# Extended RE (cleaner)
find /usr/bin -regextype posix-extended \
              -regex ".*\/..(.)(.).\2\1.*"

# Matches files with 5-char palindrome after 2 chars
# Example: /usr/bin/abcivic123
```

**Example 6: Depth Control**
```bash
find /home/usr/ -mindepth 2 -maxdepth 4 \
                -name "*.exe"
# Search levels 2-4 only
```

**Example 7: Execute Command**
```bash
# Delete all .old files
find . -name "*.old" -type f -exec rm -f \{\} \;

# Add execute permission
find /home/usr/ -mindepth 2 -maxdepth 2 \
                -name "*.exe" -type f \
                -exec chmod +x \{\} \;

# View first 2 lines of each .txt file
find . -name "*.txt" -exec head -n 2 \{\} \;

# Concatenate all files owned by root
find / -user root -exec cat \{\} \;

# Append to file
find / -user root -exec cat '{}' >> file.txt ';'
```

### 14.5 find with Pipes ⭐⭐⭐⭐

**Understanding the Difference:**

**Piping find output:**
```bash
find . -name "*.txt" | wc
# wc counts the LIST of files (number of filenames)
# Output: number of matching files
```

**Using exec:**
```bash
find . -name "*.txt" -exec wc \{\} \;
# wc processes CONTENT of each file
# Output: line/word/char count for each file
```

**Combined Example:**
```bash
find . -name "*.txt" -exec wc \{\} \; | cut -f 3 -d " "
# Get character count from each file's wc output
```

### What to Memorize

**find Syntax:**
- [ ] `find directory options actions`
- [ ] `-regextype` MUST come before `-regex`
- [ ] `\{\}` = placeholder for filename
- [ ] `\;` = terminates exec command

**Common Patterns:**
- [ ] `-name "*.ext"` for simple search
- [ ] `-regex "pattern"` for complex
- [ ] `-type f` for files, `-type d` for directories
- [ ] `-mindepth` and `-maxdepth` for depth control
- [ ] `-exec command \{\} \;` for execution

**Regex in find:**
- [ ] Matches full relative path
- [ ] Must escape special chars: `\.` for dot
- [ ] Use posix-extended for cleaner syntax

### Exam Patterns

**HEAVILY TESTED!**

**Pattern 1: Regex Matching**
"Which files match this find command?"
```bash
find . -regextype posix-extended \
       -regex "\./(2[468]|[3-5][02468])(xx|yy)*\.(c|java)"
```

**Strategy:**
1. Break down regex parts
2. `\./` = starts with ./
3. `(2[468]|[3-5][02468])` = 24,26,28,30,...,68
4. `(xx|yy)*` = zero or more xx or yy
5. `\.(c|java)` = ends with .c or .java

**Example files:**
- `./24.c` ✓
- `./26xx.java` ✓
- `./35.c` ✗ (35 is odd)
- `./48xxyy.c` ✓

**Pattern 2: exec vs Pipe**
"Count characters in all .txt files"
```bash
# Wrong (counts number of files):
find . -name "*.txt" | wc -c

# Correct (counts characters in all files):
find . -name "*.txt" -exec wc -c \{\} \;
```

**Common Mistakes:**
- Forgetting to escape regex metacharacters
- Not using -regextype before -regex
- Confusing -exec behavior with pipe
- Missing `\;` terminator in -exec

---

## SECTION 15: Filter Commands ⭐⭐⭐⭐
**Slides 2-33 (Filters chapter) | Exam Relevance: High**

### 15.1 Filter Concept ⭐⭐⭐

**Definition:**
- Takes input from stdin
- Processes/filters according to parameters
- Produces output to stdout
- Useful for text file processing

**Common Filters:**
```bash
awk, cat, cut, grep, head, tail, sort, uniq, tr, wc
```

### 15.2 cut Command ⭐⭐⭐

**Purpose:** Select sections from each line

**Syntax:**
```bash
cut [options] file
```

**Options:**
```bash
-c LIST                # Select characters
-f LIST                # Select fields (default delimiter: TAB)
-d DELIM               # Specify delimiter

# LIST format: n (field n), -n (≤n), n- (≥n), n1-n2 (range)
```

**Examples:**
```bash
cut -f 1,3 file.txt              # Fields 1 and 3
cut -f 1-3,5-6 -d " " file.txt   # Fields 1-3 and 5-6, space-delimited
cut -c 1-10 file.txt             # First 10 characters
```

**With find:**
```bash
find . -name "*.txt" -exec wc \{\} \; | cut -f 3 -d " "
# Extract 3rd field (character count) from wc output
```

### 15.3 tr Command ⭐⭐⭐

**Purpose:** Translate, compress, or delete characters

**Syntax:**
```bash
tr [options] set1 [set2]
```

**Must be used with input redirection or pipe!**

**Options:**
```bash
-d              # Delete characters in set1
-s              # Squeeze repeated characters
-c              # Complement of set1
```

**Special characters:**
```bash
\n              # Newline
\\              # Backslash
\num            # ASCII code num
```

**Examples:**
```bash
tr -d abcd < file.txt                # Delete a,b,c,d
cat file.txt | tr ab BA              # a→B, b→A
echo ciao | tr ia IA                 # cIAo
echo "a b  c" | tr -s ' '            # Squeeze spaces: "a b c"
```

### 15.4 uniq Command ⭐⭐

**Purpose:** Report or omit repeated lines

**Syntax:**
```bash
uniq [options] [inFile] [outFile]
```

**IMPORTANT:** File must be sorted first!

**Options:**
```bash
-c              # Prefix with occurrence count
-d              # Only print duplicates
-f N            # Skip first N fields
-i              # Case insensitive
```

**Examples:**
```bash
uniq -c file.txt                     # Count occurrences
uniq -d file.txt                     # Show only duplicates

# Typical usage:
sort file.txt | uniq                 # Remove adjacent duplicates
sort file.txt | uniq -c              # Count unique lines
```

### 15.5 basename Command ⭐⭐

**Purpose:** Strip directory path and optionally extension

**Syntax:**
```bash
basename pathname [extension]
```

**Examples:**
```bash
basename /home/user/file.txt         # Output: file.txt
basename /home/user/file.txt ".txt"  # Output: file
basename /home/user/file.txt .txt    # Output: file (quotes optional)
basename /home/user/file.txt txt     # Output: file. (removes only "txt")
```

### 15.6 sort Command ⭐⭐⭐⭐

**Purpose:** Sort lines in files

**Syntax:**
```bash
sort [options] [file]
```

**Common Options:**
```bash
-n              # Numeric sort
-r              # Reverse order
-k c1[,c2]      # Sort by key (field)
-f              # Ignore case
-b              # Ignore leading blanks
-d              # Dictionary order (only spaces and alphanumeric)
-m              # Merge sorted files
-o file         # Output to file
```

**Examples:**
```bash
sort file.txt                        # Alphabetic sort
sort -r file.txt                     # Reverse order
sort -n file.txt                     # Numeric sort

# Sort by field:
ls -la | sort -n -k 5                # Sort by size (field 5)
cat file1.txt file2.txt | sort -r -k 1,3 -f
                                     # Reverse, fields 1-3, case-insensitive
```

**Practical Example (EXAM PATTERN):**
```bash
# Sort files by size:
ls -la | sort -n -k 5
# Note: "total", ".", ".." lines remain at start
```

### 15.7 grep Command ⭐⭐⭐⭐⭐

**CRITICAL FOR EXAM!**

**Purpose:** Global Regular Expression Print
- Search files for lines matching pattern
- Print matching lines

**Syntax:**
```bash
grep [options] pattern [file]
```

**Versions:**
```bash
grep            # Standard
egrep           # Extended RE (same as grep -E)
grep -E         # Use Extended Regular Expressions
```

**Common Options:**
```bash
-e PATTERN      # Specify pattern (allows multiple)
-n              # Show line numbers
-r, -R          # Recursive search
-v              # Inverse match (non-matching lines)
-i              # Case insensitive
-H              # Show filename
-A N            # Show N lines after match
-B N            # Show N lines before match
```

**Examples:**
```bash
grep abc file.txt                    # Lines containing "abc"
grep -n abc file.txt                 # With line numbers
grep -v abc file.txt                 # Lines NOT containing "abc"

# Multiple patterns:
grep -e "l." -e a file.txt          # Lines with 'l' + any char, OR 'a'

# Context lines:
grep -H -A 4 abc file.txt           # Match + 4 lines after, with filename

# Regex (use Extended RE for clarity):
grep -E "(.).\1" *.txt              # 3-char palindrome
grep -E "(.)(.).\2\1" *.txt         # 5-char palindrome
```

**With find:**
```bash
find /home/foo -name "L*.txt" -exec grep -H "laib" '{}' \;
# Search for "laib" in files starting with L
```

**Filtering ls output:**
```bash
ls -la | grep -v -e "total" -e "\.$" -e "\.\.$"
# Remove "total" line and . and .. directories
```

### What to Memorize

**cut:**
- [ ] `-f` for fields, `-c` for characters
- [ ] `-d` to change delimiter from TAB

**tr:**
- [ ] Must use with `<` or `|`
- [ ] `-d` delete, `-s` squeeze

**sort:**
- [ ] `-n` numeric, `-r` reverse
- [ ] `-k` for field-based sorting

**grep:**
- [ ] `-v` inverse match
- [ ] `-E` for Extended RE
- [ ] `-A/-B` for context lines
- [ ] Backward refs: `\1`, `\2` (Basic) or `$1`, `$2` (Extended)

### Exam Patterns

**Pattern 1: Pipeline Combinations**
```bash
# Count unique words in file:
tr -s ' ' '\n' < file.txt | sort | uniq -c

# Find palindromes in .txt files:
grep -E "(.)(.).\2\1" *.txt
```

**Pattern 2: ls Filtering and Sorting**
```bash
# Sort by size, remove directory entries:
ls -la | grep -v -e "total" -e "\.$" -e "\.\.$" | sort -n -k 5
```

**Pattern 3: find + grep**
```bash
# Find files and search within them:
find . -name "*.c" -exec grep -H "main" '{}' \;
```

---

## SECTION 16: File System and File I/O ⭐⭐⭐⭐⭐
**Slides 2-39 (Files in Linux chapter) | Exam Relevance: CRITICAL**

### 16.1 File Encoding ⭐⭐

**ASCII Encoding:**
- 128 characters (7-bit)
- 32 non-printable, 96 printable
- Based on English alphabet

**Extended ASCII:**
- 255 characters (8-bit)
- ISO 8859-1 (Latin-1), ISO 8859-2 (Eastern European), etc.

**Unicode:**
- 110,000+ characters
- UTF-8: 1-4 bytes per character (ASCII in first 8 bits)
- UTF-16: 1-2 groups of 16 bits
- UTF-32: Fixed 32 bits

**Exam Relevance:** Minimal (background only)

### 16.2 Text vs Binary Files ⭐⭐⭐

**Textual Files:**
- ASCII/Unicode encoded
- Line-oriented
- Newline characters:
  - UNIX/Linux/macOS: LF (Line Feed, `\n`, 0x0A)
  - Windows: CR+LF (Carriage Return + Line Feed, `\r\n`, 0x0D 0x0A)

**Binary Files:**
- Sequence of arbitrary bytes
- Not necessarily printable characters
- More compact
- Fixed record structures

**Advantages of Binary:**
- Compactness (e.g., number 10000010: 6 bytes as text, 4 as int)
- Fixed size for same data type
- Easy positioning with fixed records

**Disadvantages of Binary:**
- Limited portability
- Cannot use standard text editor
- Architecture-dependent (endianness, alignment)

**UNIX/Linux Key Point:**
- Kernel makes NO distinction between text and binary files!
- `"r"` == `"rb"`, `"w"` == `"wb"` in UNIX

### 16.3 ISO C Standard Library ⭐⭐⭐⭐

**I/O Categories:**
1. Character by character
2. Line by line
3. Formatted I/O
4. Binary I/O

**File Operations:**
```c
#include <stdio.h>

FILE *fopen(char *path, char *type);
int fclose(FILE *fp);
```

**Access Modes:**
```c
"r"     // Read
"w"     // Write (create/truncate)
"a"     // Append
"r+"    // Read and write
"w+"    // Write and read (create/truncate)
"a+"    // Read and append
// "b" suffix has no effect in UNIX (rb, wb, etc.)
```

**Character I/O:**
```c
int getc(FILE *fp);            // Read character
int fgetc(FILE *fp);           // Same as getc
int putc(int c, FILE *fp);     // Write character
int fputc(int c, FILE *fp);    // Same as putc
int getchar(void);             // getc(stdin)
int putchar(int c);            // putc(c, stdout)

// Returns: character on success, EOF on error/end-of-file
```

**Line I/O:**
```c
char *gets(char *buf);                    // NEVER USE! (unsafe)
char *fgets(char *buf, int n, FILE *fp);  // Safe alternative
int puts(char *buf);
int fputs(char *buf, FILE *fp);

// Returns: buf or non-negative on success, NULL/EOF on error
// Lines must be delimited by newline
```

**Formatted I/O:**
```c
int scanf(char *format, ...);
int fscanf(FILE *fp, char *format, ...);
int printf(char *format, ...);
int fprintf(FILE *fp, char *format, ...);

// High flexibility: formats, conversions
```

**Binary I/O:** ⭐⭐⭐⭐
```c
size_t fread(void *ptr, size_t size, size_t nObj, FILE *fp);
size_t fwrite(void *ptr, size_t size, size_t nObj, FILE *fp);

// Returns: number of objects read/written
// If return != nObj: error or EOF (use ferror/feof to distinguish)
```

**Serialization:**
- Store entire struct in single operation
- Potential issues: architecture compatibility, alignment

**Buffering:**
```c
void setbuf(FILE *fp, char *buf);
int fflush(FILE *fp);

// Standard I/O is fully buffered
// For concurrent processes (EXAM IMPORTANT!):
setbuf(stdout, 0);    // Disable buffering
fflush(stdout);        // Flush buffer
```

**CRITICAL for fork() output prediction!**
- `setbuf(stdout, 0)` appears in fork() exam questions
- Forces immediate output without buffering

### 16.4 POSIX System Calls ⭐⭐⭐⭐⭐

**MOST EXAM RELEVANT!**

**File Descriptors:**
- Non-negative integers
- Conventionally:
  - 0 = `STDIN_FILENO` (standard input)
  - 1 = `STDOUT_FILENO` (standard output)
  - 2 = `STDERR_FILENO` (standard error)

**open() System Call:** ⭐⭐⭐⭐⭐
```c
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>

int open(const char *path, int flags);
int open(const char *path, int flags, mode_t mode);

// Returns: file descriptor on success, -1 on error
```

**Flags (MEMORIZE!):**

**Mandatory (choose one):**
```c
O_RDONLY        // Read-only
O_WRONLY        // Write-only
O_RDWR          // Read and write
```

**Optional (combine with OR):**
```c
O_CREAT         // Create if doesn't exist
O_EXCL          // Error if O_CREAT and file exists
O_TRUNC         // Truncate to zero length
O_APPEND        // Append mode
O_SYNC          // Wait for physical write to complete
```

**Mode (permissions for new files):**
```c
S_IRUSR         // User read
S_IWUSR         // User write
S_IXUSR         // User execute
S_IRGRP         // Group read
S_IWGRP         // Group write
S_IXGRP         // Group execute
S_IROTH         // Others read
S_IWOTH         // Others write
S_IXOTH         // Others execute

// Actual permissions affected by umask
```

**Examples:**
```c
// Read-only:
int fd = open("file.txt", O_RDONLY);

// Write, create if needed, truncate:
int fd = open("file.txt", O_WRONLY | O_CREAT | O_TRUNC, 0644);

// Append mode:
int fd = open("file.txt", O_WRONLY | O_APPEND);

// Create with specific permissions:
int fd = open("file.txt", O_WRONLY | O_CREAT, 
              S_IRUSR | S_IWUSR | S_IRGRP | S_IROTH);  // 0644
```

**read() System Call:** ⭐⭐⭐⭐⭐
```c
#include <unistd.h>

ssize_t read(int fd, void *buf, size_t nbytes);

// Returns: 
//   number of bytes read (0-nbytes) on success
//   0 on EOF
//   -1 on error
```

**Key Points:**
- May return less than nbytes (EOF or pipe without enough data)
- Must check return value!
- 0 indicates end-of-file

**write() System Call:** ⭐⭐⭐⭐⭐
```c
#include <unistd.h>

ssize_t write(int fd, void *buf, size_t nbytes);

// Returns:
//   number of bytes written (normally nbytes)
//   -1 on error
```

**Key Points:**
- Writes to system buffer, not immediately to disk
- Use `O_SYNC` flag for immediate disk write (ext2 only)
- Should check return value

**Examples:**
```c
// Write array:
float data[10];
if (write(fd, data, 10*sizeof(float)) == -1) {
    fprintf(stderr, "Write error\n");
}

// Write struct:
struct {
    char name[100];
    int n;
    float avg;
} item;
if (write(fd, &item, sizeof(item)) == -1) {
    fprintf(stderr, "Write error\n");
}
```

**lseek() System Call:** ⭐⭐⭐⭐
```c
#include <unistd.h>

off_t lseek(int fd, off_t offset, int whence);

// Returns: new offset on success, -1 on error
```

**whence Values:**
```c
SEEK_SET        // Offset from beginning of file
SEEK_CUR        // Offset from current position
SEEK_END        // Offset from end of file

// offset can be positive or negative
// Can create "holes" in files (filled with zeros)
```

**Examples:**
```c
lseek(fd, 0, SEEK_SET);        // Go to beginning
lseek(fd, 0, SEEK_END);        // Go to end
lseek(fd, 100, SEEK_CUR);      // Forward 100 bytes
lseek(fd, -50, SEEK_CUR);      // Backward 50 bytes
```

**close() System Call:** ⭐⭐⭐
```c
#include <unistd.h>

int close(int fd);

// Returns: 0 on success, -1 on error
// All open files closed automatically when process terminates
```

### 16.5 Complete File Copy Example ⭐⭐⭐⭐⭐

**EXAM PATTERN: Understanding this code!**

```c
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>

#define BUFFSIZE 4096

int main(int argc, char **argv) {
    int nR, nW, fdR, fdW;
    char buf[BUFFSIZE];
    
    // Open source file (read-only)
    fdR = open(argv[1], O_RDONLY);
    
    // Open/create destination file (write, create, truncate)
    fdW = open(argv[2], O_WRONLY | O_CREAT | O_TRUNC,
               S_IRUSR | S_IWUSR);  // 0600 permissions
    
    if (fdR == -1 || fdW == -1) {
        fprintf(stdout, "Error Opening a File.\n");
        exit(1);
    }
    
    // Copy loop
    while ((nR = read(fdR, buf, BUFFSIZE)) > 0) {
        nW = write(fdW, buf, nR);
        if (nR != nW)
            fprintf(stderr, "Error: Read %d, Write %d\n", nR, nW);
    }
    
    if (nR < 0)
        fprintf(stderr, "Write Error.\n");
    
    close(fdR);
    close(fdW);
    exit(0);
}
```

**Key Points:**
- Works with both text and binary files
- Uses buffered reading (BUFFSIZE chunks)
- Checks for errors
- Writes exactly what was read (`nR` bytes)

### What to Memorize

**ISO C:**
- [ ] `fopen()`, `fclose()`, `fread()`, `fwrite()`
- [ ] Access modes: r, w, a, r+, w+, a+
- [ ] `setbuf(stdout, 0)` disables buffering
- [ ] EOF indicates end-of-file or error

**POSIX:**
- [ ] File descriptors: 0=stdin, 1=stdout, 2=stderr
- [ ] `open()` flags: O_RDONLY, O_WRONLY, O_RDWR
- [ ] Optional flags: O_CREAT, O_TRUNC, O_APPEND
- [ ] `read()` returns 0 on EOF, -1 on error
- [ ] `write()` returns number of bytes written
- [ ] `lseek()` whence: SEEK_SET, SEEK_CUR, SEEK_END
- [ ] `close()` closes file descriptor

**Mode Bits:**
- [ ] S_IRUSR, S_IWUSR, S_IXUSR (user)
- [ ] S_IRGRP, S_IWGRP, S_IXGRP (group)
- [ ] S_IROTH, S_IWOTH, S_IXOTH (others)

### Exam Patterns

**Pattern 1: open() Flags**
"Which open() call creates file if it doesn't exist and truncates if it does?"
```c
open("file", O_WRONLY | O_CREAT | O_TRUNC, 0644);
```

**Pattern 2: read() Return Value**
"What does read() return when end-of-file is reached?"
- Answer: 0 (zero)

**Pattern 3: File Copy Logic**
Understanding the while loop pattern:
```c
while ((nR = read(fd, buf, SIZE)) > 0) {
    write(fdW, buf, nR);  // Write exactly what was read
}
```

**Common Mistakes:**
- Confusing return values (0 vs -1 vs bytes)
- Forgetting O_CREAT needs mode parameter
- Not checking return values for errors
- Assuming read() always returns nbytes

---

## SECTION 17: Directory Systems ⭐⭐
**Slides 2-9 (Directories chapter) | Exam Relevance: Low-Medium**

### 17.1 Directory Concepts ⭐⭐

**Purpose:**
- Organize files
- Store information about files
- Both directories and files saved in mass storage

**Advantages:**
- **Efficiency:** Speed in searching/modifying
- **Naming:** Same name for different files in different directories
- **Grouping:** Organize by characteristics

**Operations:**
- Create, delete, list, rename, search, traverse

### 17.2 Single-Level Directory ⭐

**Structure:**
- All files in one directory
- Files differentiated by name only
- Each name unique in entire system

**Advantages:**
- Simple and efficient
- Easy to understand

**Disadvantages:**
- Naming conflicts as files increase
- No grouping capability
- Multi-user management impossible

### 17.3 Two-Level Directory ⭐

**Structure:**
- Main directory contains user directories
- Each user has own directory
- All operations in user's home directory

**Advantages:**
- User-oriented view
- Same filename possible for different users
- Simplified searches within user

**Disadvantages:**
- Complex for individual user organization
- Path name required for each file

### 17.4 Tree Directory Structure ⭐⭐

**Modern Approach:**
- Generalization of two-level
- Directories and files organized as tree
- Any node can contain other nodes

**Exam Relevance:** Conceptual understanding only

---

## SUMMARY: Key Takeaways from Pages 201-300

### CRITICAL CONCEPTS (Must Master):

**1. find Command** ⭐⭐⭐⭐⭐
- Format: `find directory options actions`
- `-regex` with posix-extended
- `-exec command \{\} \;` syntax
- Difference between pipe and exec

**2. grep with Regex** ⭐⭐⭐⭐⭐
- Basic vs Extended RE
- Backward references: `\1`, `\2`
- `-v` for inverse match
- Combined with find

**3. File I/O System Calls** ⭐⭐⭐⭐⭐
- open() with flags and modes
- read() returns 0 on EOF
- write() system behavior
- File descriptors: 0, 1, 2

**4. Filter Commands** ⭐⭐⭐⭐
- cut, tr, sort, uniq
- Pipeline combinations
- grep patterns

### MEMORIZATION CHECKLIST:

**find:**
- [ ] `-regextype posix-extended` before `-regex`
- [ ] `-exec command \{\} \;` syntax
- [ ] `-type f` (file) vs `-type d` (directory)
- [ ] `-mindepth/-maxdepth` for depth control

**grep:**
- [ ] `-E` for Extended RE
- [ ] `-v` inverse match
- [ ] `-A N` lines after, `-B N` lines before
- [ ] Backward refs: `(.)\1` for repetition

**System Calls:**
- [ ] `open()` flags: O_RDONLY, O_WRONLY, O_RDWR, O_CREAT, O_TRUNC
- [ ] `read()` returns: bytes read, 0 (EOF), -1 (error)
- [ ] `write()` returns: bytes written, -1 (error)
- [ ] `lseek()` whence: SEEK_SET, SEEK_CUR, SEEK_END
- [ ] File descriptors: 0=stdin, 1=stdout, 2=stderr

**Filters:**
- [ ] `cut -f` for fields, `-d` for delimiter
- [ ] `tr` requires input redirection
- [ ] `sort -n` numeric, `-k` for key
- [ ] `uniq` requires sorted input

### EXAM STRATEGY:

**High Priority (70% of time):**
1. find with regex (30%)
2. File I/O system calls (25%)
3. grep patterns (15%)

**Medium Priority (25%):**
- Filter command combinations
- Pipeline construction
- sort and uniq usage

**Low Priority (5%):**
- File encoding details
- Directory structures
- Binary file advantages

### COMMON EXAM TRAPS:

❌ "find . -regex '*.c'" → WRONG (*.c is not valid regex)
❌ "read() returns -1 on EOF" → FALSE (returns 0)
❌ "tr pattern file.txt" → WRONG (must use < or |)
❌ "uniq without sort" → May not work correctly
❌ "`open()` with O_CREAT without mode" → Compilation error
❌ "Pipe find to wc counts file contents" → FALSE (counts filenames)

### PRIORITY FOCUS FOR NEXT SECTIONS:

Pages 301-400 will cover **THE MOST CRITICAL EXAM TOPICS**:
- **Process creation with fork()** ⭐⭐⭐⭐⭐ (HIGHEST exam frequency!)
- **Process control and exec()** ⭐⭐⭐⭐⭐
- **Signals** ⭐⭐⭐⭐
- **Pipes and IPC** ⭐⭐⭐⭐⭐

These topics represent **41 questions worth 111 points** - the most tested material!

---

**End of Pages 201-300 Study Guide**
**Total Coverage: Pages 1-300 Complete**
**Next: Process Management (Most Critical Section!)**
