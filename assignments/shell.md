## Build a Shell 🐚

# FIXME NOLIBC :)

The objective of this assignment is to build a UNIX style shell.

### Outcomes:

* Exercise your practical knowledge of Linux systems programming

* Practice generating more complex, well-formatted patchsets

### What to submit:

* Patch 1 creates `username/shell/Makefile` which builds an executable named `shell` as the default target

    * Make sure to have compiler warnings enabled (at least `-Wall` but ideally `-Wextra`, `-Wpedantic`, or even `-Weverything` if you use `clang`) and that your code compiles without warnings or errors

* Patch 2 implements your shell as source code files within `username/shell`

    * Your program should build and run with no memory leaks[^1]

    * You may find it prudent, and later expedient, to implement tests to ensure correctness of the code at each level

    * Use `git commit --amend` to fix mistakes in the most recent commit, and explore `man git-rebase` for more complex revision

* Don't forget a cover letter

* Submit your patches to `shell@COURSE_DOMAIN`

### Procedure:

0. This assignment will guide you through the process of building a shell by iteratively improving your code and adding more features, starting with a very simple program

    0. At the end you should have a shell that mostly works

0. You must make at least one commit per level completed, and label clearly at which commit you think you completed a particular level

0. You must implement up to level 7 at the minimum if you are an undergraduate, and up to level 9 if you are a graduate student

    0. If you want to go further for an extra challenge, see the list of ideas at the bottom

### Levels:

0. The shell prints a prompt, which consists of the absolute path the shell is currently
running in (see `man 3 getcwd`) followed by a $ and a space
(e.g. `/your/current/directory$ `). The shell then prints a new line and exits without
any user interaction.

0. The shell reads lines of user input, but doesn't do anything with them.
It just prints a new prompt before each line.
This loops until the shell gets EOF from user input (ctrl+d),
at which point it exits with code 0.

0. If the user types anything, the shell prints "Unrecognized command"
(but does not exit the loop). However, if the user just hits enter without typing anything,
no error message is printed.

0. The shell splits the line of input into pieces delimited by whitespace characters
(see `man 3 isspace`). Instead of just printing "Unrecognized command,"
the shell shall include the name of the program in the error message
(e.g. if the user types `cat shell.c`, the shell prints "Unrecognized command: cat").
If the user enters only whitespace, no error message is printed.

0. The shell supports a few builtin commands.
<br><br>
`exit` takes no arguments and closes the shell (return value of 0).
If arguments are provided, it prints an error message and does not exit the shell.
<br><br>
`cd` takes exactly one argument and changes the working directory of the shell process
to the provided path (see `man 2 chdir`). If an incorrect number of arguments are provided,
it prints usage info. If `chdir` fails, an error message with a description of
the errno (see `man 3 warn`) is printed.
<br><br>
`exec` takes _at least one_ argument and replaces the shell with an instance of
the executable file whose path is provided as the first argument (see `man 2 execve`).
It provides all its arguments as the `argv` for the called command, i.e.
`argv[0]` is the first argument to `exec`, `argv[1]` is the second argument
(if it exists), etc. If `execve` fails, an error message with a description of the errno
is printed, and the shell continues running.

0. The shell supports running executable files as commands within child processes.
If the first piece of the input looks like a relative or absolute path (contains a `/`),
a child process is created (see `man 2 fork`), and the command specified by the first
argument is executed within the child using the provided arguments (see `man 2 execve`),
similar to the `exec` builtin.
<br><br>
    If executing the command fails,
the child process prints an error message including a description of the errno
(don't forget to exit the child process).
The shell waits for the child to finish running before printing the next prompt
(see `man 2 waitpid`).
0. In the case that the user types something that isn't a path or a builtin,
before printing the unrecognized command error,
the shell checks whether a file with that name exists in each of the directories listed
in the `PATH` environment variable in order (see `man 3 getenv` and `man 2 stat`).
<br><br>
    If a file with that name is found, the search can stop and that file is executed with
arguments in a child process. Only if no file is found in any of the directories
should the unrecognized command error be printed.
<br><br>
You must do the path searching manually, and cannot rely on a member of the exec family that does path searching automatically, such as `execvp`.

0. Before processing the entered commands,
the shell performs home directory substitution on the pieces (command name or arguments)
that start with a `~`. It should first determine a username string by taking a substring
of the piece after the `~` until the end of the string or the first `/`,
whichever comes first.
    * If the username string is empty, then the `~` is replaced with the value of the `HOME` environment variable (see `man 3 getenv`)
    * If the username string is not empty, the shell attempts to locate the user with that username and, if successful, replaces the `~` and the username substring with their home directory (see `man 3 getpwnam`)
    * If `getpwnam` does not locate such a user, the shell leaves the piece unmodified
        * **Note:** in this case, `valgrind` may report a leak[^1]

0. As the shell is processing the commands and arguments, if it finds a `<` or `>`,
it skips any whitespace characters and attempts to treat the next piece of the input as
a filename for redirection. If there is no filename before the end of the string,
an error message is printed.
<br><br>

    A command can have more than one redirection.
If there are multiple of the same redirections, then the right-most one takes precedence.
A redirection causes the shell to open the corresponding file and replace one (or both)
of the child process' IO streams with the file descriptor of the open file
(see `man 2 dup`).

    * `>` replaces stdout
    * `<` replaces stdin

0. The shell supports the `|` (pipe) operator to chain multiple commands and their
inputs and outputs. Each command separated by a `|` is spawned as its own child process.
The shell creates a unidirectional pipe (see `man 2 pipe`) for each `|`, redirects
the stdout from the left command to the writing end of the pipe, and redirects the stdin
of the right command to the reading end of the pipe.
<br><br>
Any file redirections specified by the user take precedence over the implied
redirections from the `|`.

### Additional features for enthusiastic students with free time

* Add support for `;` to put multiple commands on one line, and `#` for comments

* Add support for the `&&` and `||` operators to chain commands based on their return codes

* Add support for special variables like `$?`, `$#`, `$*`, `$$`

* Add support for reading environment variables using `$NAME` (i.e. that would be replaced with `getenv("NAME")`)

* Add support for modifying environment variables using an `export` builtin

* Add support for background jobs (`jobs`, `fg`, `bg` builtins), and `&` for spawning a job in the background

* Add support for the `source` builtin to run commands from a file

* Add support for control flow operators (e.g. loops, conditionals, selection statements)

[Policies & Procedures](/faq/procedures.md)

[^1]: This is because the glibc implementation of `getpwnam` will open a shared library
with `dlopen` to try and find the specified user by other means if its search of
`/etc/passwd` is fruitless. This is a known issue, and the leak can be safely
ignored--but
if it really bothers you, feel free to use musl libc instead.

