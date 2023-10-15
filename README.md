# OS Assignment 2

## Overview

This repository contains the code and documentation for Assignment 2. In this assignment, we are tasked with implementing the **timer_sleep** function as described in the **Pintos** documentation. Additionally, we need to create documentation similar to the format found in Pintos reference manual.

## Table of Contents
[Getting Started](#start)<br>
[Documentation](#documentation)<br>
[Report](#report)<br>
[Team Contributions](#team)<br>
[Debugging Efforts](#debugging)<br>
[Presentation](#presentation)<br>

### Request:

The information at this link: 

> [https://www.scs.stanford.edu/23wi-cs212/reference/index.html](https://www.scs.stanford.edu/23wi-cs212/reference/index.html)


1.	You must implement void timer_sleep (int64_t ticks) as described in

>  [https://www.scs.stanford.edu/23wi-cs212/pintos/pintos_2.html](https://www.scs.stanford.edu/23wi-cs212/pintos/pintos_2.html)

2.	You will have to do a documentation like as in
> [https://www.scs.stanford.edu/23wi-cs212/pintos/pintos_9.html](https://www.scs.stanford.edu/23wi-cs212/pintos/pintos_9.html)

Pintos reference manual is available online at
> [https://web.stanford.edu/class/cs140/projects/pintos/pintos.pdf](https://web.stanford.edu/class/cs140/projects/pintos/pintos.pdf)

## Getting Started <a name="start"></a>


To get started with this project, you'll need to clone this repository:

> ``git clone https://github.com/imnorookie/COMP_7035_OS_Kernel_Hacking.git``


## Documentation <a name="documentation"></a>

We have provided documentation in the docs folder, following the format specified in the assignment instructions. The main documentation file is **sleep_ref.txt**, which contains details about the timer_sleep function.

## Report <a name="report"></a>

Our report for this assignment is available in PDF format. It consists of the following sections:

### Section 0: GitHub Repository
You can access our code on GitHub at the following link: GitHub Repository

### Section 1: Code Changes
In this section, we provide annotated code snippets explaining the changes and additions we made to the codebase to implement the timer_sleep function.

Added to struct thread:
```c
    /* Members for implementiong thread_sleep(). */
    struct list_elem sleeping_elem;     /* List element for sleeping threads list. */
    long long ticks_to_wakeup;          /* tick count this thread should wake up on (if slept). */
```
Added to thread.c:
```c
/* List of all sleeping processes, that is, processes that have
   had timer_sleep called on them and are yet to be woken up. */
static struct list sleeping_list;
```

Modified timer.c:
```c
void
timer_sleep (int64_t ticks) 
{
  enum intr_level old_level;
  int64_t start = timer_ticks ();
  int64_t ticks_to_sleep = start + ticks;
  old_level = intr_disable();  
  if (timer_elapsed(start) < ticks)
    thread_sleep(ticks_to_sleep); // have to move code into thread.c, can't access
                                        // lists, idle_thread, etc. otherwise
  intr_set_level(old_level);
}

/* Timer interrupt handler. */
static void
timer_interrupt (struct intr_frame *args UNUSED)
{
  ticks++;
  thread_tick ();
  wake_up_threads(ticks); // our change here
}
```

Added to thread.c:
```c
void thread_sleep(int64_t ticks_to_sleep) {
  struct thread *t = thread_current();
  ASSERT (intr_get_level() == INTR_OFF);
  if (t == idle_thread) return; // do nothing with idle thread, shud never be slept
  t->ticks_to_wakeup = ticks_to_sleep;
  list_insert_ordered(&sleeping_list, &t->sleeping_elem,sort_sleeping_threads,NULL); 
  thread_block();
}

The below is the sorting function used in list_insert_ordered in thread_sleep.
bool sort_sleeping_threads(const struct list_elem *a, const struct list_elem *b, void *aux UNUSED) {
  int64_t a_ticks, b_ticks;
  a_ticks = list_entry(a, struct thread, sleeping_elem)->ticks_to_wakeup;
  b_ticks = list_entry(b, struct thread, sleeping_elem)->ticks_to_wakeup;
  return (a_ticks < b_ticks);
}

void wake_up_threads(int64_t ticks){
    struct list_elem *e;
    for (e = list_begin (&sleeping_list); e != list_end (&sleeping_list);
     e = list_next (e))
    {
      struct thread *t = list_entry(e, struct thread, sleeping_elem);
      if ((t->ticks_to_wakeup) <= ticks) {
        list_push_back(&ready_list, &t->elem);
        t->status = THREAD_READY;
        list_remove(e);
      } else {
        return; // we are stepping thru the list which is inserted into sorted order.
      }         // logically we can stop iterating as soon as we see a thread that 
    }           // shouldn't be woken up.
}
```

### Section 2: Code Demonstration
We have recorded a video demonstrating our code in action, showcasing the functionality of the timer_sleep function. You can watch the video on YouTube at this URL: YouTube Video Demo

### Section 3: Team Contributions
This section outlines the specific contributions made by each team member to the project.

### Section 4: Debugging Efforts
We document the challenges and debugging efforts encountered during the development process.


## Team Contributions <a name="team"></a>

## Debugging Efforts <a name="debugging"></a>

## Presentation <a name="presentation"></a>

We will present our work during the upcoming lab session, providing a detailed overview of our code, documentation, and the challenges we faced during development.

