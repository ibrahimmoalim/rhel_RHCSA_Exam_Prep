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
