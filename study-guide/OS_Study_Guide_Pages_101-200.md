# Operating Systems - Exam-Oriented Study Guide
## Pages 101-200: Linux Commands, File Management, C Tools, and Regular Expressions

**Previous Section:** Pages 1-100 covered course introduction, OS concepts, and classification

---

## SECTION 7: Linux Session and Help ⭐⭐
**Slides 16-18 | Exam Relevance: Low (background knowledge)**

### Session Management

**Session Opening:**
```bash
login: <username>
password: <password>
```
- **IMPORTANT:** Linux is case-sensitive!

**Remote Connection:**
```bash
ssh <username>@<hostname>           # Command line interface
ssh -X <username>@<hostname>        # With X11 display forwarding
putty                                # Windows graphical interface
```
- Both use secure encrypted connection protocol

**Session Termination:**
```bash
exit
logout
Ctrl-D
```

### Help System ⭐⭐

**Manual Pages:**
```bash
man <command>          # Display manual page
man ln                 # Example: link command manual
man wc                 # Example: word count manual
```

**Related Commands:**
```bash
apropos <command>      # Search related commands
whatis <command>       # Brief description
whereis <command>      # Locate binary, source, and manual
```

**Command Options:**
```bash
command --help         # Display help
command --version      # Display version
```

### What to Know (Brief)
- Remote access via ssh
- Use `man` for documentation
- Linux is case-sensitive (critical!)

---

## SECTION 8: Command Syntax and Basics ⭐⭐
**Slides 19-20 | Exam Relevance: Low-Medium**

### Command Syntax

**General Format:**
```bash
command [options] [arguments]
```

**Option Formats:**
```bash
-c                     # Single character (can combine: -abc = -a -b -c)
--string               # Long format string
```

### Command Features

**Tab Completion:**
- Press Tab for automatic command/filename completion

**Command History:**
- Up/Down arrows to retrieve previous commands

**Multi-line Commands:**
```bash
gcc -Wall -g \         # Backslash continues on next line
    -o myexe \
    file1.c file2.c
```

**Sequential Commands:**
```bash
command1 ; command2 ; command3     # Execute sequentially
```

---

## SECTION 9: Filenames and Filesystem ⭐⭐⭐
**Slides 21-25 | Exam Relevance: Medium**

### Filename Rules ⭐⭐⭐

**Allowed Characters:**
- Letters, digits, dots '.', underscores '_'
- Case-sensitive: `File.txt` ≠ `file.txt`

**Characters to AVOID:**
```
Space / \ " ' * ; ? [ ] ( ) ~ ! $ { } < > # @ & |
```

**Special Rules:**
- **Hidden files:** Start with dot '.' (e.g., `.bashrc`)
  - Not listed by default with `ls`
  - Use `ls -a` to show hidden files
- **Maximum length:** Typically 255 characters
- **Must be unique** within directory
- **No formal extension** (but conventions exist: `.c`, `.sh`, `.o`, `.tar.gz`)
- Backup files: Often appended with `~` (e.g., `file.txt~`)

**Reserved Character:**
- `/` = directory separator (slash)
- `\0` = null character (cannot use in filenames)

### Filesystem Structure ⭐⭐⭐⭐

**Hierarchical Organization:**
- Tree structure
- Root directory: `/`
- Current directory: `.`
- Parent directory: `..`
- Directories separated by: `/`

**Pathname Types:**

**1. Absolute Pathname** (from root):
```bash
/dir1/dir2/file
/home/user/documents/file.txt
```

**2. Relative Pathname** (from current directory):
```bash
./subdir1/subdir2/file
subdir1/subdir2/file          # ./ is implicit
../sibling_dir/file           # Go up one level
```

### What to Memorize

**Special Directory Symbols:**
- `/` = root directory
- `.` = current directory
- `..` = parent directory
- `~` = home directory

**Key Distinctions:**
- Absolute path starts with `/`
- Relative path starts with `.` or directory name
- Hidden files start with `.`

---

## SECTION 10: File Management Commands ⭐⭐⭐⭐
**Slides 26-42 | Exam Relevance: High (commands tested in exam)**

### 10.1 Listing Files: ls ⭐⭐⭐⭐

