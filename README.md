# Readers-writers-problem

The Reader-Writer Problem is a classic synchronisation and concurrency problem. It is a circumstance in which multiple threads that are running concurrently can receive and modify a data structure at the same time. At any given point in time, only one of the Writers is authorised to enter the critical section. In states where there is no active Writer, an unlimited number of Readers are able to access the critical section.

The processes here are of two types:

- **Readers:** These are the processes which can only read the data from the shared memory. They do not perform any updates.
- **Writers:**  These are the processes which can write/update the data to the shared memory.

It is permissible for two or more concurrent readers to access shared data, as long as no changes are made and the file format remains unchanged after each reading.

However, if one writer (let's say p1) is editing or writing the file, it should be locked and no other writer (let's say p2) should be able to make changes until p1 has completed writing the file.

There are several variations of the problem, which includes:

- **First readers-writers problem:** It stipulates that no reader may be held in limbo unless the writer has obtained permission to use the shared object. In other words, no reader should hold up other readers because a writer is waiting**.**
- **Second readers-writers problem:** As soon as a writer is prepared, it must begin writing as soon as feasible. In other words, if a writer is awaiting access to an object, no additional readers may begin reading.

A solution to either problem may result in starvation. In the first case, writers may starve; in the second case, readers may starve.

Firstly, we discuss the **classical solution** to the problem followed by a **popular starvation free solution** to this problem. We then discuss an **optimised starve free solution** to the problem.





## Classical solution to The Readers–Writers Problem

**Global variables declaration and initialisation:**
``` cpp
semaphore mutex=1;
semaphore db=1;
int read_count=0;
```

**Reader process:**
``` cpp

void reader(void)
{
	do{
		wait(mutex);
        read_count++;
        if(read_count==1) wait(db);
        signal(mutex);

        /**
        *
        * Critical solution
        *
        */

        wait(mutex);
        read_count--;
        if(read_count==0) signal(db);
        signal(mutex);
    }while(true);
}
```


**Writer process:**
``` cpp
void writer(void)
{
    do{
        wait(db);

        /**
        *
        * Critical solution
        *
        */

        signal(db);
    }while(true);
}
```

**Explanation of classical solution:**

In the classical solution we use two semaphores **mutex and the db** and one variable **read\_count**. The **read\_count** variable indicates the number of readers who are presently accessing the critical section. 

The **mutex** semaphore controls write access to the read\_count variable. Any reader who enters or leaves the critical section is required to obtain this mutex. 

Both readers and writers utilise the **db** semaphore to obtain access to the critical section. Every writer must acquire it before entering the critical section and signify its exit from the section, but only the first reader must acquire it to enter the critical section. The final reader must signal the db semaphore to indicate that it is now free and available to writers.

The only drawback is that it starves the Writer: a Writer thread cannot execute while any number of Readers are perpetually entering and exiting the working area.

## Popular Starve free solution:

**Global variables declaration and initialisation:**
```cpp
semaphore mutex=1;
semaphore db=1;
semaphore in_mutex=1;
int read_count=0;
```

**Reader process:**
```cpp
void reader(void)
{
    do{
        wait(in_mutex);
        wait(mutex);
        read_count++;
        if(read_count==1) wait(db);
        signal(mutex);
        signal(in_mutex);

        /**
        *
        * Critical solution
        *
        */

        wait(mutex);
        read_count--;
        if(read_count==0) signal(db);
        signal(mutex);
    }while(true);
}
```

**Writer process:**
```cpp
void writer(void)
{
    do{
		wait(in_mutex);
		wait(db);

        /**
        *
        * Critical solution
        *
        */

        signal(db);
        signal(in_mutex);
	}while(true);
}
```

**Explanation of popular starve free solution:**

In this starve free solution, another semaphore has been introduced called the **in\_mutex.** This lock must be acquired by any process before it can enter the critical section. This semaphore ensures that no writer starves to access the shared resources because of continuous arrival of readers. Suppose few readers exist in the critical section and a writer arrives. Since there are readers in the critical section, the value of **db semaphore=0** and the writer will be blocked. However, before blocking, the writer acquires the **in\_mutex** lock. This ensures that any other reader which arrives after the writer will also be blocked behind the writer i.e. the writer is at the head of queue. Once all the readers exit the critical section, db is updated to **db=1,** the writer acquires the **db** lock and enters the critical section. Hence, by using the extra semaphore we have prioritised readers and writers equally and none of them starves.

