# Assignment 6 — Election Algorithms (Bully & Ring)
### Files: `Bully.java` & `Ring.java`

---

## 1. Code Explanation

---

### `Bully.java` — Bully Election Algorithm

#### Constructor

```java
public Bully(int max) {
    max_processes = max;
    processes = new boolean[max_processes];
    coordinator = max;
```
Creates `max` processes, all initially `true` (up/alive). The coordinator is set to `max` — the highest-numbered process — as Bully always starts with the largest ID as leader.

---

#### `downProcess` / `upProcess`

```java
void downProcess(int process_id) {
    processes[process_id - 1] = false;
}
```
Marks a process as down/crashed in the boolean array. `process_id - 1` because processes are 1-indexed but the array is 0-indexed.

---

#### `runElection` — The Core Bully Logic

```java
void runElection(int process_id) {
    coordinator = process_id;
    boolean keepGoing = true;
    for (int i = process_id; i < max_processes && keepGoing; i++) {
        System.out.println("Election message sent from process " + process_id + " to process " + (i+1));
        if (processes[i]) {
            keepGoing = false;
            runElection(i + 1);
        }
    }
}
```
This is a **recursive** implementation of the Bully algorithm:
- The initiating process sets itself as coordinator (`coordinator = process_id`).
- It sends election messages to all **higher-numbered** processes (loop from `process_id` to `max_processes`).
- If a higher-numbered process is **alive** (`processes[i] == true`), that process takes over and calls `runElection` recursively with its own ID.
- The recursion stops when no higher alive process exists — that process becomes the final coordinator.
- `keepGoing = false` stops the current level's loop once a responder is found.

---

### `Ring.java` — Ring Election Algorithm

#### Constructor

```java
public Ring(int max) {
    coordinator = max;
    pid = new ArrayList<Integer>();
    processes = new boolean[max];
}
```
Similar setup to Bully. Additionally uses an `ArrayList<Integer> pid` to collect process IDs as the election message travels around the ring.

---

#### `initElection` — The Core Ring Logic

```java
void initElection(int process_id) {
    if (processes[process_id-1]) {
        pid.add(process_id);
        int temp = process_id;

        System.out.print("Process P" + process_id + " sending the following list:- ");
        displayArrayList(pid);

        while (temp != process_id - 1) {
            if (processes[temp]) {
                pid.add(temp + 1);
                System.out.print("Process P" + (temp+1) + " sending the following list:- ");
                displayArrayList(pid);
            }
            temp = (temp + 1) % max_processes;
        }
        coordinator = Collections.max(pid);
        System.out.println("Process P" + process_id + " has declared P" + coordinator + " as the coordinator");
        pid.clear();
    }
}
```
- The initiating process adds its own ID to `pid` and sends it to the next node.
- The message travels **clockwise** around the ring (`temp = (temp+1) % max_processes`).
- Each **alive** node appends its ID to the list and forwards it.
- The loop stops when we've gone all the way around (`temp == process_id - 1`), i.e., back to the node before the initiator.
- The initiator then picks `Collections.max(pid)` — the highest ID in the list — as the new coordinator.
- `pid.clear()` resets for the next election.

---

#### `displayArrayList`

```java
void displayArrayList(ArrayList<Integer> pid) {
    System.out.print("[ ");
    for (Integer x : pid) System.out.print(x + " ");
    System.out.print(" ]\n");
}
```
A helper that prints the current election message list in a readable `[ 2 3 1 ]` format.

---

## 2. Concept — Election Algorithms in Distributed Systems

### Overview
In a distributed system, a **coordinator** (also called leader or master) manages shared resources or coordinates tasks. When the coordinator **crashes or becomes unreachable**, the remaining processes must **elect a new coordinator** automatically. This is solved by **Election Algorithms**.

---

### Bully Algorithm

#### How It Works
1. Any process that detects the coordinator has failed **initiates an election**.
2. It sends an **ELECTION message** to all processes with a **higher ID**.
3. If no higher process responds → it declares itself coordinator and sends a **COORDINATOR message** to all.
4. If a higher process receives the ELECTION message, it **bullies** the lower process out (sends OK back, starts its own election).
5. The highest-numbered **alive** process always wins — hence the name "Bully."

#### Key Properties
| Property | Detail |
|---|---|
| Winner | Highest-numbered alive process |
| Message complexity | O(n²) in worst case |
| Time complexity | O(1) rounds in best case |
| Assumption | Each process knows all others' IDs |
| Failure detection | Timeout-based |

---

### Ring Election Algorithm

#### How It Works
1. The detecting process starts an **election message** containing its own ID.
2. The message is passed **clockwise** around the ring, with each alive node **appending its ID**.
3. When the message completes a full circle back to the initiator, the initiator selects the **maximum ID** from the list.
4. That maximum ID process is declared the **new coordinator**.

#### Key Properties
| Property | Detail |
|---|---|
| Winner | Highest ID in the collected list |
| Message complexity | O(n) — one full ring traversal |
| Topology required | Logical ring |
| Assumption | Processes know their neighbors |
| Advantage | Simple and low message overhead |

---

### Bully vs Ring Algorithm Comparison

