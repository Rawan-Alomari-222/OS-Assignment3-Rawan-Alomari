# Assignment 3 - Complete Documentation

**Student Name**: [Rawan Alomari]  
**Student ID**: [445052081]  
**Date Submitted**: [2 may]

---

## 🎥 VIDEO DEMONSTRATION LINK (REQUIRED)

> **⚠️ IMPORTANT: This section is REQUIRED for grading!**
> 
> Upload your 3-5 minute video to your **PERSONAL Gmail Google Drive** (NOT university email).
> Set sharing to "Anyone with the link can view".
> Test the link in incognito/private mode before submitting.

**Video Link**: [Paste your personal Gmail Google Drive link here]

**Video filename**: `[YourStudentID]_Assignment3_Synchronization.mp4`

**Verification**:
- [ ] Link is accessible (tested in incognito mode)
- [ ] Video is 3-5 minutes long
- [ ] Video shows code walkthrough and commits
- [ ] Video has clear audio
- [ ] Uploaded to PERSONAL Gmail (not @std.psau.edu.sa)

---

## Part 1: Development Log (1 mark)

Document your development process with **minimum 3 entries** showing progression:

### Entry 1 - [May 2, 2026, 6:00 PM]
**What I implemented**: 
I began by going over the starter code in `SchedulerSimulationSync.java` and reading the assignment requirements. I determined that `contextSwitchCount`, `completedProcessCount`, `totalWaitingTime`, and `executionLog` are the shared resources that require synchronisation.

**Challenges encountered**: 
Understanding which code segments are crucial and why they might result in race circumstances was the biggest obstacle.

**How I solved it**: 
Every location where several threads update a shared variable has been noted. I related these sections to Operating System Concepts' idea of mutual exclusion.

**Testing approach**: 
I executed the original program and looked at the output, which included the size of the execution log, the number of processes, context switches, and completed processes.

**Time spent**: 
1 hour

---

### Entry 2 - [May 2, 2026, 6:30]
**What I implemented**: 
The necessary synchronisation imports, `ReentrantLock` and `Semaphore`, were introduced. The `CounterLock`, `logLock`, and `cpuSemaphore` were then added to the `SharedResources` class.

**Challenges encountered**: 
For the shared resources, I had to choose between using one lock and several locks.

**How I solved it**: 
I used one lock for the counters and another lock for the execution log. This separates list protection from counter protection while maintaining simplicity in the code.

**Testing approach**: 
After adding the imports and synchronisation objects, I made sure there were no syntax issues by compiling the application.

**Time spent**: 
30 minutes
---

### Entry 3 - [ May 2, 2026, 7:30 PM]
**What I implemented**: 
I used `counterLock` to safeguard the three common counters. I moved the update operations inside the `lock`, `try`, and `finally` blocks to update `incrementContextSwitch()`, `incrementCompletedProcess()`, and `addWaitingTime()`.

**Challenges encountered**: 
Making ensuring that every lock is always released, even in the event of an exception, was the difficult part.

**How I solved it**: 
To ensure that `counterLock.unlock()` always runs, I placed a `finally` block after each crucial section.

**Testing approach**: 
After running the application, I verified that the final data were still displayed accurately.

**Time spent**: 
1 hour
---

### Entry 4 - [May 2, 2026, 8:30 PM]
**What I implemented**: 
I used `logLock` to safeguard the shared `executionLog` ArrayList. In a protected critical section, I put `executionLog.add(message)`.

**Challenges encountered**: 
Because `ArrayList` is not thread-safe, concurrent access may corrupt the log or result in inconsistent entries.

**How I solved it**: 
To stop multiple threads from changing the list simultaneously, I used a different `ReentrantLock` for the execution log.

**Testing approach**: 
I examined the finished product to make sure the execution log summary accurately showed the total number of log entries.

**Time spent**: 
45 minutes
---

