## troubleshooting virt-manager

- if you get network 'default' is not active (the virtual network switch that your VM relies on to connect to the outside world isn't running) when you try to run a VM, open the terminal and run:
    ```bash
    sudo virsh net-start default
    ```
    Set it to autostart (so you don't have to deal with this error again after a reboot):
    ```bash
    sudo virsh net-autostart default
    ```

## terminal shortcuts

- ctrl+a => go to the start of line (command)
- ctrl+e => go to the end
- ctrl+u => clear line
- ctrl+l => clear terminal

- when viewing pages like `man` pages in the terminal, use these keys or shortcuts to move around:
    - g => go to top
    - shift+g => go to bottom
    - up and down arrow keys => move up and down
    - hold enter => go down
    - space move down a whole page
    - /(pattern) => search for a string, like /list (shows all 'list')
    - n => go to the next search result item (top to bottom)
    - shift+n => go to the prev search result item (top to bottom)
    - ?(pattern) => same as '/' but it's reversed so 'n' is go to next item (bottom to top)
    - q => quit

## terminal commands

- `tree -d` => `tree` command but only shows directories
- `tree -d -L 1` =>  `tree -d` but only shows 1 **level** (the first directories, like doing ls but only seeing directories, you can use higher numbers like 3 to see 3 levels)
```bash
~/mobile_project_demo$ tree -d -L 3
.
├── backend_api
│   ├── src
│   │   ├── main
│   │   └── test
│   └── target
│       ├── classes
│       ├── generated-sources
│       ├── generated-test-sources
│       ├── maven-archiver
│       ├── maven-status
│       ├── site
│       ├── surefire-reports
│       └── test-classes
├── docker
└── mobile_app
    ├── app
    │   ├── build
    │   └── src
    └── gradle
        └── wrapper

21 directories
```
- `tree -d -L 2 ./mobile_project_demo/` => using the tree command target a specific dir in cwd
- less => cat command but uses pages so you can scroll down by holding enter and use terminal shortcuts for man pages since they are similar
- more => same as less but it's older (less has more features)
- tail -f (file-path) => best for viewing live logs (-f means follow, so any new lines in that file is shown live)

### grep
- grep with regex
    - `... | grep '^2026-06-17'` => greps every line that starts with that string (^ means line starts with), this is useful for logs where you want to target a specific data, if the logs start with date you can use that, but it's used for anything not just logs and dates.
    - `... | grep 'removed$'` => greps everyline that ends with 'removed' (used $ at the end of arg)
    - `... | grep '....x86_64...'` => this will show every line that has x86_64 in it and highlight 3 chars before 'x86_64' and 2 chars after (the closest dots that are before and after 'x86_64' are the arg providers, the other dots are the amount of chars). It can be also used like:
      `... | grep '11:..:..'` => this will show every line that includes 11th hour from 11:00:00 to 11:59:59. You can also check what happened in a specific time frame like 10:30:00 to 10:50:00:
      `... | grep '10:[3-5]'` => this will only target any line that includes 10:3... , 10:4... and 10:5... (you can do '10:[3-5]0' , it'll only show 30, 40 and 50 mins exactly)

- **`grep -ri 'ibrahim' java_notes/`** => this will find any directory or file name in java_notes/ that contain the string 'ibrahim' (-r recursive -i ignore-case), showing what lines in what files
- `grep -vi 'system' java_notes/src/com/java/basics/Methods.java` => (-v means exclude) this will only show lines that don't contain 'system' (case is ignored with -i)
- `grep -in 'String' java_notes/src/com/java/basics/Arrayss.java` => (-n numbered) will show line numbers associated with the found lines, result is like this:
```java
7:    public static void main(String[] args) {
10:        // for example this string variable can be turned into an array
11:        // to hold more strings
12:        // String fruit = "banana";
14:        String[] fruits = { "orange", "banana", "apple" };
19:        // ([Ljava.lang.String;@2a139a55)
34:        // sort the strings in the array alphabetically
41:        for (String fruit : fruits) {
61:        String[] foods = new String[4];
72:        for (String food : foods) {

```
- `grep -ril 'for' java_notes/src/com/java/` => (-l means files with matches) this will grep recursively through the give dir and show files that contain 'for' in their name or inside the file it self, but it doesn't show text lines inside files, just the file names.
- `grep -o 'for' java_notes/src/com/java/basics/Arrayss.java` => shows only how many times 'for' appears, not full lines, only the string is shown:
```java
for
for
for
for
for
for
for
```

### tar
- archive a file or a dir
```bash
tar -czvf <nameOfOutputFile>.tar.gz <nameOfFileOrDirToBeArchived>
```
> -c: create, -z: zip format. -v: verbose, -f: file
- unarchive/unzip to a specific dir with -C (default unzips to cwd)
```bash
tar -xzvf <nameOfTheArchive>.tar.gz -C /home/<user>/
```
> -x: extract
> e.g `tar -xzvf someBackup.tar.gz -C /home/ibrahim/tests/`, this will extract to the tests dir in home dir


## vim notes

- `:r !cat vimrc` => this is used to copy contents of a file into the file you currently have open in vim, in this instance `vimrc` contents will be put into the current file open in vim. You can use any command and get it's output copied into the editor where your cursor is (here we are using cat)
- gg => go to top of file (while is `esc` mode)
- G => go to bottom of file
- yy => copy line
- p => paste line
- dd => delete line
- u => undo
- :wq => save and quit
- :q! => quit without saving

## extra terminal notes

- A **#** infront of a command makes it do nothing, comments it out
