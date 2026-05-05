# Assignment 4 — Clock Synchronization (Berkeley Algorithm)
### Files: `client.py` & `server.py`

---

## 1. Code Explanation

### `server.py` — Master Node (Clock Server)

```python
client_data = {}
```
A shared dictionary that stores each connected client's address, their reported clock time, and the time difference from the server.

```python
def startReceivingClockTime(connector, address):
    clock_time_string = connector.recv(1024).decode()
    clock_time = parser.parse(clock_time_string)
    clock_time_diff = datetime.datetime.now() - clock_time
    client_data[address] = { "clock_time": clock_time, "time_difference": clock_time_diff, "connector": connector }
```
Each connected client gets its own thread. The server receives the client's local time, computes how far it is from the server's own clock, and stores this difference.

```python
def startConnecting(master_server):
    master_slave_connector, addr = master_server.accept()
    current_thread = threading.Thread(target=startReceivingClockTime, ...)
    current_thread.start()
```
The master thread runs in a loop, accepting new client connections. For each accepted connection, a dedicated receiver thread is spawned.

```python
def getAverageClockDiff():
    sum_of_clock_difference = sum(time_difference_list, datetime.timedelta(0, 0))
    average_clock_difference = sum_of_clock_difference / len(client_data)
```
Computes the **average clock difference** across all connected clients — the heart of the Berkeley Algorithm.

```python
def synchronizeAllClocks():
    synchronized_time = datetime.datetime.now() + average_clock_difference
    client['connector'].send(str(synchronized_time).encode())
```
Every 5 seconds, the server calculates the average offset and sends each client an adjusted/synchronized time.

---

### `client.py` — Slave Node

```python
def startSendingTime(slave_client):
    slave_client.send(str(datetime.datetime.now()).encode())
    time.sleep(5)
```
Sends the client's current local time to the server every 5 seconds so the server can calculate the clock drift.

```python
def startReceivingTime(slave_client):
    Synchronized_time = parser.parse(slave_client.recv(1024).decode())
    print("Synchronized time at the client is: " + str(Synchronized_time))
```
Receives the synchronized time sent back by the server and displays it.

```python
def initiateSlaveClient(port=8080):
    slave_client = socket.socket()
    slave_client.connect(('127.0.0.1', port))
```
Creates a TCP socket and connects to the server at `127.0.0.1:8080`. Then launches two threads — one for sending, one for receiving — so both happen concurrently.

---

## 2. Concept — Berkeley Clock Synchronization Algorithm

### Overview
The **Berkeley Algorithm** is a method for **internal clock synchronization** in distributed systems. Unlike NTP (which syncs to an external time source), Berkeley uses an internal master node to average out the clocks of all participating nodes.

### How It Works
1. **Master polls** all slave nodes for their current local clock time.
2. Each slave sends its timestamp to the master.
3. The master computes the **average time** across itself and all slaves, factoring out faulty outliers.
4. The master sends each node a **correction offset** (not the absolute time), telling each how much to adjust.
5. Each slave adjusts its clock accordingly.

### Key Properties
| Property | Detail |
|---|---|
| Type | Internal synchronization |
| Initiator | Master/server node |
| What is sent | Clock offset (delta), not absolute time |
| Fault tolerance | Outlier clocks can be excluded from averaging |
| Use case | Systems where no external time source is available |

### Why Not Send Absolute Time?
Sending the corrected absolute time would introduce network delay errors. Sending an **offset** (±Δt) is more accurate because the slave adjusts relative to its own clock.

### Distributed Systems Context
- Used in **LANs** where nodes need a consistent notion of time
- Important for **event ordering**, **transaction logs**, **distributed databases**
- Predecessor concept to more modern protocols like **PTP (IEEE 1588)**
- Contrasted with **Cristian's Algorithm** (client asks server for time — simpler but single point of failure)

### Comparison: Berkeley vs Cristian's Algorithm
| Feature | Berkeley | Cristian's |
|---|---|---|
| Time source | Internal average | External server |
| Direction | Server polls clients | Client polls server |
| Accuracy | Better (averaged) | Single server dependent |
| Fault tolerance | Higher | Lower |

---

## 3. Terminologies & QnAs

**Q: What is clock drift?**
A: The gradual divergence of a clock from the actual time due to hardware imperfections (crystal oscillator variation). All computer clocks drift slightly over time.

**Q: What is clock skew?**
A: The difference in time readings between two clocks at a single point in time. Skew is the result of accumulated drift.

**Q: What is internal vs external synchronization?**
A: External sync aligns clocks to a universal standard (e.g., UTC via NTP). Internal sync only ensures clocks agree with each other, not necessarily with the real world.

**Q: Why does the Berkeley algorithm use averaging?**
A: Averaging smooths out individual clock errors. No single node's clock is trusted absolutely; the consensus of all clocks reduces individual biases.

**Q: What is the role of threading in this code?**
A: Threading allows the server to handle multiple clients simultaneously and the client to send/receive at the same time without blocking.

**Q: What port is used and why?**
A: Port `8080` — a commonly used non-privileged alternative to port 80, suitable for local application communication.

**Q: What happens if a client disconnects?**
A: The server's `recv()` will throw an exception. The code doesn't handle reconnections explicitly, but the `client_data` dict retains the last known state.

**Q: What is `timedelta` used for?**
A: `datetime.timedelta` represents the difference between two datetime objects. It is used here to store and average the clock differences between server and clients.

---

## 4. Execution Steps

### Prerequisites
```bash
pip install python-dateutil
```

### Terminal 2 — Start the Server First
```bash
python server.py
```
Expected output:
```
Socket at master node created successfully
Clock server started...
Starting to make connections...
Starting synchronization parallelly...
New synchronization cycle started.
Number of clients to be synchronized: 0
No client data. Synchronization not applicable.
```

### Terminal 1 — Start the Client
```bash
python client.py
```
Expected output:
```
Starting to receive time from server
Starting to receiving synchronized time from server
Recent time sent successfully

Synchronized time at the client is: 2026-05-05 09:51:21.904966
Recent time sent successfully
Synchronized time at the client is: 2026-05-05 09:51:26.905324
```

### Server output after client connects:
```
127.0.0.1:XXXXX got connected successfully
Client Data updated with: 127.0.0.1:XXXXX
New synchronization cycle started.
Number of clients to be synchronized: 1
```

> **Note:** Always start `server.py` before `client.py`, as the client immediately attempts to connect on startup.
> Multiple clients can be connected simultaneously by opening more terminals and running `python client.py` in each.
