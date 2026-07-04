# Create simple shell scripts

## Conditionally execute code (use of: if, test, [], etc.)
- `test` and `[ ]` are literally the exact same tool under the hood, `[[ ]]` is the newer smarter version of `[ ]` so make sure to use `[[ ]]` in the exam.
- In Bash, spaces are actually syntax. If you bunch things together, the script will crash. Always leave a space after the opening bracket/parenthesis and a space before the closing one.
    - Wrong Syntx:    if [[$NAME=="root"]]
    - Correct Syntax: if [[ $NAME == "root" ]]
- Memorize These 4 File Flags:
`-e` -> If `any` file or folder exists -> [[ -e /etc/hosts ]]
`-f` -> If it exists and is a `regular file` -> [[ -f /etc/passwd ]]
`-d` -> If it exists and is a `directory` -> [[ -d /var/log ]]
`-s` -> If a `file` exists and is `not empty` (size > 0) -> [[ -s /tmp/output.txt ]]
- Exam style:
```bash
#!/bin/bash

# Check if a directory exists using [[ ]]
if [[ -d /backup ]]; then
    echo "Backup directory is already there."
else
    mkdir /backup
fi

# Check a numerical argument using (( ))
if (( $1 > 5 )); then
    echo "The argument is greater than 5."
fi

# check if a file exists and is not empty (has size greater than 0),
# then empty it if it's not empty already.
if [[ -s file.txt ]]; then
    echo "file.txt is not empty, clearing it..."
    # this wipes the file and makes it's size exactly zero
    # : is the do nothing command, followed by a redirection.
    :> file.txt
    echo "file.txt has been cleared."
else
    echo "file.txt is empty."
fi
```
![conditionals image](conditionals.png)