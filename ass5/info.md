# Assignment 5 — Token Ring Algorithm
### File: `Tring.java`

---

## 1. Code Explanation

### Class Structure

```java
class Tring {
    public static void main(String args[]) {
        Scanner sc = new Scanner(System.in);
```
The entire program lives in a single `main` method. A `Scanner` reads user input from the terminal interactively.

---

### Ring Topology Display

```java
int token = 0;
for (int i = 0; i < n; i++)
    System.out.print(" " + i);
System.out.println(" " + 0);
```
Displays the ring as a linear sequence wrapping back to `0`. For `n=4`: `0 1 2 3 0`, simulating a circular ring. The variable `token` tracks which node currently holds the token (starts at node 0).

---

### Main Interaction Loop

```java
while (true) {
    int s = sc.nextInt();   // sender node
    int r = sc.nextInt();   // receiver node
    String d = sc.next();   // data to send
```
Runs indefinitely, accepting new send/receive requests. `s` is the sender, `r` is the intended receiver, and `d` is the data string to transmit.

---

### Token Passing Phase

```java
for (int i = token, j = token; (i % n) != s; i++, j = (j + 1) % n) {
    System.out.print(" " + j + "->");
}
System.out.println(" " + s);
```
The token travels around the ring from its current position (`token`) until it reaches the sender node `s`. The variable `i` counts iterations and checks `i % n != s`. The variable `j` wraps around using modulo so the ring loops correctly. Each intermediate node is printed with `->` to show the token's path.

---

### Data Forwarding Phase

```java
for (int i = (s + 1) % n; i != r; i = (i + 1) % n) {
    System.out.println("Data " + d + " forwarded by " + i);
}
System.out.println("Receiver " + r + " received data: " + d);
```
Once the sender has the token, data is forwarded hop-by-hop from the node after the sender (`s+1 % n`) around the ring until it reaches `r`. Every intermediate node prints a forwarding message. The receiver confirms receipt.

---

### Token Update

```java
token = s;
```
After each transmission, the token remains at the sender node. In the next round, the token will travel from `s` to the new sender.

---

### Exception Handling

```java
} catch (Exception e) {
    System.out.println("Error occurred: " + e.getMessage());
}
```
Catches any input mismatch (e.g., letters instead of numbers) or runtime errors and prints a message instead of crashing with a stack trace.

---

## 2. Concept — Token Ring Network

### Overview
**Token Ring** is a LAN (Local Area Network) technology where all nodes are arranged in a logical ring. A special control frame called a **token** circulates continuously around the ring. Only the node that **holds the token** is allowed to transmit data, eliminating collisions entirely.

### How It Works
1. A single **token** (a small data frame) continuously circulates around the ring.
2. A node that wants to send data **waits** until the token reaches it.
3. The node **captures the token**, replaces it with a data frame, and transmits.
4. The data frame travels **hop by hop** around the ring, being forwarded by each intermediate node.
5. Upon reaching the destination, the receiver copies the data and marks the frame as received.
6. The frame continues back to the sender, who then **releases a new token** onto the ring.

### Key Properties
| Property | Detail |
|---|---|
| Topology | Logical ring (physical star in modern 802.5) |
| Access method | Controlled / deterministic (no collisions) |
| Standard | IEEE 802.5 |
| Speed | 4 Mbps or 16 Mbps (classic) |
| Token holder | Only one node transmits at a time |
| Fairness | Every node gets a turn — no starvation |

### Advantages
- **No collisions** — only token holder transmits
- **Deterministic** — predictable maximum wait time
- **Fair access** — round-robin by design
- **Works well under heavy load** — unlike CSMA/CD which degrades with load

### Disadvantages
- **Single point of failure** — if any node or cable breaks, the ring fails
- **Slower than Ethernet** under light load (must wait for token)
- **Token loss** — if the token is lost (node crash), the ring stalls until a monitor regenerates it
- **Complex management** — requires an Active Monitor to manage the ring

