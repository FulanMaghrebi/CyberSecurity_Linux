# Filesystem

This README deals with important Linux commands used to operate and navigate the filesystem.

---

## pwd

Prints the current working directory.

```bash
pwd
```

### Options

`-L` → shows the logical path and keeps symbolic links in the path.

```bash
pwd -L
```

`-P` → shows the physical path and resolves symbolic links.

```bash
pwd -P
```

---

## ls

Lists files and directories.

```bash
ls
```

### Options

`-l` → detailed view.

```bash
ls -l
```

`-a` → shows hidden files.

```bash
ls -a
```

`-h` → shows file sizes in a human-readable format. Usually combined with `-l`.

```bash
ls -lh
```

Example:

```text
4096 → 4.0K
1048576 → 1.0M
```

Options can be combined:

```bash
ls -lah
```

This means:

```text
-l → detailed view
-a → show hidden files
-h → human-readable file sizes
```

---

## cd

Changes the current working directory.

```bash
cd DIRECTORY
```

Example:

```bash
cd Documents
```

### Useful variations

Go to the parent directory:

```bash
cd ..
```

Go to your home directory:

```bash
cd ~
```

or simply:

```bash
cd
```

Go to the previous directory:

```bash
cd -
```

Go to the root directory:

```bash
cd /
```

---

## mkdir

Creates a new directory.

```bash
mkdir DIRECTORY
```

Example:

```bash
mkdir projects
```

### Options

`-p` → creates parent directories if they do not exist.

```bash
mkdir -p projects/linux/filesystem
```

Without `-p`, the command fails if one of the parent directories does not exist.

Create multiple directories:

```bash
mkdir dir1 dir2 dir3
```

---

## touch

Creates an empty file if the file does not already exist.

```bash
touch FILE
```

Example:

```bash
touch notes.txt
```

If the file already exists, `touch` updates its timestamps instead of deleting or overwriting its contents.

Create multiple files:

```bash
touch file1.txt file2.txt file3.txt
```

---

## cp

Copies files or directories.

```bash
cp SOURCE DESTINATION
```

Example:

```bash
cp notes.txt backup.txt
```

Copy a file into another directory:

```bash
cp notes.txt backup/
```

### Options

`-r` → recursively copies directories and their contents.

```bash
cp -r directory backup/
```

`-i` → asks before overwriting an existing file.

```bash
cp -i file.txt backup/
```

`-v` → verbose mode. Shows what is being copied.

```bash
cp -v file.txt backup/
```

Options can be combined:

```bash
cp -riv source/ destination/
```

---

## mv

Moves files or directories.

It can also be used to rename files and directories.

```bash
mv SOURCE DESTINATION
```

Move a file:

```bash
mv file.txt Documents/
```

Rename a file:

```bash
mv old.txt new.txt
```

Rename a directory:

```bash
mv old_directory new_directory
```

### Options

`-i` → asks before overwriting an existing file.

```bash
mv -i file.txt destination/
```

`-v` → shows what is being moved.

```bash
mv -v file.txt destination/
```

---

## rm

Removes files.

```bash
rm FILE
```

Example:

```bash
rm notes.txt
```

### Options

`-i` → asks before deleting.

```bash
rm -i notes.txt
```

`-r` → recursively removes directories and their contents.

```bash
rm -r directory/
```

`-f` → forces removal without asking for confirmation.

```bash
rm -f file.txt
```

Options can be combined:

```bash
rm -rf directory/
```

**Warning:** `rm -rf` can recursively delete entire directory structures without asking for confirmation. Use it carefully.

---

## rmdir

Removes an empty directory.

```bash
rmdir DIRECTORY
```

Example:

```bash
rmdir empty_directory
```

The directory must be empty.

If the directory contains files, `rmdir` will fail.

For directories containing files, `rm -r` can be used instead.

---

## cat

Displays the contents of a file directly in the terminal.

```bash
cat FILE
```

Example:

```bash
cat notes.txt
```

Display multiple files:

```bash
cat file1.txt file2.txt
```

### Options

`-n` → displays line numbers.

```bash
cat -n notes.txt
```

`cat` can also combine files:

```bash
cat file1.txt file2.txt > combined.txt
```

