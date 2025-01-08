# Design And Implementation Of A Shell Program

This project will implement a basic command-line interpreter (shell) that can run commands both interactively and in batch mode, while handling multiple commands simultaneously. The shell will operate by creating a child process to execute the commands entered by the user and will manage processes, concurrency, and basic error handling.

---

## Table of Contents
1. [Overview.](#overview)
2. [Features.](#features)
3. [Code Specifications.](#code-specifications)
4. [Testing.](#testing)

---

## Overview

This project is a custom shell program that works as a command-line interface, allowing users to run commands and interact with the system. The shell can handle multiple commands, execute them, and manage processes efficiently.

It can be operated in two modes:

 - Interactive Mode:
  Users type commands directly, and the shell executes them immediately.

- Batch Mode:
 Commands are read from a file and executed automatically.

The shell uses key system features like creating processes (fork()), running programs (execvp()), and waiting for tasks to finish (wait()). It also includes error handling to guide users when something goes wrong.

---

## Features

- Command Execution: Supports execution of multiple commands separated by semicolons (;).
- User-Friendly Interface.
- Interactive Mode: Users type commands directly, and the shell executes them immediately.
- Batch Mode: Commands are read from a file and executed automatically.
- Error Handling.

---

## Code Specifications

- #### Headings
```C
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <errno.h>
```

- <stdio.h>: Lets us print messages or read input (e.g., printf and fgets).
- <stdlib.h>: Helps with things like exiting the program (exit) and handling memory.
- <string.h>: Allows us to work with text, like splitting commands or comparing strings.
- <unistd.h>: Gives access to system features like running processes.
- <sys/types.h> and <sys/wait.h>: Allow us to manage child processes (like running a command).
- <errno.h>: Helps us understand what went wrong when something breaks.

---

```C
#define MAX_LINE_LENGTH 512
#define MAX_COMMANDS 10
```
- MAX_LINE_LENGTH: This defines the maximum length (512 characters) of a single command line.
- MAX_COMMANDS: This defines how many separate commands we can handle at once (up to 10).

---
This C code consists of **three functions**, each is meant to implement specific and certain features and functionalities.

---

#### 1. The init_shell() Function
```
void init_shell() {
    clear();
    printf("\n\n\n\n****");
    printf("\n\n\tWELCOME TO THE SHELL PROJECT");
    printf("\n\n****");
    printf("\n\tDeveloped by: Group of Statistics & Computer Science Department");
    printf("\n\tUniversity of Khartoum, Faculty of Mathematical Sciences and Informatics");
    printf("\n\n****");
    sleep(3); // Pause to display the message
    clear();
}
```
This function do the following:

- Displays a welcome message about the project and who developed it.
- Shows the username of the person running the shell.
- Pauses for 3 seconds (so you can read the welcome message) using sleep(3).
- After the pause, it clears the screen again, getting ready for the user to enter commands.
---

#### 2. The execute_commands() Function
```C
void execute_commands(char *line) {
    char *commands[MAX_COMMANDS];
    int i = 0;

    // Split input by ';' into separate commands
    char *token = strtok(line, ";");
    while (token != NULL && i < MAX_COMMANDS) {
        commands[i++] = token;
        token = strtok(NULL, ";");
    }
    commands[i] = NULL;

    // Execute each command
    for (int j = 0; j < i; j++) {
        pid_t pid = fork();
        if (pid == 0) {
            // Parse individual command arguments
            char *args[MAX_COMMANDS];
            char *arg = strtok(commands[j], " \t\n");
            int k = 0;
            while (arg != NULL && k < MAX_COMMANDS) {
                args[k++] = arg;
                arg = strtok(NULL, " \t\n");
            }
            args[k] = NULL;

            // Execute the command
            if (args[0] != NULL) {
                if (execvp(args[0], args) == -1) {
                    fprintf(stderr, "Error executing '%s': %s\n", args[0], strerror(errno));
                }
            }
            exit(EXIT_FAILURE);
        } else if (pid < 0) {
            perror("Fork failed");
        }
    }

    // Wait for all child processes to finish
    while (wait(NULL) > 0);
}
```
This function do the following:

- Splits the input into different commands.
- For each command, it creates a "child process" using fork().
- It then splits the command into arguments.
- Executes the command using execvp(). If the command fails, it prints an error message.
- After all the child processes run their commands, the parent process waits for them to finish using wait().

---
#### 3. The main() function
```C
 main() Function

int main(int argc, char *argv[]) {
    char line[MAX_LINE_LENGTH];

    // Initialize the shell
    init_shell();

    if (argc == 1) {
        // Interactive mode: Read commands from the user
        while (1) {
            printf("shell> ");
            if (fgets(line, MAX_LINE_LENGTH, stdin) == NULL) {
                break; // Exit on EOF (Ctrl+D)
            }
            if (strcmp(line, "quit\n") == 0) {
                break;
            }
            execute_commands(line);
        }
    } else if (argc == 2) {
        // Batch mode: Execute commands from a file
        FILE *batch_file = fopen(argv[1], "r");
        if (batch_file == NULL) {
            fprintf(stderr, "Error opening file '%s': %s\n", argv[1], strerror(errno));
            return EXIT_FAILURE;
        }

        while (fgets(line, MAX_LINE_LENGTH, batch_file) != NULL) {
            printf("%s", line); // Display the command
            if (strcmp(line, "quit\n") == 0) {
                break;
            }
            execute_commands(line);
        }
        fclose(batch_file);
    } else {
        // Display usage instructions for incorrect arguments
        fprintf(stderr, "Usage: %s [batchFile]\n", argv[0]);
        return EXIT_FAILURE;
    }

    return EXIT_SUCCESS;
}
```
This code initializes the shell with the welcome message by calling the init_shell() function.

- #### Interactive Mode (No Arguments):
If you just run the program like this: ./shell, it starts in interactive mode.
You’ll see a prompt (shell>) where you can type commands one by one, and the shell will run them.
The user can type quit to exit, or hit ctrl-D (end of input).

- #### Batch Mode (File as an Argument):
If you run the program like this: ./shell commands.txt, it starts in batch mode.
The program will read commands from the file (commands.txt), run them, and show the results.
It will run until it catches a quit command within the file.

- #### Incorrect Usage:
If the user enters wrong number of arguments, a message will appear that shows how to fix the error:
"Usage: ./shell [batchFile]"

---

What to Expect in Tests

1. Interactive Mode:

When you run the program with no arguments, you’ll see the welcome message.

The shell will then prompt you for commands (e.g., shell> ).

You can type commands like ls, pwd, etc.

After typing quit, the program exits.



2. Batch Mode:

If you run the program with a file containing commands (e.g., ./shell commands.txt), it will read and execute each command from the file.

The shell will show the results of each command.

When the file contains quit, the program will stop.




Example: If commands.txt contains:

ls
pwd
quit

The output will show:

$ ls
(Your directory contents)
$ pwd
/your/current/directory

After it reads quit, the shell will stop.


---

Summary

This program is a simple shell that lets you run commands either by typing them directly or by reading from a file. It supports basic features like command execution and handles errors if something goes wrong. The flow is interactive, where you type commands, or batch, where commands are read from a file.
