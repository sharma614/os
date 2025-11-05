# os
Here are the C programs and shell scripts for all the items on your list, each with its explanation and code (without in-code comments, as requested).

-----

### 1\. FCFS Scheduling Algorithm

#### 💡 Explanation

First-Come, First-Served (FCFS) is a non-preemptive scheduling algorithm. Processes are served in the exact order they arrive in the ready queue. The program calculates completion time, turnaround time (`CT - AT`), and waiting time (`TAT - BT`) for each process. To work correctly, it first sorts all processes by their arrival time.

#### 💻 C Code

```c
#include <stdio.h>

struct Process {
    int pid;
    int at; 
    int bt; 
    int ct; 
    int tat;
    int wt; 
};

void sortByArrival(struct Process proc[], int n) {
    for (int i = 0; i < n - 1; i++) {
        for (int j = 0; j < n - i - 1; j++) {
            if (proc[j].at > proc[j + 1].at) {
                struct Process temp = proc[j];
                proc[j] = proc[j + 1];
                proc[j + 1] = temp;
            }
        }
    }
}

void findTimes(struct Process proc[], int n) {
    int currentTime = 0;
    float total_wt = 0;
    float total_tat = 0;

    sortByArrival(proc, n);

    for (int i = 0; i < n; i++) {
        if (currentTime < proc[i].at) {
            currentTime = proc[i].at;
        }

        proc[i].ct = currentTime + proc[i].bt;
        proc[i].tat = proc[i].ct - proc[i].at;
        proc[i].wt = proc[i].tat - proc[i].bt;

        currentTime = proc[i].ct;

        total_wt += proc[i].wt;
        total_tat += proc[i].tat;
    }

    printf("PID\tArrival\tBurst\tCompletion\tTurnaround\tWaiting\n");
    for (int i = 0; i < n; i++) {
        printf("P%d\t%d\t%d\t%d\t\t%d\t\t%d\n",
               proc[i].pid, proc[i].at, proc[i].bt,
               proc[i].ct, proc[i].tat, proc[i].wt);
    }

    printf("\nAverage Waiting Time:    %.2f\n", total_wt / n);
    printf("Average Turnaround Time: %.2f\n", total_tat / n);
}

int main() {
    int n;
    printf("Enter the number of processes: ");
    scanf("%d", &n);
    struct Process proc[n];

    for (int i = 0; i < n; i++) {
        proc[i].pid = i + 1;
        printf("\nProcess P%d:\n", proc[i].pid);
        printf("  Enter Arrival Time: ");
        scanf("%d", &proc[i].at);
        printf("  Enter Burst Time: ");
        scanf("%d", &proc[i].bt);
    }

    findTimes(proc, n);
    return 0;
}
```

-----

### 2\. Round Robin Scheduling Algorithm

#### 💡 Explanation

Round Robin (RR) is a preemptive scheduling algorithm. Each process is given a small time slice called a "Time Quantum." The process runs for this quantum, and if it's not finished, it's preempted and placed at the end of the ready queue. This code maintains a `remaining_bt` for each process and simulates this circular, time-sliced execution using a ready queue.

#### 💻 C Code

