# Distributed Systems Lab — SPPU Final Year IT (2019 Pattern)

> **University:** Savitribai Phule Pune University (SPPU)  
> **Year:** Final Year IT  
> **Pattern:** 2019  
> **Subject:** Distributed Systems

---

## Assignments Covered

| # | Assignment | Concept | Language | OS |
|---|---|---|---|---|
| Assignment 1 | Implement multi-threaded client/server Process communication using RMI. | Remote Method Invocation | Java | Ubuntu / Windows |
| Assignment 4 | Implement Berkeley algorithm for clock synchronization. | Clock Synchronization | Python | Ubuntu |
| Assignment 5 | Implement token ring based mutual exclusion algorithm. | Token Ring / Mutual Exclusion | Java | Ubuntu / Windows |
| Assignment 6 | Implement Bully and Ring algorithm for leader election. | Election Algorithms | Java | Ubuntu / Windows |

---


---

## OS Recommendation per Assignment

| Assignment | Recommended OS | Reason |
|---|---|---|
| Assignment 1 (RMI) | **Ubuntu** (preferred) or Windows | `rmiregistry` runs cleaner on Linux; no path issues |
| Assignment 4 (Clock Sync) | **Ubuntu** | Python socket/threading works best on Linux; `dateutil` installs cleanly |
| Assignment 5 (Token Ring) | **Ubuntu** or Windows | Pure Java, runs on both |
| Assignment 6 (Election) | **Ubuntu** or Windows | Pure Java, runs on both |

> **General Recommendation:** Use **Ubuntu 20.04 / 22.04 LTS** for all assignments. Linux handles multi-process/multi-terminal workflows, sockets, and Java tooling more reliably than Windows, with no firewall or path separator issues.

---

## Installation Guide

### Ubuntu — Install Java (JDK 21)

```bash
# Update package list
sudo apt update

# Install Java Development Kit
sudo apt install default-jdk -y

# Verify installation
java -version
javac -version

# Expected output:
# openjdk version "21.x.x"
# javac 21.x.x
```

> Java is **not pre-installed** on a fresh Ubuntu. The above installs OpenJDK 21 which works for all Java assignments (1, 5, 6).

---

### Ubuntu — Install Python 3 & dateutil (Assignment 4)

```bash
# Python 3 is pre-installed on Ubuntu 20.04+
# Verify:
python3 --version

# Install pip if not present
sudo apt install python3-pip -y

# Install required library
pip install python-dateutil --break-system-packages

# Verify:
python3 -c "from dateutil import parser; print('dateutil OK')"
```

---

### Windows — Install Java (JDK 21)

```powershell
# Option 1: Using winget (Windows 10/11)
winget install Microsoft.OpenJDK.21

# Option 2: Manual download
# Go to: https://adoptium.net/
# Download: OpenJDK 21 LTS (Windows x64 MSI installer)
# Run the installer — tick "Set JAVA_HOME" and "Add to PATH"

# Verify in a new Command Prompt / PowerShell:
java -version
javac -version
```

---

### Windows — Install Python 3 (Assignment 4)

```powershell
# Option 1: Using winget
winget install Python.Python.3.12

# Option 2: Manual download
# Go to: https://www.python.org/downloads/
# Download Python 3.12.x Windows installer
# During install: CHECK "Add python.exe to PATH"

# Verify in a new terminal:
python --version

# Install required library:
pip install python-dateutil
```

---

## Assignment 1 — RMI

**Problem Statement:** Implement multi-threaded client/server process communication using RMI.

**Files:** `AddServerIntf.java`, `AddServerImpl.java`, `AddServer.java`, `AddClient.java`

**Quick Run:**
```bash
# Terminal 1
javac *.java

# Terminal 2
rmiregistry

# Terminal 3
java AddServer

# Terminal 4
java AddClient 127.0.0.1 5 8
```

**Expected Output:**
```
The first number is: 5
The second number is: 8
The sum is: 13.0
```


---

## Assignment 4 — Clock Synchronization

**Problem Statement:** Implement Berkeley algorithm for clock synchronization.

**Files:** `server.py`, `client.py`

**Quick Run:**
```bash
# Terminal 1 (server first!)
python3 server.py

# Terminal 2
python3 client.py
```

**Expected Output (client):**
```
Starting to receive time from server
Synchronized time at the client is: 2026-05-05 09:51:21.904966
```

---

## Assignment 5 — Token Ring

**Problem Statement:** Implement token ring based mutual exclusion algorithm.

**File:** `Tring.java`

**Quick Run:**
```bash
javac Tring.java
java Tring
# Enter: number of nodes, then sender, receiver, data
```

**Expected Output:**
```
Enter the number of nodes: 4
 0 1 2 3 0
Enter sender: 1
Enter receiver: 3
Enter Data: Hello
Token passing: 0-> 1
Sender 1 sending data: Hello
Data Hello forwarded by 2
Receiver 3 received data: Hello
```

---

## Assignment 6 — Election Algorithms

**Problem Statement:** Implement Bully and Ring algorithm for leader election.

**Files:** `Bully.java`, `Ring.java`

**Quick Run:**
```bash
# Bully
javac Bully.java && java Bully

# Ring
javac Ring.java && java Ring

# In both: Choose 1 (create) → 4 (down coordinator) → 5 (run election)
```

**Expected Output (Bully):**
```
Process P4 is the coordinator
[Down P4]
Election message sent from process 2 to process 3
Process P3 is the coordinator
```

---

## Key Concepts Quick Reference

| Concept | Assignment | One-liner |
|---|---|---|
| RMI | 1 | Call methods on remote JVM objects as if local |
| Berkeley Algorithm | 4 | Master averages all clocks and sends correction offsets |
| Token Ring | 5 | Only the token-holder may transmit — no collisions |
| Bully Algorithm | 6 | Highest alive process ID wins the election |
| Ring Election | 6 | IDs circulate the ring; max ID becomes coordinator |

---

## Notes

- Always start **server before client** in Assignment 1 and 4.
- For Assignment 1 on **JDK 15+**, skip the `rmic` step — stubs are generated dynamically.
- For Assignment 6, always run **menu option 1 first** to initialize processes before other options.
- All assignments tested on **Ubuntu 24.04, OpenJDK 21, Python 3.12**.