### Token Ring in Distributed Systems
- Demonstrates **mutual exclusion** using a hardware/protocol-level mechanism
- The token acts as a **distributed lock** — only the holder has permission to use the shared medium
- Conceptually related to **software token-based mutual exclusion** algorithms (Suzuki-Kasami, Raymond's tree)
- Used in **industrial networks** (PROFIBUS, industrial Ethernet rings) where determinism is critical

### Token Ring vs Ethernet (CSMA/CD)
| Feature | Token Ring | Ethernet |
|---|---|---|
| Access Control | Token passing | Collision detection |
| Collisions | None | Possible |
| Load behavior | Stable under load | Degrades under load |
| Fairness | Guaranteed | Not guaranteed |
| Complexity | Higher | Lower |
| Common use | Industrial/legacy | Modern LANs |

---

## 3. Terminologies & QnAs

**Q: What is a token in Token Ring?**
A: A token is a special 3-byte control frame that grants permission to transmit. It circulates continuously around the ring. Only the node holding the token may send data.

**Q: What is the role of the Active Monitor?**
A: A designated node on the ring that ensures the token is not lost. If the token disappears (e.g., a node crashes mid-transmission), the Active Monitor regenerates it after a timeout.

**Q: What is token starvation?**
A: It doesn't occur in Token Ring — every node is guaranteed a turn. This is one of Token Ring's key advantages over CSMA/CD.

**Q: What does `i % n` do in the code?**
A: It implements wrap-around (modulo arithmetic) so the ring loops: after node `n-1`, the next node is `0`, simulating a circular structure.

**Q: Why does token stay at `s` after transmission?**
A: `token = s` means the next cycle starts the token from the sender's position, simulating the sender releasing the token after use.

**Q: What is the difference between token passing and CSMA/CD?**
A: Token passing is deterministic — you wait for the token, then transmit. CSMA/CD (Ethernet) is probabilistic — you transmit and retry if a collision occurs.

**Q: Can two nodes transmit simultaneously in Token Ring?**
A: No. Only one token exists in the ring at any time, guaranteeing mutual exclusion at the network level.

**Q: What happens if a node holding the token crashes?**
A: The token is lost. The Active Monitor detects no token circulation after a timeout and regenerates a new free token.

**Q: What is beaconing in Token Ring?**
A: When a node detects a ring failure, it sends beacon frames identifying the fault domain (the node upstream), helping administrators locate the broken link.

**Q: How is this code different from real Token Ring?**
A: The code simulates the logical behavior (token travel + data forwarding) but skips real networking, acknowledgment frames, token regeneration, and the Active Monitor.

---

## 4. Execution Steps

### Prerequisites
- Java JDK installed (`java`, `javac` available in PATH)

### Step 1 — Compile
```bash
javac Tring.java
```
Expected: No output (successful compilation produces `Tring.class`)

### Step 2 — Run
```bash
java Tring
```

### Step 3 — Sample Interaction

```
Enter the number of nodes: 4
 0 1 2 3 0              ← Ring displayed, wraps to 0

Enter sender: 1
Enter receiver: 3
Enter Data: Hello
Token passing: 0-> 1    ← Token travels from 0 to sender 1
Sender 1 sending data: Hello
Data Hello forwarded by 2   ← Hops through intermediate nodes
Receiver 3 received data: Hello

Enter sender: 0
Enter receiver: 2
Enter Data: Hi
Token passing: 1-> 2-> 3-> 0  ← Token travels from 1 to sender 0
Sender 0 sending data: Hi
Data Hi forwarded by 1
Receiver 2 received data: Hi
```

### To Exit
Enter non-numeric input (e.g., press `Ctrl+C` or type a letter when prompted for a number) — the `catch` block will print an error message and terminate.

> **Tip:** The number of nodes `n` determines the ring size. Nodes are numbered `0` to `n-1`. Do not enter a sender or receiver number ≥ n, as this may cause unexpected loop behavior.