This solution is straightforward and quick enough. In contrast to the previous solution, the only  penalty in this solution is that the Reader must lock two mutexes in order to access the critical section. If the critical section is quick and mutex locking is a costly system call, it would be advantageous to have an algorithm that permits locking one mutex upon entering the critical section and another upon leaving.





## Optimised Starve free solution:

**Global variables declaration and initialisation**
```cpp
Semaphore wrt=0;
Semaphore in_mutex=1;
Semaphore out_mutex=1;
Int reader_enter=0;
Int reader_exit=0;
Boolean flag=false;
```

**Reader process:**
```cpp
void reader(void)
{
    do{
		wait(in_mutex);
        reader_enter++;
        signal(in_mutex);

        /**
        *
        * Critical solution
        *
        */

        wait(out_mutex);
        reader_exit++;
        if(flag&&reader_enter==reader_exit) signal(wrt);
        signal(out_mutex);
    }while(true);
}
```

**Writer process:**
```cpp
void writer(void)
{
    
    do{
		wait(in_mutex);
		wait(out_mutex);
        if(reader_enter==reader_exit) signal(out_mutex);
        else
        {
            flag=true;
            signal(out_mutex);
            wait(wrt);
            flag=false;
        }

        /**
        *
        * Critical solution
        *
        */

        signal(in_mutex);
    }while(true);
}
```

**Explanation of optimised starve free solution:**

Once a reader process obtains the in mutex semaphore, it enters the critical portion by increasing reader\_enter and signalling the in\_mutex. If another reader comes up, the same occurrence occurs, and it can enter the critical section while the previous reader is still present. If a writer appears, it is first queued in the in\_ mutex. After acquiring it, it obtains the out\_mutex and then determines whether or not the critical section is available. If the values of reader\_enter and reader\_exit are identical, the critical section is empty. If the reader process is still active, the flag is set to true and the writer is blocked on wrt mutex . When prepared to enter the critical section, the flag is reset to false. If a writer arrives directly, reader\_enter would be equal to reader\_exit; consequently, it would acquire semaphores and enter the critical section. Any subsequent process would be enqueued in the in mutex as it finally discharges it upon completion of the critical section.

For a reader process exiting the critical section, it would first increase the reader\_  exit after acquiring the out\_mutex semaphore, and if it is not the last process (checked by comparing reader\_enter and reader\_exit) or there is no writer process waiting in the queue of the wrt mutex, for critical section (checked by flag), it would simply exit the critical section after signalling the out\_mutex. If it is the final reader process in the critical section and a writer process is waiting, it would first signal the wrt semaphore to activate the writer process and allow it to enter the critical section. In the case of a writer process, it merely departs the critical section after releasing the in\_mutex semaphore, thereby making the critical section accessible to other processes in the queue.

In this case, both reader and writer processes are blocked by the in mutex. This ensures that both readers and writers will not go starving. Additionally, the implementation outlined above demonstrates that only a single access to semaphore is required at the start of any reader process attempting to enter the critical section. Thus, it is quicker.

In this case, both reader and writer processes are blocked by the in mutex. This ensures that both readers and writers will not go starving. Additionally, the implementation outlined above demonstrates that only a single access to semaphore is required at the start of any reader process attempting to enter the critical section. Thus, it is quicker.

## Properties of starve free solution:

**Mutual exclusion:**

- In the popular starve free solution, mutual exclusion is achieved by using the semaphores. The **mutex** semaphore ensures that no two readers update the value of read\_count at the same time. The **db** semaphore ensure mutual exclusion between the writers and the first reader. The **in\_mutex** semaphore ensures mutual exclusion among all the processes, it prioritises the readers and writers equally.
- In the optimised starve free solution, mutual exclusion is achieved by using the semaphores. The **wrt** semaphore ensure mutual exclusion between the readers and waiting writer. The **in\_mutex** semaphore ensures mutual exclusion among all the processes. The **out\_mutex** semaphore ensures mutual exclusion among the reader\_exit and flag.

**Progress:** Both the solutions prevent deadlock. Moreover, execution of one process is not dependent on the execution of another process. Hence, progress criteria achieved.

**Bounded waiting:** Before accessing the critical section in both methods, all processes must pass through in\_mutex, which stores all waiting processes in First In First Out order. Consequently, for a finite number of processes, the waiting time for each process in the queue is limited or finite**.**

**References:**

- Operating System Concepts, Ninth Edition, Silberschatz, Galvin, Gagne
- *Modern Operating Systems, Fourth Edition, Andrew S. Tanenbaum, Herbert Bos*
- [*Faster Fair Solution for the Reader-Writer Problem*](https://arxiv.org/ftp/arxiv/papers/1309/1309.4507.pdf)



