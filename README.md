# Linux-IPC-Semaphores
Ex05-Linux IPC-Semaphores

# AIM:
To Write a C program that implements a producer-consumer system with two processes using Semaphores.

# DESIGN STEPS:

### Step 1:

Navigate to any Linux environment installed on the system or installed inside a virtual environment like virtual box/vmware or online linux JSLinux (https://bellard.org/jslinux/vm.html?url=alpine-x86.cfg&mem=192) or docker.

### Step 2:

Write the C Program using Linux Process API - Sempahores

### Step 3:

Execute the C Program for the desired output. 

# PROGRAM:

## Write a C program that implements a producer-consumer system with two processes using Semaphores.

```

#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/sem.h>
#include <sys/wait.h>

#define NUM_LOOPS 10

union semun {
    int val;
};

void wait_semaphore(int sem_id) {
    struct sembuf op;
    op.sem_num = 0;
    op.sem_op = -1;
    op.sem_flg = 0;
    semop(sem_id, &op, 1);
}

void signal_semaphore(int sem_id) {
    struct sembuf op;
    op.sem_num = 0;
    op.sem_op = 1;
    op.sem_flg = 0;
    semop(sem_id, &op, 1);
}

int main(void) {
    int sem_id;
    union semun sem_val;
    int child_pid;

    sem_id = semget(IPC_PRIVATE, 1, 0600);
    if (sem_id == -1) {
        perror("semget");
        exit(1);
    }

    printf("Semaphore set created, semaphore set id %d.\n",
           sem_id);

    sem_val.val = 0;
    if (semctl(sem_id, 0, SETVAL, sem_val) == -1) {
        perror("semctl");
        semctl(sem_id, 0, IPC_RMID);
        exit(1);
    }

    child_pid = fork();

    if (child_pid < 0) {
        perror("fork");
        semctl(sem_id, 0, IPC_RMID);
        exit(1);
    }

    if (child_pid == 0) {
        /* Consumer process */
        for (int i = 0; i < NUM_LOOPS; i++) {
            wait_semaphore(sem_id);
            printf("consumer: %d\n", i);
            fflush(stdout);
        }
        exit(0);
    } else {
        /* Producer process */
        for (int i = 0; i < NUM_LOOPS; i++) {
            printf("producer: %d\n", i);
            fflush(stdout);
            signal_semaphore(sem_id);
            usleep(100000);
        }

        wait(NULL);
        semctl(sem_id, 0, IPC_RMID);
        printf("Semaphore removed.\n");
    }

    return 0;
}
```


## OUTPUT
$ ./sem.o 

<img width="671" height="587" alt="image" src="https://github.com/user-attachments/assets/2e596e71-99f1-4fa4-bf46-f68c3fc50855" />

$ ipcs





# RESULT:
The program is executed successfully.