```c
#include <stdio.h>
#include <stdbool.h>

struct Process {
    int pid;
    int at; 
    int bt; 
    int rt; 
    int ct; 
    int tat;
    int wt; 
};

int queue[100];
int front = -1, rear = -1;

void enqueue(int pid) {
    if (rear == 99) return;
    if (front == -1) front = 0;
    rear++;
    queue[rear] = pid;
}

int dequeue() {
    if (front == -1 || front > rear) return -1;
    return queue[front++];
}

bool isInQueue(int pid, int n) {
    for (int i = front; i <= rear; i++) {
        if (queue[i] == pid) return true;
    }
    return false;
}

int main() {
    int n, time_quantum;
    float total_wt = 0, total_tat = 0;

    printf("Enter the number of processes: ");
    scanf("%d", &n);
    struct Process proc[n];

    printf("Enter the Time Quantum: ");
    scanf("%d", &time_quantum);

    for (int i = 0; i < n; i++) {
        proc[i].pid = i + 1;
        printf("\nProcess P%d:\n", proc[i].pid);
        printf("  Enter Arrival Time: ");
        scanf("%d", &proc[i].at);
        printf("  Enter Burst Time: ");
        scanf("%d", &proc[i].bt);
        proc[i].rt = proc[i].bt;
    }

    int currentTime = 0;
    int completed = 0;
    
    for(int i = 0; i < n; i++) {
        if(proc[i].at == 0) {
            enqueue(i);
        }
    }

    while (completed != n) {
        int current_pid_index = dequeue();

        if (current_pid_index == -1) {
            currentTime++;
            for (int i = 0; i < n; i++) {
                if (proc[i].rt > 0 && proc[i].at == currentTime && !isInQueue(i, n)) {
                    enqueue(i);
                }
            }
            continue;
        }

        int time_slice = 0;
        if (proc[current_pid_index].rt > time_quantum) {
            time_slice = time_quantum;
            proc[current_pid_index].rt -= time_quantum;
        } else {
            time_slice = proc[current_pid_index].rt;
            proc[current_pid_index].rt = 0;
        }

        for(int t = 0; t < time_slice; t++) {
            currentTime++;
            for (int i = 0; i < n; i++) {
                if (i != current_pid_index && proc[i].rt > 0 && proc[i].at == currentTime && !isInQueue(i, n)) {
                    enqueue(i);
                }
            }
        }
        
        if (proc[current_pid_index].rt == 0) {
            completed++;
            proc[current_pid_index].ct = currentTime;
            proc[current_pid_index].tat = proc[current_pid_index].ct - proc[current_pid_index].at;
            proc[current_pid_index].wt = proc[current_pid_index].tat - proc[current_pid_index].bt;
            total_wt += proc[current_pid_index].wt;
            total_tat += proc[current_pid_index].tat;
        } else {
            enqueue(current_pid_index);
        }
    }

    printf("\nPID\tArrival\tBurst\tCompletion\tTurnaround\tWaiting\n");
    for (int i = 0; i < n; i++) {
        printf("P%d\t%d\t%d\t%d\t\t%d\t\t%d\n",
               proc[i].pid, proc[i].at, proc[i].bt,
               proc[i].ct, proc[i].tat, proc[i].wt);
    }

    printf("\nAverage Waiting Time:    %.2f\n", total_wt / n);
    printf("Average Turnaround Time: %.2f\n", total_tat / n);

    return 0;
}
```

-----

### 3\. Memory Allocation Algorithms (First, Best, Worst Fit)

#### 💡 Explanation

These are algorithms for allocating memory to processes in a fixed-partition system.

  * **First Fit:** Allocates the *first* available memory block that is large enough to hold the process.
  * **Best Fit:** Allocates the *smallest* available memory block that is large enough. This minimizes the size of the leftover fragment (internal fragmentation).
  * **Worst Fit:** Allocates the *largest* available memory block. This leaves the largest possible fragment, which might be useful for other processes later.

This program demonstrates all three by attempting to fit a list of processes into a list of memory blocks.

#### 💻 C Code