---

## less

Displays file contents one page at a time.

```bash
less FILE
```

Example:

```bash
less largefile.txt
```

This is useful for large files because the whole file does not need to be displayed at once.

### Useful controls

```text
Arrow keys → move up/down

Space → next page

b → previous page

/word → search for "word"

n → next search result

q → quit
```

---

## head

Displays the beginning of a file.

By default, it shows the first 10 lines.

```bash
head FILE
```

Example:

```bash
head notes.txt
```

### Options

`-n` → defines how many lines should be displayed.

```bash
head -n 5 notes.txt
```

This displays the first 5 lines.

A shorter form is:

```bash
head -5 notes.txt
```

---

## tail

Displays the end of a file.

By default, it shows the last 10 lines.

```bash
tail FILE
```

Example:

```bash
tail notes.txt
```

### Options

`-n` → defines how many lines should be displayed.

```bash
tail -n 5 notes.txt
```

`-f` → follows a file and displays new lines as they are added.

```bash
tail -f logfile.log
```

This is especially useful for monitoring log files.

Example:

```bash
tail -f /var/log/syslog
```

---

## find

Searches for files and directories.

Basic syntax:

```bash
find LOCATION CONDITIONS
```

Search for a file named `notes.txt`:

```bash
find . -name "notes.txt"
```

`.` means search from the current directory.

Search from your home directory:

```bash
find ~ -name "notes.txt"
```

### Useful options

`-name` → searches by name.

```bash
find . -name "*.txt"
```

`-iname` → searches by name without case sensitivity.

```bash
find . -iname "notes.txt"
```

This can find:

```text
notes.txt
Notes.txt
NOTES.txt
```

`-type f` → searches only for files.

```bash
find . -type f
```

`-type d` → searches only for directories.

```bash
find . -type d
```

Combine conditions:

```bash
find . -type f -name "*.txt"
```

This searches for all `.txt` files below the current directory.

---

## ln

Creates links to files or directories.

There are two important types:

```text
Hard links
Symbolic links
```

### Hard link

```bash
ln TARGET LINK
```

Example:

```bash
ln file.txt hardlink.txt
```

A hard link points to the same underlying file data.

Deleting one filename does not necessarily delete the data while another hard link still exists.

### Symbolic link

A symbolic link, or symlink, acts similar to a shortcut.

```bash
ln -s TARGET LINK
```

Example:

```bash
ln -s /home/laghm/projects ~/projects-link
```

Now:

```bash
cd ~/projects-link
```

leads to:

```text
/home/laghm/projects
```

### Options

`-s` → creates a symbolic link.

```bash
ln -s target link
```

`-f` → removes an existing destination file when creating the link.

```bash
ln -sf target link
```

View symbolic links with:

```bash
ls -l
```

Example output:

```text
projects-link -> /home/laghm/projects
```

If the original target of a symbolic link is removed or moved, the symbolic link can become a **broken symbolic link**.
---

# Quick Overview

| Command | Purpose                                |
| ------- | -------------------------------------- |
| `pwd`   | Show current directory                 |
| `ls`    | List files and directories             |
| `cd`    | Change directory                       |
| `mkdir` | Create directories                     |
| `touch` | Create empty files / update timestamps |
| `cp`    | Copy files and directories             |
| `mv`    | Move or rename files and directories   |
| `rm`    | Remove files and directories           |
| `rmdir` | Remove empty directories               |
| `cat`   | Display file contents                  |
| `less`  | Read files page by page                |
| `head`  | Display beginning of a file            |
| `tail`  | Display end of a file                  |
| `find`  | Search for files and directories       |
| `ln`    | Create hard links and symbolic links   |
| `netstat` | Show network connections and ports   |

#netstat Quick Overview

| Option | Purpose |
| ------ | ------- |
| `-a` | Show all sockets |
| `-t` | Show TCP sockets |
| `-u` | Show UDP sockets |
| `-l` | Show listening sockets |
| `-n` | Show numerical IP addresses and ports |
| `-p` | Show PID and program |
| `-r` | Show routing table |
| `-i` | Show network interfaces |
| `-s` | Show protocol statistics |
| `-c` | Continuously refresh output |
| `-v` | Verbose output |