**Basic Syntax:**
```bash
ls [-options] [file...]
```

**Common Options (MEMORIZE):**
```bash
ls                     # List files in current directory
ls -l                  # Long format (detailed)
ls -a                  # All files (including hidden)
ls -la                 # Long format + all files
ls -t                  # Sort by time (newest first)
ls -r                  # Reverse order
ls -R                  # Recursive (subdirectories)
ls --help              # Display help
```

**Long Format Output:** ⭐⭐⭐⭐
```bash
$ ls -la
total 72
drwxr-xr-x 8 user1 group1 4096 Oct 7 2013 .
drwxr-xr-x 34 user1 group1 4096 Oct 3 12:37 ..
-rw-r--r-- 1 user1 group1 17715 Oct 7 2013 index.htm
drwxr-xr-x 2 user1 group1 4096 Mar 22 2013 misc
```

**Reading Long Format (CRITICAL FOR EXAM):**
```
-rw-r--r-- 1 user1 group1 17715 Oct 7 2013 index.htm
│└┬┘└┬┘└┬┘ │   │     │     │     │         └─ Filename
│ │  │  │  │   │     │     │     └─ Last modification date
│ │  │  │  │   │     │     └─ Size in bytes
│ │  │  │  │   │     └─ Owner group
│ │  │  │  │   └─ Owner (user)
│ │  │  │  └─ Number of hard links
│ │  │  └─ Permissions for others
│ │  └─ Permissions for group
│ └─ Permissions for user (owner)
└─ File type
```

**File Types:**
```bash
-    # Regular file
d    # Directory
l    # Symbolic link
s    # Socket file
b    # Block device
c    # Character device
p    # Named pipe (FIFO)
```

### 10.2 File Permissions ⭐⭐⭐⭐⭐

**Permission Structure (HEAVILY TESTED!):**
```
rwx rwx rwx
│   │   └─ Others
│   └─ Group
└─ User (owner)
```

**Permission Meanings:**
- **r** = read (4)
- **w** = write (2)
- **x** = execute (1)

**Octal Representation (MEMORIZE):**
```bash
rwx = 7  (4+2+1)
rw- = 6  (4+2)
r-x = 5  (4+1)
r-- = 4
-wx = 3  (2+1)
-w- = 2
--x = 1
--- = 0
```

**Examples:**
```bash
rwxr-xr-x = 755    # Owner: full, Group: r+x, Others: r+x
rw-r--r-- = 644    # Owner: r+w, Group: r, Others: r
rwx--x--- = 710    # Owner: full, Group: x only, Others: none
```

**Permissions for Files vs Directories (EXAM CRITICAL!):**

**For Regular Files:**
- **r** = Read file content
- **w** = Modify file content
- **x** = Execute file as program

**For Directories:**
- **r** = List directory contents (`ls`)
- **w** = Create, rename, delete files within directory
- **x** = Access directory (cd), traverse it

**EXAM TRAP:**
- `cd dir` requires execute (`x`) permission on `dir`
- `ls dir` requires read (`r`) permission on `dir`
- Creating file in `dir` requires write (`w`) permission on `dir`

### 10.3 File Operations ⭐⭐⭐

**Copy Files:**
```bash
cp [options] src1 src2 ... dest
cp file1 file2                   # Copy file1 to file2
cp file1 file2 file3 dir/        # Copy multiple files to directory
cp -r dir1 dir2                  # Copy directory recursively
```

**Remove Files:**
```bash
rm [options] file1 file2 ...
rm file.txt                      # Remove file (asks confirmation if no write permission)
rm -f file.txt                   # Force removal (no confirmation)
rm -r dir/                       # Remove directory recursively
rm -rf dir/                      # Force recursive removal (DANGEROUS!)
```

**Move/Rename Files:**
```bash
mv [options] src1 src2 ... dest
mv file1 file2                   # Rename file1 to file2
mv file1 file2 dir/              # Move files to directory
```

**Common Options:**
```bash
-f     # Force (no confirmation)
-i     # Interactive (ask confirmation)
-r/-R  # Recursive (for directories)
```

**Important Notes:**
- `rm` without write permission asks confirmation
- `mv` is implemented as `ln` + `rm` (link then remove)
- File removed only when hard link count reaches 0

