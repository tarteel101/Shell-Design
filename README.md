# Design And Implementation Of A Shell Program

This project implements a custom shell program, a fundamental component of any operating system. A shell acts as a command-line interface that allows users to interact with the system by executing commands, launching programs, and automating tasks.

---

## Table of Contents
1. [Overview](#overview)
2. [Features](#features)
3. [Usage](#usage)
4. [Code Explanation](#code-explanation)
    - [Shell Initialization](#shell-initialization)
    - [Command Execution](#command-execution)
    - [Batch Mode Handling](#batch-mode-handling)
5. [Contributors](#contributors)

---

## Overview


This project is a simple custom shell program that works as a command-line interface, allowing users to run commands and interact with the system. The shell can handle multiple commands, execute them, and manage processes efficiently.

It can be operated in two modes:

- Interactive Mode:
   Users type commands directly, and the shell executes them immediately.


- Batch Mode:
  Commands are read from a file and executed automatically.



The shell uses key system features like creating processes (fork()), running programs (execvp()), and waiting for tasks to finish (wait()). It also includes error handling to guide users when something goes wrong.

This project helps demonstrate how operating systems manage commands, processes, and user interaction.

---

## Features

- Command Execution: Supports execution of multiple commands separated by semicolons (;).
- Custom Built-in Commands: Includes commands like quit to exit the shell.
- Pipe Handling: Processes commands effectively, ensuring smooth execution.
- User-Friendly Interface: Offers an interactive and visually appealing experience.
- Batch Mode: Executes commands from a file for automated tasks.

---