```c
#include <stdio.h>
#include <string.h>

void firstFit(int blockSize[], int m, int processSize[], int n) {
    int allocation[n];
    memset(allocation, -1, sizeof(allocation));
    int blockCopy[m];
    for (int i = 0; i < m; i++)
        blockCopy[i] = blockSize[i];

    for (int i = 0; i < n; i++) {
        for (int j = 0; j < m; j++) {
            if (blockCopy[j] >= processSize[i]) {
                allocation[i] = j;
                blockCopy[j] -= processSize[i];
                break;
            }
        }
    }

    printf("\n--- First Fit Allocation ---\n");
    printf("Process No.\tProcess Size\tBlock No.\n");
    for (int i = 0; i < n; i++) {
        printf(" %d\t\t%d\t\t", i + 1, processSize[i]);
        if (allocation[i] != -1)
            printf("%d\n", allocation[i] + 1);
        else
            printf("Not Allocated\n");
    }
}

void bestFit(int blockSize[], int m, int processSize[], int n) {
    int allocation[n];
    memset(allocation, -1, sizeof(allocation));
    int blockCopy[m];
    for (int i = 0; i < m; i++)
        blockCopy[i] = blockSize[i];

    for (int i = 0; i < n; i++) {
        int bestIdx = -1;
        for (int j = 0; j < m; j++) {
            if (blockCopy[j] >= processSize[i]) {
                if (bestIdx == -1)
                    bestIdx = j;
                else if (blockCopy[bestIdx] > blockCopy[j])
                    bestIdx = j;
            }
        }

        if (bestIdx != -1) {
            allocation[i] = bestIdx;
            blockCopy[bestIdx] -= processSize[i];
        }
    }

    printf("\n--- Best Fit Allocation ---\n");
    printf("Process No.\tProcess Size\tBlock No.\n");
    for (int i = 0; i < n; i++) {
        printf(" %d\t\t%d\t\t", i + 1, processSize[i]);
        if (allocation[i] != -1)
            printf("%d\n", allocation[i] + 1);
        else
            printf("Not Allocated\n");
    }
}

void worstFit(int blockSize[], int m, int processSize[], int n) {
    int allocation[n];
    memset(allocation, -1, sizeof(allocation));
    int blockCopy[m];
    for (int i = 0; i < m; i++)
        blockCopy[i] = blockSize[i];

    for (int i = 0; i < n; i++) {
        int worstIdx = -1;
        for (int j = 0; j < m; j++) {
            if (blockCopy[j] >= processSize[i]) {
                if (worstIdx == -1)
                    worstIdx = j;
                else if (blockCopy[worstIdx] < blockCopy[j])
                    worstIdx = j;
            }
        }

        if (worstIdx != -1) {
            allocation[i] = worstIdx;
            blockCopy[worstIdx] -= processSize[i];
        }
    }

    printf("\n--- Worst Fit Allocation ---\n");
    printf("Process No.\tProcess Size\tBlock No.\n");
    for (int i = 0; i < n; i++) {
        printf(" %d\t\t%d\t\t", i + 1, processSize[i]);
        if (allocation[i] != -1)
            printf("%d\n", allocation[i] + 1);
        else
            printf("Not Allocated\n");
    }
}

int main() {
    int m, n;

    printf("Enter number of memory blocks: ");
    scanf("%d", &m);
    int blockSize[m];
    for (int i = 0; i < m; i++) {
        printf("Enter size of block %d: ", i + 1);
        scanf("%d", &blockSize[i]);
    }

    printf("\nEnter number of processes: ");
    scanf("%d", &n);
    int processSize[n];
    for (int i = 0; i < n; i++) {
        printf("Enter size of process %d: ", i + 1);
        scanf("%d", &processSize[i]);
    }

    firstFit(blockSize, m, processSize, n);
    bestFit(blockSize, m, processSize, n);
    worstFit(blockSize, m, processSize, n);

    return 0;
}
```

-----

### 4\. Reader/Writer Problems using Semaphore

#### 💡 Explanation

This is a classic synchronization problem. We have two types of processes: Readers and Writers.

1.  Any number of Readers can read the shared resource at the same time.
2.  Only one Writer can write to the shared resource at any given time.
3.  If a Writer is writing, no Reader can read.

This code uses POSIX threads (`pthread`) and semaphores (`semaphore.h`).

  * `mutex`: A binary semaphore (or mutex) to protect the `read_count` variable.
  * `wrt`: A binary semaphore to control access for writers. It is also used by the first reader entering and the last reader exiting to block writers.
  * `read_count`: An integer that tracks how many readers are currently active.

#### 💻 C Code

```c
#include <stdio.h>
#include <pthread.h>
#include <semaphore.h>
#include <unistd.h>

sem_t mutex;
sem_t wrt;
int read_count = 0;
int shared_data = 1; 

void *writer(void *arg) {
    int id = *((int *)arg);
    
    sem_wait(&wrt); 
    
    printf("Writer %d is writing... ", id);
    shared_data = shared_data * 2;
    printf("New data is %d\n", shared_data);
    sleep(1);
    
    sem_post(&wrt); 
    
    return NULL;
}

void *reader(void *arg) {
    int id = *((int *)arg);
    
    sem_wait(&mutex);
    read_count++;
    if (read_count == 1) {
        sem_wait(&wrt); 
    }
    sem_post(&mutex);

    
    printf("Reader %d is reading... Data is %d\n", id, shared_data);
    sleep(1);

    
    sem_wait(&mutex);
    read_count--;
    if (read_count == 0) {
        sem_post(&wrt); 
    }
    sem_post(&mutex);
    
    return NULL;
}

int main() {
    int num_readers, num_writers;
    
    printf("Enter number of readers: ");
    scanf("%d", &num_readers);
    printf("Enter number of writers: ");
    scanf("%d", &num_writers);

    pthread_t readers[num_readers], writers[num_writers];
    int reader_ids[num_readers], writer_ids[num_writers];

    sem_init(&mutex, 0, 1);
    sem_init(&wrt, 0, 1);

    for (int i = 0; i < num_writers; i++) {
        writer_ids[i] = i + 1;
        pthread_create(&writers[i], NULL, writer, &writer_ids[i]);
    }

    for (int i = 0; i < num_readers; i++) {
        reader_ids[i] = i + 1;
        pthread_create(&readers[i], NULL, reader, &reader_ids[i]);
    }

    for (int i = 0; i < num_writers; i++) {
        pthread_join(writers[i], NULL);
    }

    for (int i = 0; i < num_readers; i++) {
        pthread_join(readers[i], NULL);
    }

    sem_destroy(&mutex);
    sem_destroy(&wrt);

    return 0;
}
```