### 10.4 Directory Management ⭐⭐⭐

**Basic Commands:**
```bash
cd dir          # Change to directory
cd              # Change to home directory
cd ~            # Change to home directory
cd ..           # Change to parent directory
cd -            # Change to previous directory

pwd             # Print working directory

mkdir dir       # Create directory
mkdir -p dir1/dir2/dir3   # Create parent directories as needed

rmdir dir       # Remove empty directory
rm -rf dir      # Remove directory and contents (DANGEROUS!)
```

### 10.5 Permission Management ⭐⭐⭐⭐⭐

**Change Permissions: chmod** (HEAVILY TESTED!)

**Octal Method:**
```bash
chmod 755 filename              # rwxr-xr-x
chmod 644 filename              # rw-r--r--
chmod 700 filename              # rwx------
```

**Symbolic Method:**
```bash
chmod u+x filename              # Add execute for user
chmod g+r filename              # Add read for group
chmod o-w filename              # Remove write for others
chmod a+x filename              # Add execute for all
chmod uo+rx filename            # Add read+execute for user and others
chmod +x filename               # Add execute for all (implicit 'a')
```

**Symbolic Format:**
```
[ugoa][+-=][rwx]

u = user (owner)
g = group
o = others
a = all

+ = add permission
- = remove permission
= = set exactly (overwrite)
```

**User Management Commands:**
```bash
su username                     # Switch user
sudo command                    # Run command as superuser
sudo -u user command            # Run command as specified user
whoami                          # Show current user
```

**Change Ownership:**
```bash
chown user file                 # Change owner
chown user:group file           # Change owner and group
chown -R user:group dir/        # Recursive ownership change
chgrp group file                # Change group
```

### 10.6 File Content Viewing ⭐⭐⭐

**Display File Content:**
```bash
cat file1 file2 ...             # Concatenate and display files
head file                       # First 10 lines (default)
head -n 5 file                  # First 5 lines
head -5 file                    # First 5 lines (compact)
tail file                       # Last 10 lines (default)
tail -n 5 file                  # Last 5 lines
tail -5 file                    # Last 5 lines (compact)
tail -f file                    # Follow file (continuous monitoring)
```

**Page Through Files:**
```bash
more file                       # View file page by page
less file                       # Advanced paging (can go backward)
pg file                         # Page-wise browsing
```

**Navigation in less/more:**
```bash
Space     # Next page
Return    # Next line
b         # Previous page
/str      # Search forward for 'str'
?str      # Search backward for 'str'
q         # Quit
```

### 10.7 File Comparison ⭐⭐

**Compare Files:**
```bash
diff file1 file2                # Show differences
diff -q file1 file2             # Brief (only if different)
diff -b file1 file2             # Ignore trailing spaces
diff -i file1 file2             # Case insensitive
diff -w file1 file2             # Ignore all whitespace
diff -B file1 file2             # Ignore blank lines
diff dir1 dir2                  # Compare directories
```

**Difference Markers:**
- `a` = added lines
- `d` = deleted lines
- `c` = changed lines

### 10.8 Word Count ⭐⭐⭐

**wc Command:**
```bash
wc file                         # Lines, words, bytes
wc -l file                      # Line count only
wc -w file                      # Word count only
wc -c file                      # Byte count only
wc -m file                      # Character count only
```

**Output format:**
```bash
$ wc file.txt
  42  256  1847 file.txt
  │   │    │    └─ Filename
  │   │    └─ Bytes
  │   └─ Words
  └─ Lines
```

### 10.9 Links ⭐⭐⭐⭐

**Two Types of Links (EXAM IMPORTANT!):**

**1. Hard Link:**
- Association between name and content (directory entry → inode)
- **Same inode** as original file
- Cannot cross filesystems
- Cannot link directories (usually)
- File removed only when last hard link removed
- Link count shown in `ls -l`

**2. Symbolic (Soft) Link:**
- Special file containing path to target
- **Different inode** from original
- Can cross filesystems
- Can link directories
- Becomes "dangling" if target deleted
- Shows target in `ls -l` output

**Creating Links:**
```bash
ln source destination           # Create hard link
ln -s source destination        # Create symbolic link
ln -s /home/foo/file link       # Symbolic link to file
ln /home/foo/file .             # Hard link in current directory
```

