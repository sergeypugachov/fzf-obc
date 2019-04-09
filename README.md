# FZF-OBC (FZF Over Bash Complete)

## What is fzf-obc

A bash completion script intend to add [fzf](https://github.com/junegunn/fzf) over all known bash completion functions on your system with minimal modifications on original completion scripts.  
It is a replacement to the completion script natively provided by [fzf](https://github.com/junegunn/fzf) who replace original completion functions with its own (and create some behavior originally well implemented into original completion scripts).

## Demo

[![asciicast](https://asciinema.org/a/g41AU7DB2iJtp5plhswJnczeS.svg)](https://asciinema.org/a/g41AU7DB2iJtp5plhswJnczeS?speed=3)



## Prerequisites

fzf-obc require to **not use** the default fzf bash complete script provided with fzf since it change the default complete functions too and fzf-obc need to be over default complete functions.  
fzf-obc should be the last completion function loaded into your profile.

- In case you install fzf for the first time, follow the [official instructions](https://github.com/junegunn/fzf#using-git) but don't forget to install fzf without its own completion script:

   `$ ./install --no-completion`

     instead of

   `$ ./install`

- In case fzf is already installed and the default fzf completion script is activated, deactivate it inside your fzf config (usually `~/.fzf.bash` or `~/.config/fzf/fzf.bash`) by commenting the auto-completion section.

  ```shell
  # Auto-completion
  # ---------------
  #[[ $- == *i* ]] && source "~/.local/opt/fzf/shell/completion.bash" 2> /dev/null
  ```

## Install

```shell
$ INSTALL_PATH=~/.local/opt/fzf-obc
$ mkdir -p ${INSTALL_PATH}
$ git clone https://github.com/rockandska/fzf-obc ${INSTALL_PATH}
$ source ${INSTALL_PATH}/fzf-obc.bash && echo "source ${INSTALL_PATH}/fzf-obc.bash" >> ~/.bashrc
```

## Details

### What fzf-obc does

- update original complete definition with a wrapper who :
  - call the original completion script (the one in charge to populate `COMPREPLY`)
  - if a specific `__fzf_obc_post_[completion_script_name]` function exist, run it (used to add / modify `COMPREPLY` generated by the original function)
  - a default function in charge to update `COMPREPLY` with fzf filtering
  - if a specific `__fzf_obc_trap_[function_name]`  function exist, add a trap on the "RETURN" signal to the corresponding private functions. Those traps are used to do some modifications who can't be done after completion functions used by `complete`.
- overwrite `_filedir` / `_filedir_xspec` original bash complete functions in charge of the files / dir lookup for those reasons :
  - compgen used in original bash complete functions doesn't handle newline in filenames
  - having a unique function for the files/dir is simpler to maintain than a trap

fzf-obc startup sequence is :

- add `_longopt` as `fzf` completion script if there is not already one defined
- clean previous `fzf-obc` load (rollback complete definition, remove wrapper functions, remove post complete functions, remove traps)
- source default fzf-obc `traps ` / `posts ` functions then load functions from paths specified in `$FZF_OBC_PATH`
- take a look at already bash complete functions defined and add wrapper to them if not already done.
- Add traps to private complete functions if they're exists and not already add.

### What fzf-obc does not

fzf-obc will only add fzf filtering over existing functions and is not magic.  
If a specific post complete function exists ( `__fzf_obc_post__kill` for example ), it will only be functional if the `_kill` function exists.  
If the complete function for kill command is not `_kill`, you will see no changes in the `kill` completion.  
Same for traps, if a specific trap exist (`__fzf_obc_trap__filedir`) , it will only be add if `_filedir` exists.  

### Pros

- Compatible with almost known bash completion functions without efforts
- User configuration / functions / traps loader

### Cons

- young project (some bugs not found yet, need to be test by other users, functions rewrite, ...)
- results are not shown as they arrive (live streaming). Not a real deal if you not using globs to search inside a directory with too many depths.
- **/!\ during search using globs inside directories with too many depth, the current prompt is freeze until the end of the search even if pressing CTRL+C (need to find a fix)**

## Basic / Globs / Specific completion

### Basic

This behavior is the default one.  
[fzf](https://github.com/junegunn/fzf) wil be trigger over the default bash `COMREPLY` to let you filter the result easily.  
The default binding to select an entry is the key \<TAB\> ( you already have your finger on it right ? )

### Globs

Like the original [fzf](https://github.com/junegunn/fzf) completion script, you could use recursive search with some bash complete functions by adding `**` at the end of your path, then hit key \<TAB\> to recursively looking for corresponding files / path / directories.

- **Works with complete functions who use for path/files lookup :**
  - **_filedir**
    - cd
    - ls
    - and more than 400 commands
  - **_filedir_xspec**
    - vi(m)
    - bunzip2
    - lynx
    - and more than 140 commands
- **If there is no results, you will be aware by seeing the "\*\*" removed from your current search**
- **Be cautious that using this capability on huge directories could freeze your shell for ages without be able to cancel it with CTRL-C (need to fix it)**
- **The bindings with globs are different ( \<TAB\>, \<SHIFT-TAB\> are used to (un)select multiples results and \<ENTER\> to validate )**

### Specific completion

Some bash complete reply could be partially / totally override if specific functions exists.  
A good example is how `_kill` is surcharged to display a nice preview of the process but let the other default complete results untouched.

## Configuration

Default fzf-obc configuration could be customize by settings some environment variables with your own needs and are describe bellow.

- `$FZF_OBC_PATH`
  - default: 
  - additional paths containing specific complete traps / functions to load
  - if using multiples paths, paths need to be separate by `:` and will be load in the order they appear
- `$FZF_OBC_COLORS`
  - default: 1
  - add colors similar to `ls`
  - **doesn't handle broken symlink for now**
- `$FZF_OBC_HEIGHT`
  - default: `40%`
  - height of the fzf filtering windows
- `$FZF_OBC_EXCLUDE_PATH`
  - default: `.git:.svn`
  - paths to exclude from the completion results
  - if using multiples paths, paths need to be separate by `:`
  - **works only with globs completion**
- `$FZF_OBC_OPTS`
  - default: `--select-1 --exit-0`
  - options used for basic completion
- `$FZF_OBC_BINDINGS`
  - default: `--bind tab:accept`
  - bindings options used for basic completion
- `$FZF_OBC_GLOBS_MAXDEPTH`
  - default: `999999`
  - maximum depth to look when using globs completion
- `$FZF_OBC_GLOBS_OPTS`
  - default: `-m --select-1 --exit-0`
  - options used for globs completion
- `$FZF_OBC_GLOBS_BINDINGS`
  - default:
  - bindings options used for globs completion

## Override fzf-obc functions

If you need / want to override some fzf-obc functions add the paths containing the functions to `$FZF_OBC_PATH` ( '`:`' as separator) **just before sourcing** `fzf-obc`.

Example:

```
$ echo "FZF_OBC_PATH='~/.local/opt/myfzfcomplete:~/.local/opt/fzf-test'" >> ~/.bashrc
```

### Functions

- `_fzf_obc`
  - main function
  - load :
    - __fzf_obc_cleanup
    - __fzf_obc_init_vars
    - __fzf_obc_load
    - __fzf_obc_update_complete
  - define complete function for fzf if not already done
- `__fzf_obc_init_vars`
  - define default variables
- `__fzf_obc_load`
  - load default / users defined functions
- `__fzf_obc_update_complete`
  - add the wrapper to the complete function
- `__fzf_obc_globs_exclude`
  - create exclusion string for __fzf_obc_search
- `__fzf_obc_search`
  - recursively search for paths / dirs / files
  - return list separate by $'\0'
- `__fzf_obc_expand_tilde_by_ref`
  - copy of original _expand_tilde_by_ref
- `__fzf_obc_tilde`
  - copy of original _tilde
- `__fzf_obc_cmd`
  - fzf command to run
- `__fzf_obc_read_compreply`
  - default command used to read COMPREPLY
  - use `$FZF_OBC_GLOBS_OPTIONS` if `**` detected in `$cur`
- `__fzf_obc_post__kill`
  - post function to `_kill`
- `__fzf_obc_sort_cmd`
  - sort command used to sort results from `compgen` / `find` results
- `__fzf_obc_post__completion_loader`
  - post function to `_completion_loader`
  - reload `fzf-obc` to add fzf-obc wrapper new complete functions automatically loaded by `_completion_loader`