| Feature | Bully | Ring |
|---|---|---|
| Topology | Any (fully connected) | Ring |
| Winner | Highest alive ID | Highest ID in election list |
| Messages sent | O(n²) worst case | O(n) |
| Speed | Fast (few rounds) | One full traversal |
| Failure handling | Immediate response | Skips down nodes |
| Complexity | Higher | Lower |

---

### Election Algorithms in Distributed Systems Context
- **Leader election** is a fundamental problem in distributed computing
- Used in: **Apache Zookeeper** (ZAB protocol), **etcd** (Raft), **Hadoop HDFS** (NameNode election), **Kafka** (controller election)
- Related to the **consensus problem** — all processes must agree on one leader
- Must handle **network partitions**, **message delays**, and **simultaneous elections**
- Modern systems use **Raft** and **Paxos** which extend election concepts with log replication for fault-tolerant consensus

---

## 3. Terminologies & QnAs

**Q: What is a coordinator in distributed systems?**
A: A special process responsible for managing shared resources, task scheduling, or coordination. If it fails, a new one must be elected.

**Q: Why is the highest-ID process elected in both algorithms?**
A: Using the highest ID is a simple, unambiguous tiebreaker. It ensures all processes independently agree on the same winner without extra negotiation.

**Q: What is the "bully" in the Bully algorithm?**
A: When a higher-priority process receives an election message from a lower one, it "bullies" the lower one by taking over the election. The strongest (highest ID) always wins.

**Q: What triggers an election?**
A: A timeout — when a process sends a message to the coordinator and receives no response within an expected time, it assumes the coordinator has failed and initiates an election.

**Q: What is the message complexity of the Bully algorithm?**
A: O(n²) in the worst case (process 1 starts, all higher processes respond and start their own elections). Best case is O(1) if the second-highest starts the election.

**Q: What is the message complexity of the Ring algorithm?**
A: O(n) — the election message makes exactly one full trip around the ring of n nodes.

**Q: What happens if two processes start an election simultaneously in Ring?**
A: Both messages travel the ring. The initiator that started with a lower ID will see the higher-ID message and discard its own. The higher-ID message wins.

**Q: What does `Collections.max(pid)` do?**
A: Returns the largest integer in the `ArrayList`, which becomes the new coordinator — the process with the highest ID among all alive nodes.

**Q: What is `processes[process_id - 1]`? Why -1?**
A: Processes are numbered starting from 1 (P1, P2...) but Java arrays are 0-indexed. So P1 maps to `processes[0]`, P2 to `processes[1]`, etc.

**Q: What is the difference between election and consensus?**
A: Election selects a single leader. Consensus is a broader problem where all processes agree on a value. Leader election is one form of consensus.

**Q: What is a split-brain scenario?**
A: When a network partition causes two groups of nodes to each elect their own coordinator, leading to two leaders — a dangerous inconsistency. Real systems use quorums to prevent this.

**Q: Why does `runElection` use recursion in Bully?**
A: When a higher process responds and takes over, it starts its own election from its position. Recursion naturally models this "passing the torch" behavior up the process hierarchy.

---

## 4. Execution Steps

### Prerequisites
- Java JDK installed

---

### Bully Algorithm

#### Step 1 — Compile
```bash
javac Bully.java
```

#### Step 2 — Run
```bash
java Bully
```

#### Step 3 — Sample Interaction
```
Bully Algorithm
1. Create processes  2. Display processes  3. Up a process
4. Down a process    5. Run election       6. Exit Program
Enter your choice:- 1
Enter the number of processes:- 4
Creating processes..
P1 created  P2 created  P3 created  P4 created
Process P4 is the coordinator

Enter your choice:- 4
Enter the process number to down:- 4
Process 4 is down.            ← Coordinator crashes!

Enter your choice:- 5
Enter the process number which will perform election:- 2
Election message sent from process 2 to process 3
Election message sent from process 3 to process 4
P1 is up  P2 is up  P3 is up  P4 is down
Process P3 is the coordinator  ← New coordinator elected!
```

---

### Ring Algorithm

#### Step 1 — Compile
```bash
javac Ring.java
```

#### Step 2 — Run
```bash
java Ring
```

#### Step 3 — Sample Interaction
```
Ring Algorithm
1. Create processes  2. Display processes  3. Up a process
4. Down a process    5. Run election       6. Exit Program
Enter your choice:- 1
Enter the total number of processes:- 4
P1 created.  P2 created.  P3 created.  P4 created.
P4 is the coordinator

Enter your choice:- 4
Enter the process to down:- 4
Process P4 is down.           ← Coordinator crashes!

Enter your choice:- 5
Enter the process which will initiate election:- 2
Process P2 sending the following list:- [ 2 ]
Process P3 sending the following list:- [ 2 3 ]
Process P1 sending the following list:- [ 2 3 1 ]  ← P4 skipped (down)
Process P2 has declared P3 as the coordinator      ← max(2,3,1) = 3
```

---

### Menu Reference

| Choice | Action |
|---|---|
| 1 | Create N processes (must do this first) |
| 2 | Display all process statuses + coordinator |
| 3 | Bring a process back up |
| 4 | Crash/bring down a process |
| 5 | Run election from a chosen process |
| 6 | Exit the program |

> **Tip:** Always run choice `1` first to initialize processes before using any other option, otherwise you'll get a `NullPointerException`.
> To simulate a real scenario: create processes → bring down the coordinator → run election from any lower process.
