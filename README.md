# Design And Implementation Of A Shell Program

This project implements a basic command-line interpreter (shell) that can run commands both interactively and in batch mode, while handling multiple commands simultaneously. The shell will operate by creating a child process to execute the commands entered by the user and will manage processes, concurrency, and basic error handling.

---

## Table of Contents
1. [Overview.](#overview)
2. [Features.](#features)
3. [Code Specifications.](#code-specifications)
4. [Testing.](#testing)
5. [Resources.](#Resources)
6. [Contributers.](#Contributers)

---

## Overview

This projec implements a shell program that works as a command-line interface, allowing users to run commands and interact with the system. The shell can handle multiple commands, execute them, and manage processes efficiently.

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
#define MAX_LINE_LENGTH 512 
#define MAX_COMMANDS 10      
#define clear()
printf("\033[H\033[J") // Clears the screen using ANSI escape codes

```

- <stdio.h>: Lets us print messages or read input (e.g., printf and fgets).
- <stdlib.h>: Helps with things like exiting the program (exit) and handling memory.
- <string.h>: Allows us to work with text, like splitting commands or comparing strings.
- <unistd.h>: Gives access to system features like running processes.
- <sys/types.h> and <sys/wait.h>: Allow us to manage child processes (like running a command).
- <errno.h>: Helps us understand what went wrong when something breaks.
  
- MAX_LINE_LENGTH: This defines the maximum length (512 characters) of a single command line.
- MAX_COMMANDS: This defines how many separate commands we can handle at once (up to 10).
---
This C code consists of **three functions**, each is meant to implement specific and certain features and functionalities.

---
#### 1. The initialize() Function
```C
void initialize() {
    clear();
    printf("\n\n\n\n****");
    printf("\n\n\tWELCOME TO THE SHELL PROJECT");
    printf("\n\n****");
    printf("\n\tDeveloped by: Group of Statistics & Computer Science Department");
    printf("\n\tUniversity of Khartoum, Faculty of Mathematical Sciences and Informatics");
    printf("\n\n****");
    printf("\n\n\tFeatures Implemented:");
    printf("\n\t-User Command Execution");
    printf("\n\t-Processes Management");
    printf("\n\t- Error Handling");
    printf("\n\t- User-Friendly Interface");
    printf("\n\n****");
    sleep(3); // Display the message briefly
    clear();
}
```
- **Purpose:** This function is used to display a welcome message when the shell starts.
- **Clear the screen:** It uses clear() to wipe the terminal screen to make the welcome message stand out.
- **Display information:** Shows details about the shell's features and development team.
- **Pause for reading:** It sleeps for 3 seconds to give the user time to read the message before clearing the screen again.

---

#### 2. The execute() Function
```C
void execute(char *line) {
    char *commands[MAX_COMMANDS]; // Array to store multiple commands separated by ;
    int i = 0;

    // Split the input line into individual commands using ; as a delimiter
    char *token = strtok(line, ";");
    while (token != NULL && i < MAX_COMMANDS) {
        commands[i++] = token; // Store each command in the array
        token = strtok(NULL, ";"); // Get the next command
    }
    commands[i] = NULL; // Mark the end of the array

    // Execute each command
    for (int j = 0; j < i; j++) {
        pid_t pid = fork(); // Create a child process for each command
        if (pid == 0) { // In the child process
            char *args[MAX_COMMANDS]; // Array to store the command arguments
            char *arg = strtok(commands[j], " \t\n"); // Split the command into arguments
            int k = 0;
            while (arg != NULL && k < MAX_COMMANDS) {
                args[k++] = arg; // Store each argument
                arg = strtok(NULL, " \t\n"); // Get the next argument
            }
            args[k] = NULL; // End of arguments

            if (args[0] != NULL) {
                if (execvp(args[0], args) == -1) { // Execute the command
                    // If execution fails, display an error
                    fprintf(stderr, "Error executing '%s': %s\n", args[0], strerror(errno));
                    exit(EXIT_FAILURE); // Exit with failure
                }
            }
            exit(EXIT_SUCCESS); // Exit child process after executing the command
        } else if (pid < 0) {
            // Fork failed, show error
            perror("Fork failed");
        }
    }
    // Parent does not wait here, allowing simultaneous execution
}
```
- **Purpose:** This function takes the user's input (which may contain multiple commands), splits it into individual commands, and executes them.
- **Splitting Commands:** The input string is split into separate commands by semicolons (;), and each command is processed individually.
- **Forking child processes:** For each command, a child process is created using fork(), allowing commands to run in parallel.
- **Argument parsing:** Each command is further split into its arguments (separated by spaces or tabs).
- **Executing Commands:** Each command is executed using execvp(), replacing the child process with the specified command.
- **No waiting:** The parent process doesn't wait for child processes to finish, allowing them to run simultaneously.
- **Error handling:** If the execvp() fails, an error message is displayed, and the child process exits.

---
#### 3. The main() function
```C
int main(int argc, char *argv[]) {
    char line[MAX_LINE_LENGTH]; // Buffer to store user input or file commands

    initialize(); // Display the welcome message and clear the screen

    if (argc == 1) {
        // Interactive mode: Commands are entered directly by the user
        while (1) {
            printf("shell> "); // Display the shell prompt
            if (fgets(line, MAX_LINE_LENGTH, stdin) == NULL) { // Read user input
                printf("\nExiting shell.\n");
                break; // Exit the shell on EOF (Ctrl+D)
            }
            line[strcspn(line, "\n")] = 0; // Remove the newline character from input
            if (strcmp(line, "quit") == 0) { // Check for the "quit" command
                break; // Exit the shell
            }
            if (strlen(line) >= MAX_LINE_LENGTH - 1) { // Check for overly long commands
                fprintf(stderr, "Error: Command too long.\n");
                continue; // Skip execution of this input
            }
            execute(line); // Execute the command(s)
        }
    } else if (argc == 2) {
        // Batch mode: Commands are read from a file
        FILE *batch_file = fopen(argv[1], "r"); // Open the file
        if (batch_file == NULL) {
            // File could not be opened, print an error message
            fprintf(stderr, "Error opening file '%s': %s\n", argv[1], strerror(errno));
            return EXIT_FAILURE; // Exit with failure
        }

        // Read and execute each line from the file
        while (fgets(line, MAX_LINE_LENGTH, batch_file) != NULL) {
            line[strcspn(line, "\n")] = 0; // Remove the newline character
            printf("%s\n", line); // Display the command
            if (strcmp(line, "quit") == 0) { // Check for the "quit" command
                break; // Stop reading the file
            }
            execute(line); // Execute the command(s)
        }
        fclose(batch_file); // Close the file
    } else {
        // Invalid arguments: Display usage instructions
        fprintf(stderr, "Usage: %s [batchFile]\n", argv[0]);
        return EXIT_FAILURE; // Exit with failure
    }

    return EXIT_SUCCESS; // Exit successfully
}
```
- **Purpose:** This is the main entry point of the shell program, controlling how it functions according the arguments passed.
- **Interactive Mode:** If no batch file is provided, the user is prompted to input commands interactively. It reads input, processes commands, and executes them in parallel.
- **Batch Mode:** If a batch file is provided, it reads commands from the file and executes them one by one. Commands in the file are processed similarly to interactive commands but are read from the file.
- **Error handling:** It checks if the file exists and handles the case of wrong number of arguments.
- **Exit condition:** The program exits when the user enters "quit" or reaches the end of the file.
- **Buffering Input:** The program reads the entire line of input, removing newline characters before processing it.

---
## Testing
How to Test the Code:

- **Basic Commands**: Test simple commands like ls, pwd, or date.
**Expected Output**: Output of the respective commands.

- **Multiple Commands:** Enter commands separated by ; (e.g., ls; pwd).
**Expected Output:** Results of all commands are executed simultaneously.

- **Invalid Command:** Enter a non-existent command as when to misspell a command.
**Expected Output:** Error executing 'misspelled-command': <error message>

- **Quit Command:** Type quit to exit the shell.
**Expected result:** Shell exits.

- **Batch File:** Create a file with multiple commands and pass it as an argument (e.g., ./shell my_commands.txt).
**Expected result:** All commands in the file are executed simultaneously.

- **Long Command:** Input a command longer than MAX_LINE_LENGTH.
**Expected Output:** Error: Command too long.

---
## Resources
References Used for Designing and Implementing the Shell Program in C

The following references were utilized in the design and implementation of the shell program in C. These resources provided valuable insights into the core concepts of shell programming, C programming techniques, process management, system calls, and best practices for building efficient and functional shells. The materials helped guide the development process, from basic command execution to handling user inputs and system interactions effectively.

1. Tanenbaum, A. S., & Bos, H. (2015). Modern Operating Systems (5th ed.). Pearson.
This book provided essential insights into process management, system calls, and shell operations, forming the foundation for the design and execution of our shell program.

2. Linux Foundation. (2019). Linux Command Line and Shell Scripting Bible (4th ed.). Wiley.

3. Lindholm, T. (2012). Advanced C Programming. O'Reilly Media.

---
## Contributers