-----

### 5\. Banker's Algorithm for Deadlock Avoidance

#### 💡 Explanation

The Banker's Algorithm is a deadlock avoidance algorithm. It checks if a system is in a "safe state" before granting any resource request. A safe state is one where there exists a sequence of process execution (a "safe sequence") such that all processes can run to completion.

This program implements the "Safety Algorithm" part. It checks if a safe sequence exists given the current allocation, maximum needs, and available resources.

  * `Available`: Vector of available resources.
  * `Max`: Matrix of maximum resources each process can request.
  * `Allocation`: Matrix of resources currently allocated to each process.
  * `Need`: Matrix of remaining resources needed (`Need = Max - Allocation`).

#### 💻 C Code

```c
#include <stdio.h>
#include <stdbool.h>

int main() {
    int n, m;

    printf("Enter the number of processes: ");
    scanf("%d", &n);
    printf("Enter the number of resource types: ");
    scanf("%d", &m);

    int max[n][m];
    int allocation[n][m];
    int need[n][m];
    int available[m];

    printf("\nEnter Max matrix:\n");
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < m; j++) {
            scanf("%d", &max[i][j]);
        }
    }

    printf("\nEnter Allocation matrix:\n");
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < m; j++) {
            scanf("%d", &allocation[i][j]);
            need[i][j] = max[i][j] - allocation[i][j];
        }
    }

    printf("\nEnter Available resources:\n");
    for (int j = 0; j < m; j++) {
        scanf("%d", &available[j]);
    }

    bool finish[n];
    for (int i = 0; i < n; i++) {
        finish[i] = false;
    }

    int safeSequence[n];
    int count = 0;

    while (count < n) {
        bool found = false;
        for (int i = 0; i < n; i++) {
            if (!finish[i]) {
                bool canExecute = true;
                for (int j = 0; j < m; j++) {
                    if (need[i][j] > available[j]) {
                        canExecute = false;
                        break;
                    }
                }

                if (canExecute) {
                    for (int j = 0; j < m; j++) {
                        available[j] += allocation[i][j];
                    }
                    safeSequence[count++] = i;
                    finish[i] = true;
                    found = true;
                }
            }
        }

        if (!found) {
            printf("\nSystem is not in a safe state (Deadlock potential).\n");
            return 0;
        }
    }

    printf("\nSystem is in a safe state.\nSafe Sequence: ");
    for (int i = 0; i < n; i++) {
        printf("P%d", safeSequence[i]);
        if (i != n - 1) {
            printf(" -> ");
        }
    }
    printf("\n");

    return 0;
}
```

-----

### 6\. Study of Basic UNIX Commands and Editors

#### 💡 Explanation

This item is a practical lab, not a single program. The goal is to become familiar with the UNIX/Linux command-line environment and its text editors.

  * **Basic Commands:** You are expected to learn and practice commands like:
      * `ls`: List directory contents.
      * `cd`: Change directory.
      * `pwd`: Print working directory.
      * `mkdir`: Create a new directory.
      * `rmdir`: Remove an empty directory.
      * `cp`: Copy files.
      * `mv`: Move or rename files.
      * `rm`: Remove files.
      * `cat`: Concatenate and display file contents.
      * `grep`: Search for text within files.
      * `man`: Read the manual for any command.
  * **Editors:**
      * **`vi` / `vim`:** A powerful, modal text editor. You have to switch between "Insert" mode (for typing) and "Command" mode (for saving, quitting, navigating).
      * **`ed` / `ex`:** Line-based editors, which are more primitive and less common now.
      * **`emacs`:** Another very powerful, extensible editor (almost an operating system itself).

There is no single "code" for this; the task is to practice using these tools in a terminal.

-----

### 7\. Process Management System Calls

#### 💡 Explanation

These are small C programs demonstrating fundamental UNIX system calls for process management. You must compile and run them on a UNIX-like system (like Linux or macOS).