**Key Differences (MEMORIZE):**
```
                Hard Link    Symbolic Link
Same inode      YES          NO
Cross filesystem NO           YES
Link directories NO (usually) YES
Target deleted   Still works  Broken (dangling)
```

**EXAM PATTERN:**
- "rm removes file when link count = 0" → TRUE (hard links)
- "mv = ln + rm" → TRUE
- Hard links share inode, soft links don't

### What to Memorize

**Permission Octal Values:**
- [ ] rwx = 7, rw- = 6, r-x = 5, r-- = 4
- [ ] 755 = rwxr-xr-x (common for executables)
- [ ] 644 = rw-r--r-- (common for files)

**ls -l Format:**
- [ ] Order: type, permissions, links, owner, group, size, date, name
- [ ] File types: - (file), d (directory), l (link)

**Permission Meanings:**
- [ ] Files: r=read, w=write, x=execute
- [ ] Directories: r=list, w=create/delete, x=access/traverse

**Link Distinctions:**
- [ ] Hard: same inode, can't cross filesystem
- [ ] Soft: different inode, can cross filesystem

### Exam Patterns

**True/False Questions:**
1. "cp file1 file2 fails if file1 lacks read permission" → TRUE
2. "cd dir fails if dir lacks execute permission" → TRUE
3. "ls dir fails if dir lacks read permission" → TRUE
4. "Hard links can cross filesystem boundaries" → FALSE
5. "Symbolic links become dangling when target deleted" → TRUE
6. "chmod 755 gives full permissions to owner only" → FALSE (group/others get r-x)

**Common Mistakes:**
- Confusing file vs directory permission meanings
- Forgetting execute permission needed to cd into directory
- Thinking hard links can cross filesystems
- Confusing octal permission values

---

## SECTION 11: Archive Management ⭐⭐
**Slides 53-56 | Exam Relevance: Low-Medium**

### tar Command

**Create Archive:**
```bash
tar czvf file.tgz dir/          # Create compressed archive
```

**Extract Archive:**
```bash
tar xzvf file.tgz               # Extract archive
```

**Common Options:**
```bash
c     # Create archive
x     # Extract archive
z     # gzip compression
j     # bzip2 compression
J     # xz/7z compression (highest compression)
v     # Verbose (show progress)
f     # Specify filename (always required)
```

**Alternative Commands:**
```bash
gzip/gunzip                     # Compress/decompress .gz
zip/unzip                       # Compress/decompress .zip
rar/unrar                       # Compress/decompress .rar
```

### Disk Usage ⭐⭐

**Check Filesystem Space:**
```bash
df                              # Disk free space
df -h                           # Human-readable (K, M, G)
df -k                           # Output in kilobytes
```

**Check Directory Size:**
```bash
du directory                    # Disk usage
du -a directory                 # All files
du -s directory                 # Summary only
du -h directory                 # Human-readable
```

### What to Know (Brief)
- tar for archiving/compression
- df for disk space
- du for directory size
- Rarely directly tested in exam

---

## SECTION 12: C Programming Tools ⭐⭐⭐
**Slides 2-43 (C Programming chapter) | Exam Relevance: Medium (Makefile tested)**

### 12.1 Programming Environment ⭐

**IDEs and Editors** (Background only):
- Eclipse, Visual Studio, NetBeans (IDEs)
- Vim, Emacs, VS Code, Nano (Editors)
- **Vim** = Vi Improved (present on all UNIX systems)
- **Emacs** = Editor MACroS (powerful, extensible)

**Not directly tested** - focus on gcc and Makefile

### 12.2 GCC Compiler ⭐⭐⭐

**Basic Syntax:**
```bash
gcc [options] [files...]
```

**Common Options (MEMORIZE):**
```bash
-c file            # Compile only (create .o object file)
-o filename        # Specify output filename
-g                 # Include debug info (for gdb)
-Wall              # Show all warnings
-Idir              # Add include directory
-Ldir              # Add library directory
-lname             # Link library (e.g., -lm for math)
```