### Entry 5 - [May 2, 2026, 9:20 PM]
**What I implemented**: 
The CPU semaphore logic was incorporated into `run()` and `runToCompletion()`. The semaphore is now obtained by the application before to CPU execution and released in a block labelled "finally."

**Challenges encountered**: 
Placing `acquire()` and `release()` in the proper places without altering the original scheduler logic was the major problem.

**How I solved it**: 
To ensure that the semaphore is only released if it was successfully acquired, I utilised a boolean variable named `acquired`.

**Testing approach**: 
I ran the program several times to make sure that every process finished, the number of completed processes matched the total number of processes, and the execution log summary showed up as intended.

**Time spent**: 
30 minutes
---

## Part 2: Technical Questions (1 mark)

### Question 1: Race Conditions
**Q**: Identify and explain TWO race conditions in the original code. For each:
- What shared resource is affected?
- Why is concurrent access a problem?
- What incorrect behavior could occur?

**Your Answer**:

The shared counter variables, particularly `contextSwitchCount`, `completedProcessCount`, and `totalWaitingTime`, contain the initial race condition. Multiple process threads share these variables, and operations like `contextSwitchCount++` and `totalWaitingTime += time` are not atomic operations. Inaccurate final statistics and missed updates can result from a thread reading an outdated value while another thread is updating it.

The shared `executionLog` ArrayList is the second race condition. Since `ArrayList` is not thread-safe, the internal structure of the list may become inconsistent if multiple threads call `executionLog.add(message)` simultaneously. This could result in runtime issues like `ConcurrentModificationException`, missing log entries, or an inaccurate log size.


---

### Question 2: Locks vs Semaphores
**Q**: Explain the difference between ReentrantLock and Semaphore. Where did you use each in your code and why?

**Your Answer**:

A vital section can have mutual exclusion thanks to the use of `ReentrantLock`. In my code, I protected the shared execution log with `logLock` and the shared counter variables with `counterLock`. This guarantees that these shared resources can only be updated by one thread at a time.

Access to a restricted number of permissions is managed by a `Semaphore`. I created a binary semaphore in my code using `cpuSemaphore = new Semaphore(1)`. The simulated CPU execution region can therefore only be accessed by one process thread at a time. I used a semaphore to manage access to the CPU resource and a reentrant lock to safeguard data.


---

### Question 3: Deadlock Prevention
**Q**: What is deadlock? Explain TWO prevention techniques and what you did to prevent deadlocks in your code.

**Your Answer**:

Because each thread is holding a resource and waiting for another resource that never becomes accessible, deadlock occurs when threads wait indefinitely. Releasing locks in a `finally` block is one method of prevention. I called `unlock()` inside of `finally` to apply this strategy in all counter and log important parts.

Keeping the locking order straightforward or avoiding holding numerous locks needlessly is a second preventative strategy. Only one lock—either `counterLock` or `logLock”—is used at a time in each crucial section of my code. In order to prevent releasing a permit that was never acquired, I additionally release the permit for the CPU semaphore inside a `finally` block using the `acquired` boolean.


---

### Question 4: Lock Granularity Design Decision 
**Q**: For Task 1 (protecting the three counters), explain your lock design choice:
- Did you use ONE lock for all three counters (coarse-grained) OR separate locks for each counter (fine-grained)?
- Explain WHY you made this choice
- What are the trade-offs between the two approaches?
- Given that the three counters are independent, which approach provides better concurrency and why?

**Your Answer**:

To safeguard all three counter variables for Task 1, I employed a single ReentrantLock named counterLock, which is an example of a coarse-grained locking technique. The short and straightforward counter operations make this design easier to execute and less error-prone, which is why I chose it.

Coarse-grained locking's primary benefit is its simplicity and ease of synchronisation maintenance. Its drawback is decreased concurrency because, despite the counters' independence, only one thread may update any of them at once.

Because multiple threads could update different counters at the same time, a fine-grained solution would employ distinct locks for each counter, enabling more concurrency. The coarse-grained method is adequate in this case because of the tiny critical sections and simpler verification, even if fine-grained locking offers superior performance.

---

## Part 3: Synchronization Analysis (1 mark)

### Critical Section #1: Counter Variables

**Which variables**: `contextSwitchCount`, `completedProcessCount`, and `totalWaitingTime`.


**Why they need protection**: They are shared between multiple threads and are updated using non-atomic operations such as increment and addition.


**Synchronization mechanism used**: `ReentrantLock` using `counterLock`.


**Code snippet**:
```java
// Paste your implementation here
counterLock.lock();
try {
    contextSwitchCount++;
} finally {
    counterLock.unlock();
}

