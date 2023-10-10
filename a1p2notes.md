# Problem Statement
Currently timer_sleep uses busy waiting. This is inefficient as "slept" threads go back
into the ready queue and then .yield() until they are ready to go, as the red segments in 
the below image depict:
[!Busy Waiting in red](./timer_sleep_as_its_currently_implemented.png)

# Proposed Solution
As it is, currently there are two lists in PintOS, declared in thread.c:
```c
/* List of processes in THREAD_READY state, that is, processes
   that are ready to run but not actually running. */
static struct list ready_list;

/* List of all processes.  Processes are added to this list
   when they are first scheduled and removed when they exit. */
static struct list all_list;
```
The list struct's definition:
```c
/* List. */
struct list 
  {
    struct list_elem head;      /* List head. */
    struct list_elem tail;      /* List tail. */
  };
```

We propose a third list, the sleep_list, keeping track of all sleeping threads. We
then go through the list to wake up threads everytime the Timer Inerrupt happens. 
In order to know which threads to wake up and which to not, we need to keep track 
of how many ticks have occurred since OS boot. As such, we have to add a global tick
variable that's incremented in a similar fashion to idle_ticks in thread.c. 
Moreover, we must modify our thread struct to keep track of which tick it needs to
wake up on.

# Acceptance Criteria
By running the following command, we can determine the effectiveness of our solution:
$ pintos -- -q run alarm-multiple

In particular, ont he line that says:
Thread: X idle ticks, Y kernel ticks, Z user ticks

In busy waiting, idle ticks will be zero as the ready queue is never emptied.
However, in our solution, therefore, x != 0 to succeed.