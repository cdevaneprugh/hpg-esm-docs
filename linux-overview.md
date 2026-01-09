If you are new to HiPerGator or Linux I suggest you read through this document and utilize at least one of the resources linked below. While HiPerGator offers access to the system with a GUI or the ability to code via Jupyter notebooks, the Earth models will require you to use the command line exclusively, so it's important you are comfortable doing so.

# Contents

1. [General Linux Information](#general_info)
   1. [Basic Commands](#commands)
   2. [Text Editors](#text_editors)
   3. [External Resources](#resources)
2. [HiPerGator Specific Information](#hpg_info)
   1. [lmod](#lmod)
   2. [SLURM](#slurm)
   3. [Useful Commands](#useful_commands)
   4. [.bashrc File](#bashrc)

## 1. General Linux Information <a name="general_info"></a>

The Linux command line, also referred to as the terminal or shell, is a text-based interface for interacting with the Linux operating system. It allows users to execute commands and perform various tasks efficiently, without relying on a graphical user interface.

**Command Line Advantages:**

- **Efficiency:** The command line provides quick access to powerful tools and utilities, enabling you to perform tasks more efficiently compared to a GUI.
- **Automation:** Command line scripting allows for the automation of repetitive tasks, saving time and effort.
- **Flexibility:** Advanced users can customize and tailor their workflow to suit their specific needs, leveraging the extensive capabilities of the command line.

**Bash:**
Bash (Bourne Again Shell) is one of the most commonly used command line interpreters in Linux. It is the default shell for most Linux distributions (including HiPerGator) and provides a powerful scripting environment for automation and system administration tasks.

### 1.1 Basic Commands: <a name="commands"></a>

1. **pwd (Print Working Directory):**
   - Displays the current directory path.
   - Example: If you are in the "home" directory of the user "cooper," `pwd` would output `/home/cooper` in your terminal.
2. **ls (List):**
   - Lists files and directories in the specified directory (or current directory if one isn't given).
   - Example: `ls my_files` would list the contents of the directory "my_files."
3. **cd (Change Directory):**
   - Changes the current directory to the specified one.
   - Example: `cd dir1` will change your current working directory to `dir1` so long as that directory is in your current working path.
   - If no directory is specified, `cd` will move you to your `home` directory.
4. **touch:**
   - Creates an empty file.
   - Example: `touch filename.txt` creates an empty text file.
5. **mkdir (Make Directory):**
   - Similar to `touch`, but creates an empty directory instead. 
   - As with all these commands, you can use relative or absolute paths. For example `mkdir dir1` creates an empty directory named "dir1" in the current directory. `mkdir /home/cooper/dir1` creates "dir1" in Cooper's home directory.
6. **rm (Remove):**
   - Delete files or directories.
   - Example: `rm filename.txt` (for files), `rm -r directory` (for directories where the `-r` flag specifies to remove things recursively).
7. **cp (Copy):**
   - Copies files or directories.
   - Example: `cp file1 file2` (to copy file1 to file2), `cp -r dir1 dir2` (to copy dir1 to dir2 recursively).
8. **mv (Move):**
   - Moves or renames files or directories.
   - Example: `mv file1 file2` (to rename file1 to file2), `mv file1 /path/to/directory` (to move file1 to another directory).
9. **grep (Global Regular Expression Print):**
   - Searches for text patterns.
   - Example: `grep foo notes.txt` searches for the term "foo" in the file notes.txt.
10. __chmod__
   - Access and change permissions.
   - Example: ` chmod +x MY_SCRIPT.sh` would add the ability for a script you have written to be executed.
11. **man (Manual):**
    - Displays manual pages for commands, providing detailed information about their usage and options.
    - Example: `man ls` displays the manual page for the `ls` command.
12. **Piping (|):**
    - Allows the output of one command to be used as input to another command.
    - Example: `command1 | command2` (output of `command1` is used as input for `command2`).
    - Example: `ls Documents | grep my_file` Will search the "Documents" directory for files titled "my_file." You could also add the `-r` flag to the `grep` command to search within each file for the search term.

### 1.2 Text Editors<a name="text_editors"></a>

The three main text editors used on Linux are [Vim](https://en.wikipedia.org/wiki/Vim_(text_editor)), [Emacs](https://en.wikipedia.org/wiki/GNU_Emacs), and [Nano](https://en.wikipedia.org/wiki/GNU_nano). `nano` is the most basic, and easiest to use. `vim` and `emacs` are extremely customizable, but have a substantially steeper learning curve. Between `vim` and `emacs`, `vim` is arguably the most difficult to get comfortable with, but is powerful once you do. If you decide you are brave enough to try it, sites like [openvim](https://www.openvim.com/) or running the command `vimtutor` (which starts an interactive tutorial) on any Linux machine are good places to start. Additionally [here](https://vim.rtorr.com/) is a cheat sheet of `vim` commands that serve as a nice reference.

__WARNING__ The default text editor on Linux is usually `vim`. There may be a time when you accidentally open a file in `vim` and can't figure out how to exit the program. This may sound silly, but I promise you, the jokes about `vim` being confusing are endless in the Linux community. If you do find yourself trapped in `vim` hit the `esc` key, then type `:q!` and hit `enter`. This will exit whatever file you got yourself into without saving any changes. If you _would_ like to make a quick change to the file you got into, hit `esc`, followed by the `i` key to enter "insert" mode. At this point you can type whatever you need (use the arrow keys for navigation). To save, use `esc` to exit insert mode, then type `:wq` and hit `enter`. This will write the file (`w`) then quit out of the editor (`q`). 

### 1.3 External Resources<a name="resources"></a>

Here are some resources to get started using the Linux command line, and learn a bit more about what is going on "under the hood." You don't need to memorize everything in these links, but they can serve as a good starting point.

1. https://linuxjourney.com/

   The "Grasshopper" and "Journeyman" sections will give you a good chunk of background knowledge. I should note that because HiPerGator is not a traditional computer, some of the information (like package management) will not directly translate. However, it will help build a foundation of Linux knowledge and is certainly worth being exposed to. 

2. HiPerGator's [Introduction to the Linux Command Line](https://github.com/UFResearchComputing/Linux_training/blob/main/non_HiPerGator.md)

   A good demonstration of basic Linux commands you will be using on a daily basis, along with some practice exercises. I highly recommend going through this entire lesson and watching the accompanying [training video](https://help.rc.ufl.edu/doc/Training_Videos).

3. jlevy's [art of the command line](https://github.com/jlevy/the-art-of-command-line)

   This write up covers a wide range of commands and could serve as a good reference sheet. 

4. YouTube has plenty of tutorials. [Here's a good one](https://www.youtube.com/watch?v=s3ii48qYBxA) that serves as an introduction to the command line.

5. [ChatGPT](https://chat.openai.com/) is actually pretty good at `bash` scripting or serving as an interactive assistant.

## 2. HiPerGator Specific Information<a name="hpg_info"></a>

HiPerGator is not set up like a traditional personal computer. While Google and sites like [stack overflow](https://stackoverflow.com/) are great resources to use when you run into problems, remember to check HiPerGator's [documentation](https://help.rc.ufl.edu/doc/UFRC_Help_and_Documentation) first, as it will have information specific to our system.

Additionally, when using Linux, you should build the habit of using the commands `man` and the flag `--help` to get yourself out of trouble. `man` will access the built in documentation for Linux commands and programs that are on the system. For example, say you want to learn about the `ls` command, and all the options it has. You can type `man ls` to pull up the documentation page. You can also type `ls --help` for a list available arguments and options. 

If you are new to HiPerGator I would highly recommend reading through their [FAQ](https://www.rc.ufl.edu/documentation/frequently-asked-questions/),  [Getting Started](https://help.rc.ufl.edu/doc/Getting_Started) page, and watching all of the available training videos provided in the [documentation](https://help.rc.ufl.edu/doc/Training_Videos).

### 2.1 lmod<a name="lmod"></a>

HiPerGator uses a program called `lmod` to be able to have multiple versions of the same software installed for different use cases. [Here is a good article](https://www.admin-magazine.com/HPC/Articles/Lmod-Alternative-Environment-Modules?utm_source=ADMIN+Newsletter&utm_campaign=HPC_Update_31_Lmod_Alternative_to_Environment_Modules_2013-01-30) that gives an overview of module systems and how they work. While the HiPerGator [documentation](https://help.rc.ufl.edu/doc/Modules) has great info that I suggest you read, the full documentation for `lmod` can be found [here](https://lmod.readthedocs.io/en/latest/).

### 2.2 SLURM<a name="slurm"></a>

Unlike a personal computer, HiPerGator is a collection of connected computers (called nodes) that share resources. The node that you login to (unsurprisingly called a login node) has a limited amount of resources, and is generally used for tasks like writing code, file management, and very light jobs. As such, you will submit more computationally "expensive" jobs to the `SLURM` scheduler to run on the dedicated compute nodes.

Please read the page on our scheduler [here](https://help.rc.ufl.edu/doc/HPG_Scheduling). The full documentation for the `SLURM` scheduler can be found [here](https://slurm.schedmd.com/documentation.html).

### 2.3 Useful Commands<a name="useful_commands"></a>

Here are some miscellaneous useful commands for navigating and managing your environment in HiPerGator:

1. `env | grep HPC | sort`

   - Lists all environment variables related to HiPerGator and sorts them alphabetically.

   - Useful for checking loaded modules, system paths, and predefined environment settings.

2. `tree | less`

   - The `tree` command displays a directory structure in a hierarchical format, which is helpful for visualizing project layouts.

   - The `less` command allows you to scroll through large outputs without flooding your terminal.

3. SLURM Resource Monitoring and Job Management

   - `squeue -u $USER`: This lists all jobs associated with your username, showing job IDs, status, and resources requested. Can also use `$GROUP` to see all current jobs associated with the group.

   - `scontrol show job JOB_ID`: Replace `JOB_ID` with the actual job ID from `squeue` to see job details.
   - `scancel JOB_ID`: This will stop a running job. Replace JOB_ID with `$USER` to stop all user jobs.

### 2.4 .bashrc File<a name="bashrc"></a>

The `.bashrc` file is a configuration script that runs automatically whenever you open a new shell session. It is located in your home directory (`~/.bashrc`) and is used to set environment variables, define aliases, and customize your shell environment. Here are a few things that are helpful to add to your `.bashrc` file, but not required.

#### **Adding Environment Variables**

Environment variables store system-wide or user-specific configurations. You can add variables to `.bashrc` like this:

```bash
export MY_ENV_VAR="my_value"
```

To define your Earth System Models (ESM) directory, group name, and case output directories add the following to the end of your `.bashrc` file:

```bash
export ESM_DIR="/blue/gerber/earth_models"
export GROUP="gerber"
export CASE_OUTPUT_DIR="/blue/$GROUP/$USER/cases"
```

#### **Reloading `.bashrc`**

Whenever you modify your `.bashrc` file, you need to reload it for the changes to take effect.
To apply the changes without logging out:

```bash
source ~/.bashrc
```

Alternatively, you can log out and log back in to ensure the new environment variables are loaded.