// Protect completed process counter
counterLock.lock();
try {
    completedProcessCount++;
} finally {
    counterLock.unlock();
}

// Protect total waiting time
counterLock.lock();
try {
    totalWaitingTime += time;
} finally {
    counterLock.unlock();
}
```

**Justification**: 

These operations are critical sections because they modify shared data. The lock ensures mutual exclusion, so only one thread can update the counter values at a time.
---

### Critical Section #2: Execution Log

**What resource**: The shared executionLog ArrayList.

**Why it needs protection**: 
ArrayList is not thread-safe, and multiple threads may add log messages at the same time.
**Synchronization mechanism used**: 
ReentrantLock using logLock.
**Code snippet**:
```java
// Paste your implementation here
// Protect execution log
logLock.lock();
try {
    executionLog.add(message);
} finally {
    logLock.unlock();
}
```

**Justification**: 

The lock prevents concurrent modification of the shared list. This protects the program from inconsistent log entries and possible list-related concurrency errors.
---

### Critical Section #3: CPU Semaphore

**Purpose of semaphore**: 
The semaphore controls access to the simulated CPU execution section.
**Number of permits and why**: 
I used one permit: new Semaphore(1). This creates a binary semaphore, allowing only one process to execute on the CPU at a time.
**Where implemented**: 
The semaphore is implemented in the run() method and in the runToCompletion() method.
**Code snippet**:
```java
// Paste your implementation here
// Track CPU semaphore acquisition
boolean acquired = false;

