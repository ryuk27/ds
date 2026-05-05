# Assignment 1 — Multi-threaded Client/Server Communication using RMI
### Files: `AddServerIntf.java`, `AddServerImpl.java`, `AddServer.java`, `AddClient.java`

---

## 1. Code Explanation

### `AddServerIntf.java` — Remote Interface

```java
import java.rmi.*;
public interface AddServerIntf extends Remote {
    double add(double d1, double d2) throws RemoteException;
}
```
Defines the **contract** between client and server. Any interface intended for remote invocation must:
- Extend `java.rmi.Remote` — marks it as a remotely callable interface.
- Declare all methods to `throw RemoteException` — because network calls can fail at any time (connection drops, timeout, etc.).

This file is shared conceptually between client and server — both sides must agree on what methods exist and their signatures.

---

### `AddServerImpl.java` — Server Implementation

```java
public class AddServerImpl extends UnicastRemoteObject implements AddServerIntf {
    public AddServerImpl() throws RemoteException {}
    public double add(double d1, double d2) throws RemoteException {
        return d1 + d2;
    }
}
```
- Extends `UnicastRemoteObject` — makes the object exportable over the network using TCP/IP. It handles all low-level socket management, marshalling, and unmarshalling automatically.
- The constructor must `throw RemoteException` because the parent `UnicastRemoteObject` constructor can throw it during object export.
- `add()` is the actual business logic — simply returns `d1 + d2`. In a real system this could be any complex computation running on the server.

---

### `AddServer.java` — Server Main (Registry Binding)

```java
AddServerImpl addServerImpl = new AddServerImpl();
Naming.rebind("AddServer", addServerImpl);
```
- Creates an instance of the remote object.
- `Naming.rebind("AddServer", addServerImpl)` registers it with the **RMI Registry** under the name `"AddServer"`. If a binding already exists with that name, it is replaced (`rebind` vs `bind`).
- After this line, the server is live and waiting — the object is accessible remotely.

---

### `AddClient.java` — Client (Remote Lookup & Call)

```java
String addServerURL = "rmi://" + args[0] + "/AddServer";
AddServerIntf addServerIntf = (AddServerIntf) Naming.lookup(addServerURL);
System.out.println("The sum is: " + addServerIntf.add(d1, d2));
```
- Constructs the RMI URL: `rmi://127.0.0.1/AddServer`
- `Naming.lookup()` contacts the RMI Registry at the given host, retrieves a **stub** (proxy) for the remote object, and returns it cast to the interface type.
- `addServerIntf.add(d1, d2)` looks like a local method call but is actually transmitted over the network to the server, executed there, and the result is sent back.
- `args[0]` = server IP, `args[1]` = first number, `args[2]` = second number — passed as command-line arguments.

---

### How RMI Stub Works (Behind the Scenes)
```
Client                      Network                     Server
  |                            |                            |
  |-- addServerIntf.add(5,8) ->|                            |
  |       [Stub serializes]    |-- TCP call to server ----> |
  |                            |                            |-- AddServerImpl.add(5,8)
  |                            |                            |   returns 13.0
  |                            |<-- result sent back -------|
  |<-- 13.0 returned ----------|                            |
```
The **stub** on the client side marshals (serializes) the arguments, sends them via TCP to the server's **skeleton**, which demarshals and invokes the actual method, then returns the result the same way.

---

## 2. Concept — Java RMI (Remote Method Invocation)

### Overview
**Java RMI** is a Java API that enables an object in one JVM to invoke methods on an object in another JVM — even on a different machine — as if they were local calls. It is Java's native mechanism for **distributed object computing**.

### Architecture

```
┌─────────────────────────┐        ┌────────────────────────────┐
│        CLIENT JVM        │        │         SERVER JVM          │
│                          │        │                            │
│  AddClient               │        │  AddServer                 │
│     │                    │        │     │                      │
│     └──> Stub (Proxy)  ──┼──TCP───┼──> Skeleton               │
│           (AddServerIntf)│        │       │                    │
│                          │        │       └──> AddServerImpl   │
│                          │        │                            │
└─────────────────────────┘        └────────────────────────────┘
              │                                    │
              └──────── RMI Registry ──────────────┘
                        (lookup/bind)
```

### Key Components

| Component | Role |
|---|---|
| `Remote` interface | Contract defining remotely callable methods |
| `UnicastRemoteObject` | Base class that exports the object over TCP |
| `rmiregistry` | Naming service — maps names to remote objects |
| `Naming.rebind()` | Registers a remote object with the registry |
| `Naming.lookup()` | Client retrieves stub from registry |
| Stub | Client-side proxy that forwards calls to server |
| Skeleton | Server-side dispatcher (auto-generated in modern Java) |

### RMI in Distributed Systems
- Enables **location transparency** — the client doesn't need to know where the server is physically
- Supports **distributed computing** — offload computation to powerful servers
- Foundation for **EJB (Enterprise JavaBeans)** and early web services
- Uses **Java serialization** for passing objects between JVMs
- All remote objects must be **serializable** to pass as arguments or return values