-----

#### 7-A. `fork()` Function

The `fork()` call creates a new process (the "child") that is an exact copy of the calling process (the "parent"). `fork()` returns `0` to the child process and the child's Process ID (PID) to the parent process.

```c
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>

int main() {
    pid_t pid;

    pid = fork();

    if (pid < 0) {
        fprintf(stderr, "Fork Failed\n");
        return 1;
    } else if (pid == 0) {
        printf("Hello from the Child Process! (PID: %d)\n", getpid());
    } else {
        printf("Hello from the Parent Process! (PID: %d, Child PID: %d)\n", getpid(), pid);
    }

    return 0;
}
```

-----

#### 7-B. `execv()` Function

The `execv()` function (part of the `exec` family) *replaces* the current process with a new one. It takes the path to an executable and an array of arguments (`argv`). The original program's code stops running, and the new program (`/bin/ls` in this case) takes over.

```c
#include <stdio.h>
#include <unistd.h>

int main() {
    char *binaryPath = "/bin/ls";
    char *args[] = {"ls", "-l", NULL};

    printf("This line is from the original program.\n");
    
    execv(binaryPath, args);
    
    printf("This line will NEVER print if execv succeeds.\n");

    return 0;
}
```

-----

#### 7-C. `execlp()` Function

Similar to `execv()`, `execlp()` also replaces the current process. The 'l' means it takes arguments as a list (variable number of arguments), and the 'p' means it will search the system's `PATH` for the executable, so you don't need to provide the full `/bin/ls` path.

```c
#include <stdio.h>
#include <unistd.h>

int main() {
    printf("This line is from the original program.\n");
    
    execlp("ls", "ls", "-l", NULL);
    
    printf("This line will NEVER print if execlp succeeds.\n");

    return 0;
}
```

-----

#### 7-D. `wait()` Function

The `wait()` system call is used by a parent process to pause its own execution until one of its child processes terminates. This is crucial for "reaping" child processes and preventing "zombies" (terminated processes that still occupy a slot in the process table).

```c
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>

int main() {
    pid_t pid;
    
    pid = fork();

    if (pid < 0) {
        fprintf(stderr, "Fork Failed\n");
        return 1;
    } else if (pid == 0) {
        printf("CHILD: I am running...\n");
        sleep(2);
        printf("CHILD: I am done.\n");
    } else {
        printf("PARENT: I am waiting for my child to finish...\n");
        wait(NULL);
        printf("PARENT: My child has finished. I can now exit.\n");
    }

    return 0;
}
```

-----

#### 7-E. `sleep()` Function

The `sleep()` function is simple: it causes the calling process to be suspended for a specified number of seconds.

```c
#include <stdio.h>
#include <unistd.h>

int main() {
    printf("Hello.\n");
    sleep(3);
    printf("...Goodbye after 3 seconds.\n");
    return 0;
}
```

-----

### 8\. Simple Shell Program (Conditionals and Loops)

#### 💡 Explanation

This is a **Shell Script** (not a C program) that demonstrates the use of basic shell programming constructs: variables, conditional logic (`if`/`else`), and looping (`for`, `while`).

#### 💻 Shell Script (e.g., `demo.sh`)

```bash
#!/bin/bash

echo "--- Variable Demonstration ---"
NAME="User"
echo "Hello, $NAME"

echo "\n--- Conditional (if/else) ---"
read -p "Enter a number: " NUM
if [ "$NUM" -gt 10 ]; then
  echo "$NUM is greater than 10."
else
  echo "$NUM is not greater than 10."
fi

echo "\n--- For Loop ---"
echo "Counting to 3:"
for i in 1 2 3
do
  echo "Number $i"
done

echo "\n--- While Loop ---"
COUNT=0
while [ "$COUNT" -lt 3 ]; do
  echo "Count is $COUNT"
  COUNT=$((COUNT + 1))
done

echo "\nScript finished."
```

-----

### 9\. Shell Program to Swap Two Integers

#### 💡 Explanation

This is another simple **Shell Script**. It takes two numbers, stores them in variables, and uses a third variable (`TEMP`) to swap their values before printing the result.

#### 💻 Shell Script (e.g., `swap.sh`)

```bash
#!/bin/bash

A=10
B=20

echo "Before swap: A = $A, B = $B"

TEMP=$A
A=$B
B=$TEMP

echo "After swap:  A = $A, B = $B"
```