**Compilation Examples:**
```bash
# Compile only
gcc -c file1.c                  # Creates file1.o
gcc -c file2.c                  # Creates file2.o

# Link object files
gcc -o myexe file1.o file2.o

# Compile and link in one step
gcc -o myexe file1.c file2.c main.c

# With all options
gcc -Wall -g -I./include -o myexe main.c lib.c -lm
```

**Important Notes:**
- `-I` and `-L` have NO space after them: `-I./include`
- `-lm` links math library
- `-g` for debugging (don't optimize, add debug symbols)
- `-Wall` catches potential errors

### 12.3 Makefile ⭐⭐⭐⭐⭐

**CRITICAL FOR EXAM!**

**Purpose:**
1. Automate repetitive compilation tasks
2. Avoid unnecessary recompilation (check dependencies and timestamps)

**Basic Structure:**
```makefile
target: dependencies
<TAB>command
```

**CRITICAL:** Command line MUST start with TAB character (not spaces!)

**Execution:**
```bash
make                            # Execute first target
make targetName                 # Execute specific target
make -f filename targetName     # Use alternate makefile
```

**Example 1 - Simple Makefile:**
```makefile
target:
	gcc -Wall -o myExe main.c -lm
```
- Single target, no dependencies
- Runs: `make` or `make target`

**Example 2 - Multiple Targets:**
```makefile
project1:
	gcc -Wall -o project1 myFile1.c
project2:
	gcc -Wall -o project2 myFile2.c
```
- `make` runs first target (project1)
- `make project2` runs second target

**Example 3 - With Dependencies:** ⭐⭐⭐⭐⭐
```makefile
target: file1.o file2.o
	gcc -Wall -o myExe file1.o file2.o

file1.o: file1.c myLib1.h
	gcc -Wall -g -c file1.c

file2.o: file2.c myLib1.h myLib2.h
	gcc -Wall -g -c file2.c
```

**Dependency Resolution (EXAM CRITICAL!):**
1. Check if dependencies more recent than target
2. If yes, execute dependency rules first (recursively)
3. Then execute target rule
4. If no changes, skip execution

**Example Execution Flow:**
- `make target` called
- Checks: Is `file1.o` older than `file1.c` or `myLib1.h`?
  - If yes: Compile `file1.c` → create `file1.o`
  - If no: Skip
- Checks: Is `file2.o` older than `file2.c`, `myLib1.h`, or `myLib2.h`?
  - If yes: Compile `file2.c` → create `file2.o`
  - If no: Skip
- Execute target: Link `file1.o` + `file2.o` → `myExe`

**PHONY Targets:** ⭐⭐⭐⭐
```makefile
.PHONY: clean

clean:
	rm -rf *.o *.txt

target: file.o
	gcc -o exe file.o
```

**Purpose of .PHONY:**
- Marks target as action (not filename)
- Always executes regardless of file existence
- Even if file named "clean" exists, `make clean` still runs

**EXAM QUESTION PATTERN:**
```makefile
distclean: clean1 clean2
	rm -f distclean
clean1: clean3
	rm -f clean1
clean2:
	rm -f clean2
clean3:
	rm -f clean3
```

**Question:** "Executing `make distclean`, what's the order?"
**Answer:** clean3 → clean1 → clean2 → distclean (dependencies first!)

**Makefile Variables (Macros):** ⭐⭐⭐
```makefile
CC = gcc
FLAGS = -Wall -g
SRC = main.c bst.c list.c

project: $(SRC)
	$(CC) $(FLAGS) -o project $(SRC) -lm
```

**Automatic Variables:**
```makefile
$@      # Target name
$^      # All dependencies
$<      # First dependency
```

**Example with Automatic Variables:**
```makefile
CC = gcc
FLAGS = -Wall -g

project: main.o bst.o
	$(CC) $(FLAGS) -o $@ $^

main.o: main.c main.h
	$(CC) $(FLAGS) -c $<
```

### What to Memorize

**GCC Options:**
- [ ] `-c` = compile only
- [ ] `-o` = output filename
- [ ] `-g` = debug info
- [ ] `-Wall` = all warnings
- [ ] `-I` = include directory
- [ ] `-l` = link library

**Makefile Rules:**
- [ ] Format: `target: dependencies` then TAB + command
- [ ] Default: first target executes
- [ ] Dependencies checked recursively
- [ ] .PHONY for action targets
- [ ] TAB character mandatory (not spaces!)

**Automatic Variables:**
- [ ] `$@` = target name
- [ ] `$^` = all dependencies
- [ ] `$<` = first dependency

### Exam Patterns

**Makefile Execution Order:**
Given complex dependencies, trace execution order (dependencies first, depth-first)

**PHONY Purpose:**
"What does .PHONY do?" → Forces target execution regardless of files

**Common Mistakes:**
- Using spaces instead of TAB
- Forgetting dependencies execute before target
- Not understanding .PHONY purpose
- Confusing gcc compile vs link options

### 12.4 GDB Debugger ⭐
**Slides 41-43 | Exam Relevance: Minimal**

**Brief Overview Only:**
```bash
gdb executable                  # Start debugger
(gdb) run                       # Run program
(gdb) break main                # Set breakpoint
(gdb) next                      # Next line
(gdb) step                      # Step into function
(gdb) print variable            # Print value
(gdb) continue                  # Continue execution
(gdb) quit                      # Exit debugger
```

**Not heavily tested** - focus on using, not memorizing commands

---

## SECTION 13: Regular Expressions ⭐⭐⭐⭐⭐
**Slides 2-12 (Regex chapter) | Exam Relevance: CRITICAL**

### 13.1 Regular Expression Basics ⭐⭐⭐⭐⭐

**Definition:**
- Expression specifying set of strings
- Compact operators for complex character sequences
- Used in: find, grep, sed, awk, scripting languages

**Standards:**
- **BRE** = Basic Regular Expression
- **ERE** = Extended Regular Expression (used with `-E` flag)
- **PCRE** = Perl Compatible Regular Expression

### 13.2 Metacharacters ⭐⭐⭐⭐⭐

**MUST MEMORIZE FOR EXAM!**

**Quantifiers:**
```
*       # 0 or more times
+       # 1 or more times
?       # 0 or 1 time
{n}     # Exactly n times
{n,m}   # Between n and m times
{n,}    # n or more times
{,m}    # up to m times
```

**Character Classes:**
```
[abc]   # Any character a, b, or c
[a-z]   # Any lowercase letter
[A-Z]   # Any uppercase letter
[0-9]   # Any digit
[^abc]  # Any character NOT a, b, or c
```

**Special Characters:**
```
.       # Any character (except newline)
\d      # Digit [0-9]
\D      # Not digit [^0-9]
\w      # Word character [0-9A-Za-z_]
\W      # Not word character
\s      # Space or TAB
\c      # Control character
```

**Anchors:**
```
^       # Beginning of line
$       # End of line
\<      # Beginning of word
\>      # End of word
```

**Grouping and Alternation:**
```
(...)   # Group/capture
|       # OR (alternation)
```

**Escape Sequences:**
```
\n      # Newline
\t      # TAB
\.      # Literal dot
\+      # Literal plus
\?      # Literal question mark
```

### 13.3 Regular Expression Examples ⭐⭐⭐⭐⭐

**Basic Patterns:**
```regex
ABCDEF                  # Literal string "ABCDEF"
a*b                     # Zero or more 'a' followed by 'b'
                        # Matches: b, ab, aab, aaab, ...

ab?                     # 'a' followed by optional 'b'
                        # Matches: a, ab

a{5,15}                 # 5 to 15 repetitions of 'a'
(fred){3,9}             # 3 to 9 repetitions of "fred"

.+                      # One or more of any character (non-empty line)
.*                      # Zero or more of any character (any string, including empty)
```

**Anchored Patterns:**
```regex
^ABC.*                  # Line starting with "ABC"
.*h$                    # Line ending with "h"
^[0-9]+$                # Line containing only digits
hello\>                 # Word ending with "hello"
\<hello\>               # Exact word "hello"
```

**Character Classes:**
```regex
[a-zA-Z0-9]             # Any alphanumeric character
[aeiou]                 # Any vowel
[^0-9]                  # Any non-digit
```

**Complex Patterns:**
```regex
myfunc.*(.*)            # Function starting with "myfunc"
a+b+                    # One or more 'a' followed by one or more 'b'

(4\.[0-2])|(2\.[0-2])  # Numbers: 4.0, 4.1, 4.2, 2.0, 2.1, 2.2
                        # Must escape dot: \.

\w{8}                   # 8-character word
```

**Backward References:** ⭐⭐⭐⭐
```regex
(.)\1                   # Same character twice
                        # Matches: aa, bb, 11, etc.

(.)(.).\2\1             # 5-char palindrome
                        # Matches: radar, civic, 12321

\1, \2, \3              # Reference captured groups
```

### 13.4 Practical Examples ⭐⭐⭐⭐⭐

**Example 1: Match integers 1-50**
```regex
^[1-9]$ | ^[1-4][0-9]$ | ^50$

Breakdown:
^[1-9]$         # 1-9 (single digit)
^[1-4][0-9]$    # 10-49 (1-4 followed by 0-9)
^50$            # Exactly 50
```

**Example 2: Date format yyyy/mm/dd**
```regex
\d{2,4}\/(0[1-9]|1[0-2])\/(0[1-9]|[1-2][0-9]|3[01])

Breakdown:
\d{2,4}                    # Year: 2-4 digits
\/                         # Literal slash
(0[1-9]|1[0-2])           # Month: 01-12
\/                         # Literal slash
(0[1-9]|[1-2][0-9]|3[01]) # Day: 01-31
```

**Example 3: Matching filename patterns (EXAM PATTERN!):**
```regex
\./(2[468]|[3-5][02468]|6[02468])(xx|yy)*\.(c|java)

Breakdown:
\./                        # Literal ./ (current directory)
(2[468]|[3-5][02468]|6[02468])  # 24,26,28,30,...,68
(xx|yy)*                   # Zero or more "xx" or "yy"
\.                         # Literal dot
(c|java)                   # Extension: c or java

Matches:
./24.c, ./26xx.c, ./48xxyy.java, ./68.c
```

### What to Memorize

**Quantifiers:**
- [ ] `*` = 0 or more
- [ ] `+` = 1 or more
- [ ] `?` = 0 or 1
- [ ] `{n}` = exactly n
- [ ] `{n,m}` = n to m times

**Character Classes:**
- [ ] `[abc]` = any of a, b, c
- [ ] `[a-z]` = any lowercase
- [ ] `[^abc]` = not a, b, c
- [ ] `\d` = digit, `\w` = word char

**Anchors:**
- [ ] `^` = start of line
- [ ] `$` = end of line
- [ ] `\<` = start of word
- [ ] `\>` = end of word

**Special:**
- [ ] `.` = any character
- [ ] `|` = OR
- [ ] `()` = group/capture
- [ ] `\1` = backreference to group 1

### Exam Patterns

**HEAVILY TESTED with find command!**

**Common Question Type:**
"Which files does this regex match?"
```regex
\./(2[468]|[3-5][02468])(xx|yy)*\.(c|java)
```

**Answer Strategy:**
1. Break down each part
2. List what each group matches
3. Combine possibilities
4. Check specific filenames

**Example Questions:**
1. "Does `./35xx.java` match?" 
   - Check: 35 matches `[3-5][02468]`? NO (35 has 5, needs even)
   - Answer: NO

2. "Does `./24xxyy.c` match?"
   - Check: 24 matches `2[468]`? YES
   - Check: `xxyy` matches `(xx|yy)*`? YES
   - Check: `.c` matches `\.(c|java)`? YES
   - Answer: YES

**Common Mistakes:**
- Forgetting to escape special characters (`.` becomes `\.`)
- Confusing `*` (0+) with `+` (1+)
- Not understanding `^` and `$` anchors
- Misreading character classes `[0-9]` vs `[^0-9]`
- Forgetting parentheses for grouping with `|`

---

## SUMMARY: Key Takeaways from Pages 101-200

### CRITICAL CONCEPTS (Must Master):

**1. File Permissions** ⭐⭐⭐⭐⭐
- Octal: 755=rwxr-xr-x, 644=rw-r--r--
- File vs directory meanings
- chmod syntax (octal and symbolic)

**2. Makefile** ⭐⭐⭐⭐⭐
- Format: target: dependencies + TAB + command
- Dependency resolution order
- .PHONY targets
- Automatic variables ($@, $^, $<)

**3. Regular Expressions** ⭐⭐⭐⭐⭐
- Quantifiers: *, +, ?, {n,m}
- Character classes: [abc], [a-z], [^abc]
- Anchors: ^, $, \<, \>
- Escaping special characters

**4. File Operations** ⭐⭐⭐⭐
- ls -l output format
- cp, mv, rm with options
- Hard vs soft links
- Permission requirements for operations

**5. GCC Compilation** ⭐⭐⭐
- Options: -c, -o, -g, -Wall
- Include/library paths: -I, -L, -l
- Compilation vs linking

### MEMORIZATION CHECKLIST:

**Commands:**
- [ ] ls -l, ls -a, ls -R, ls -t
- [ ] chmod (octal and symbolic)
- [ ] cp, mv, rm with -r, -f, -i options
- [ ] ln vs ln -s (hard vs soft link)
- [ ] cat, head, tail, wc
- [ ] diff, grep basics

**Permissions:**
- [ ] rwx = 7, rw- = 6, r-x = 5, r-- = 4
- [ ] File: r=read, w=write, x=execute
- [ ] Directory: r=list, w=modify, x=access

**Makefile:**
- [ ] TAB character mandatory
- [ ] First target is default
- [ ] Dependencies execute first
- [ ] .PHONY forces execution

**Regex:**
- [ ] * = 0+, + = 1+, ? = 0 or 1
- [ ] [abc] = any, [^abc] = none
- [ ] ^ = start, $ = end
- [ ] . = any char, \. = literal dot

### EXAM STRATEGY FOR THIS MATERIAL:

**High Priority:**
1. **Regex with find** (30% of study time)
   - Practice breaking down complex patterns
   - Understand each metacharacter
   - Test with example filenames

2. **Makefile execution** (25% of study time)
   - Trace dependency order
   - Understand .PHONY
   - Practice with examples

3. **File permissions** (25% of study time)
   - Convert octal ↔ symbolic
   - File vs directory meanings
   - chmod syntax variations

4. **Linux commands** (20% of study time)
   - Common options for ls, cp, mv, rm
   - Hard vs soft links
   - wc, diff, tar basics

**Medium Priority:**
- GCC compilation options
- Directory navigation
- File viewing commands

**Low Priority:**
- IDEs and editors
- GDB commands
- Archive compression ratios

### COMMON EXAM TRAPS:

❌ "Directory execute permission allows listing files" → FALSE (need read!)
❌ "chmod 755 gives write to group" → FALSE (only r+x)
❌ "Hard links can cross filesystems" → FALSE
❌ "In regex, * means 1 or more" → FALSE (0 or more!)
❌ ". in regex means literal dot" → FALSE (need \.)
❌ "First line in Makefile is comment" → FALSE (first target executes!)
❌ "Makefile commands can use spaces instead of TAB" → FALSE (TAB mandatory!)

### PRIORITY FOCUS FOR NEXT SECTIONS:

Based on exam analysis, upcoming topics (pages 201+) will include:
- **Filters and text processing** (grep, sed, awk) ⭐⭐⭐⭐
- **Process creation with fork()** ⭐⭐⭐⭐⭐ (MOST CRITICAL!)
- **File I/O system calls** ⭐⭐⭐⭐⭐
- **Pipes and redirection** ⭐⭐⭐⭐⭐
- These are HEAVILY tested → deserve maximum focus!

---

## STUDY RECOMMENDATION:

**Time allocation for pages 101-200:**
- Quick review: 45 minutes
- Deep focus: Regex (30 min), Makefile (30 min), Permissions (25 min), Commands (20 min)
- Practice: Create sample Makefiles, test regex patterns, practice chmod

**What's coming next (pages 201+):**
- grep and text filters ⭐⭐⭐⭐
- File I/O system calls (open, read, write) ⭐⭐⭐⭐⭐
- Process management (fork, exec, wait) ⭐⭐⭐⭐⭐ (MOST CRITICAL EXAM TOPIC!)
- Pipes and IPC ⭐⭐⭐⭐⭐

**These topics have highest exam frequency → allocate maximum study time!**

---

**End of Pages 101-200 Study Guide**
**Total Coverage: Pages 1-200 Complete**