### Advantages
- Strongly typed — compile-time checking of remote calls
- Native Java — no external libraries needed
- Supports passing complex Java objects as parameters
- Automatic garbage collection for remote objects

### Disadvantages
- **Java-only** — cannot interoperate with other languages (unlike REST or CORBA)
- Tightly coupled — client and server must share interface classes
- Firewall unfriendly — uses dynamic ports
- Largely superseded by REST APIs, gRPC, and web services in modern systems

### RMI vs REST vs gRPC

| Feature | RMI | REST | gRPC |
|---|---|---|---|
| Language | Java only | Any | Any |
| Protocol | Java serialization over TCP | HTTP/JSON | HTTP/2 + Protobuf |
| Coupling | Tight | Loose | Moderate |
| Modern usage | Legacy | Very common | Microservices |

---

## 3. Terminologies & QnAs

**Q: What is RMI?**
A: Remote Method Invocation — a Java API that allows methods of objects in one JVM to be called from another JVM across a network, making distributed computing transparent to the programmer.

**Q: What is the RMI Registry?**
A: A simple naming service (like a phonebook) that maps string names to remote objects. The server registers its object; the client looks it up by name to get a stub.

**Q: What is a stub in RMI?**
A: A client-side proxy object that implements the same interface as the remote object. It intercepts method calls, serializes arguments, sends them over the network, and returns the deserialized result.

**Q: What is `UnicastRemoteObject`?**
A: A base class that handles exporting the remote object — it opens a TCP socket, listens for incoming calls, and dispatches them to the actual implementation.

**Q: Why must remote methods throw `RemoteException`?**
A: Because network communication can fail — the server might be down, the network might drop packets, or a timeout might occur. `RemoteException` signals these network-level failures to the caller.

**Q: What is `Naming.rebind()` vs `Naming.bind()`?**
A: `bind()` throws an exception if the name is already registered. `rebind()` silently replaces any existing binding — safer for server restarts.

**Q: What is marshalling/unmarshalling?**
A: Marshalling = converting Java objects into a byte stream for network transmission. Unmarshalling = reconstructing the object from the byte stream at the destination. RMI uses Java serialization for this.

**Q: What is `rmic` and is it needed in modern Java?**
A: `rmic` was a tool that pre-generated stub class files. Since JDK 5+, stubs are generated dynamically at runtime. `rmic` was deprecated in JDK 13 and removed in JDK 15+. You do **not** need it with JDK 15 or newer.

**Q: What port does `rmiregistry` run on?**
A: Default port **1099**. Can be changed: `rmiregistry 1100`.

**Q: What is location transparency in RMI?**
A: The client code looks identical whether the remote object is on the same machine or a server across the world — the stub hides all network details.

**Q: How is RMI different from a regular method call?**
A: A local call happens in the same JVM with no network. An RMI call crosses a network, involves serialization, TCP communication, and can fail with `RemoteException` — none of which exist in local calls.

**Q: What is a distributed object system?**
A: A system where objects can reside on different machines and invoke each other's methods transparently. RMI, CORBA, and DCOM are examples.

---

## 4. Execution Steps

### Prerequisites
- Java JDK installed (JDK 8–21 all work; JDK 15+ does not need `rmic`)
- All four `.java` files in the same directory

### Step 1 — Terminal 1: Compile all files
```bash
javac *.java
```
Expected: No output = success. You'll see `AddServerIntf.class`, `AddServerImpl.class`, `AddServer.class`, `AddClient.class` generated.

> **Note:** On JDK 8–14, also run:
> ```bash
> rmic AddServerImpl
> ```
> This generates `AddServerImpl_Stub.class`. On JDK 15+, skip this — stubs are generated automatically at runtime.

### Step 2 — Terminal 2: Start the RMI Registry
```bash
rmiregistry
```
Expected: No output — it runs silently in the background, listening on port 1099.
> Keep this terminal open throughout.

### Step 3 — Terminal 3: Start the Server
```bash
java AddServer
```
Expected output:
```
AddServer bound and ready.
```
> Keep this terminal open. The server is now registered in the RMI registry.

### Step 4 — Terminal 4: Run the Client
```bash
java AddClient 127.0.0.1 5 8
```
Expected output:
```
The first number is: 5
The second number is: 8
The sum is: 13.0
```

### Argument Format
```
java AddClient <server-ip> <number1> <number2>
```
- `127.0.0.1` = localhost (same machine). Replace with actual server IP for remote runs.
- Supports any decimal numbers: `java AddClient 127.0.0.1 3.14 2.72`

### Troubleshooting

| Error | Cause | Fix |
|---|---|---|
| `Connection refused` | rmiregistry not running | Start rmiregistry first |
| `NotBoundException` | Server not started | Start AddServer before client |
| `ClassNotFoundException` | Missing .class files | Recompile with `javac *.java` |
| `rmic: command not found` | JDK 15+ | Skip rmic — not needed |
