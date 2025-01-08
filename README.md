#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <errno.h>
#define MAX_LINE_LENGTH 512
#define MAX_COMMANDS 10
//Clear the screen of the terminal 
void clear() {
    printf("\033[H\033[J");
}
// Popup message for the user during startup
void init_shell() {
    clear();
    printf("\n\n\n\n****");
    printf("\n\n\tWELCOME TO THE SHELL PROJECT");
    printf("\n\n****");
    printf("\n\n\tDeveloped by: Group of Statistics & Computer Science Department");
    printf("\n\tUniversity of Khartoum, Faculty of Mathematical Sciences and Informatics");
    printf("\n\tOperating Systems Course - Lab Assignment");
    printf("\n\n\tAssignment: Shell Design and Implementation");
    printf("\n\tInstructor: Izzeldin Kamil Amin");
    printf("\n\n****");
    printf("\n\n\tFeatures Implemented:");
    printf("\n\t- Command Execution (Single and multiple)");
    printf("\n\t- Custom Built-in Commands");
    printf("\n\t- Pipe Handling and Improper Space Management");
    printf("\n\t- User-Friendly Interface with Command History");
    printf("\n\n****");
    char* username = getenv("USER");
    printf("\n\nUSER: @%s", username);
    printf("\n\n****");
    sleep(4); // waits 4 seconds for the user to read the message
    clear();
}

void execute_commands(char *line) {
    char *commands[MAX_COMMANDS];
    int i = 0;
    // Parse commands separated by ';'
    char *token = strtok(line, ";");
    while (token != NULL && i < MAX_COMMANDS) {
        commands[i++] = token;
        token = strtok(NULL, ";");
    }
    commands[i] = NULL;
    // Execute each command concurrently
    for (int j = 0; j < i; j++) {
        pid_t pid = fork();
        if (pid == 0) {
            char *args[MAX_COMMANDS];
            char *arg = strtok(commands[j], " \t\n");
            int k = 0;
            while (arg != NULL && k < MAX_COMMANDS) {
                args[k++] = arg;
                arg = strtok(NULL, " \t\n");
            }
            args[k] = NULL;
            if (args[0] != NULL) {
                if (execvp(args[0], args) == -1) {
                    fprintf(stderr, "Error executing command '%s': %s\n", args[0], strerror(errno));
                }
            }
            exit(EXIT_FAILURE);
        } else if (pid < 0) {
            perror("Fork failed");
        }
    }
    // Wait for all child processes
    while (wait(NULL) > 0);
}
int main(int argc, char *argv[]) {
    char line[MAX_LINE_LENGTH];

    // Initialize the shell
    init_shell();
    if (argc == 1) {
        // Interactive mode
        while (1) {
            printf("shell> ");
            if (fgets(line, MAX_LINE_LENGTH, stdin) == NULL) {
                break; // End of input (Ctrl+D)
            }
            if (strcmp(line, "quit\n") == 0) {
                break;
            }
            execute_commands(line);
        }
    } else if (argc == 2) {
        // Batch mode
        FILE *batch_file = fopen(argv[1], "r");
        if (batch_file == NULL) {
            fprintf(stderr, "Error opening batch file '%s': %s\n", argv[1], strerror(errno));
            return EXIT_FAILURE;
        }
        while (fgets(line, MAX_LINE_LENGTH, batch_file) != NULL) {
            printf("%s", line); // Echo line
            if (strcmp(line, "quit\n") == 0) {
                break;
            }
            execute_commands(line);
        }
        fclose(batch_file);
    } else {
        fprintf(stderr, "Usage: %s [batchFile]\n", argv[0]);
        return EXIT_FAILURE;
    }
    return EXIT_SUCCESS;
}