try {
    // Acquire CPU before execution
    SharedResources.cpuSemaphore.acquire();
    acquired = true;

    // process execution code

} catch (InterruptedException e) {
    System.out.println(Colors.RED + "  ✗ " + name + " could not acquire CPU access." + Colors.RESET);
    Thread.currentThread().interrupt();
} finally {
    // Release CPU in finally block
    if (acquired) {
        SharedResources.cpuSemaphore.release();
    }
}
```

**Effect on program behavior**: 
The semaphore makes CPU access controlled and predictable. It prevents two process threads from executing the simulated CPU section at the same time.
---

## Part 4: Testing and Verification (2 marks)

### Test 1: Consistency Check
**What I tested**: Running the program multiple times to verify that the synchronized version produces correct final results.

**Testing procedure**: 
```bash
# Commands used (run the program at least 5 times)
javac SchedulerSimulationSync.java
java SchedulerSimulationSync
java SchedulerSimulationSync
java SchedulerSimulationSync
java SchedulerSimulationSync
java SchedulerSimulationSync
```

**Results**: 
The program completed successfully in each run. The output showed that all processes completed, and the final statistics were printed correctly. In the provided run, the program generated 16 processes, the total completed processes value was 16, the total context switches value was 35, and the execution log summary showed 70 log entries.

**Why synchronization is necessary**: 
Synchronization is necessary because several process threads access shared resources. Without synchronization, updates to counters could be lost, waiting time could be calculated incorrectly, and the execution log could have missing or inconsistent entries. Even if the error does not appear in every run, the program is still unsafe without synchronization because thread scheduling is unpredictable.
**Conclusion**: 
The synchronized version protects shared resources and produces reliable final statistics.
---

### Test 2: Exception Testing
**What I tested**:Checking whether the execution log causes ConcurrentModificationException or inconsistent behavior.

**Testing procedure**: 
I ran the program multiple times after protecting executionLog.add(message) with logLock.
**Results**: 
The program completed successfully without showing ConcurrentModificationException. The execution log summary appeared at the end of the program.
**What this proves**: 
This proves that the shared ArrayList is protected during updates. Using logLock prevents multiple threads from modifying the log at the same time.
---

### Test 3: Correctness Verification
**What I tested**: I verified that the final values are logically correct after synchronization.

**Expected values**: 
The completed process count should equal the number of processes created. The execution log should contain entries for process start, yield, and completion events. The program should print final statistics after all processes finish.
**Actual values**: 
In the provided output, the number of processes was 16 and the total completed processes was also 16. The total context switches was 35, the total waiting time was 1246738 ms, and the average waiting time was 77921 ms. The execution log summary showed 70 entries.
**Analysis**: 
The completed process count matching the process count confirms that no process was lost. The final statistics confirm that the scheduler loop completed and that the shared counters were updated safely.
---

### Test 4: Different Scenarios
**Scenario tested**: I tested the scheduler behavior with the same student ID seed and repeated runs.

**Purpose**: 
The purpose was to confirm that the synchronization mechanisms do not break the scheduler logic and that the program still handles multiple processes correctly.
**Results**: 
The scheduler continued to add processes to the ready queue, execute time quanta, requeue unfinished processes, and complete all processes. The CPU semaphore did not stop the program because the permit was released correctly.
**What I learned**: 
I learned that synchronization must protect shared resources without changing the main scheduling behavior. Correct placement of acquire(), release(), lock(), and unlock() is important.
---

## Part 5: Reflection and Learning

### What I learned about synchronization:

I learned that multithreaded programs can produce incorrect results even when the code looks simple. Operations such as incrementing a counter are not always atomic. I also learned that shared data must be protected using mutual exclusion. ReentrantLock is useful when protecting critical sections that modify shared variables. Semaphore is useful when controlling access to a limited resource such as the CPU. I learned that try-finally is very important because it guarantees that locks and semaphore permits are released. This assignment also helped me understand how race conditions can affect real program output. Finally, I learned that synchronization should be added carefully without changing the original program logic.

---

### Real-world applications:

Give TWO examples where synchronization is critical:

**Example 1**: 
Banking systems need synchronization when multiple transactions update the same bank account balance at the same time.
**Example 2**: 
Operating systems need synchronization when multiple processes access shared devices such as printers, files, or CPU scheduling queues.
---

### How I would explain synchronization to others:
Synchronization is like allowing only one person to write on a shared whiteboard at a time. If many people write at the same time, the result becomes messy or incorrect. In a program, shared variables are like that whiteboard. A lock allows one thread to enter the critical section while the others wait. A semaphore controls how many threads can access a resource. In this assignment, locks protected shared data, and the semaphore controlled CPU access.

---

## Part 6: GitHub Repository Information

**Repository URL**: https://github.com/Rawan-Alomari-222/OS-Assignment3-Rawan-Alomari.git

**Number of commits**: 
9
**Commit messages**: 
1.Change Student ID to 445052081
2.Add synchronization imports
3.Add locks and CPU semaphore
4.Protect shared counters with ReentrantLock
5.Protect shared counters using ReentrantLock
6.Protect execution log with ReentrantLock
7.Control CPU access using semaphore in run method
8.Control CPU access using semaphore in run method
9.Synchronize runToCompletion using semaphore
---

## Summary

**Total time spent on assignment**: 
5 hours
**Key takeaways**: 
1. Race conditions happen when multiple threads access shared data without synchronization.
2. ReentrantLock provides mutual exclusion for critical sections.
3. Semaphore controls access to limited shared resources such as the CPU.

**Most challenging aspect**: 
The most challenging aspect was deciding the correct places to acquire and release the semaphore without changing the original scheduling logic.
**What I'm most proud of**: 
I am most proud of protecting the shared counters, execution log, and CPU access while keeping the original code structure almost unchanged.
---

**End of Documentation**
