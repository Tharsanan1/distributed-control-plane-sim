# Distributed Control Plane Simulator

[**View Live Simulation**](https://tharsanan1.github.io/distributed-control-plane-sim/)

A sophisticated interactive visualization of a **High-Availability Distributed Control Plane**. This simulation demonstrates advanced concepts in distributed systems, specifically focusing on **Leader Election**, **Partition Tolerance**, and **State Reconciliation** in a cloud-native environment.

## 🧠 Core Strategy: Connectivity-Aware Leader Election

The simulation implements a specific resilience pattern often found in critical control planes (like **Google's Chubby**, **Kubernetes Controller Manager**, or **Etcd-based systems**), but with an added twist: **"End-to-End Reachability"**.

### The Problem it Solves
In standard leader election, a node can become a leader as long as it can talk to the database (the Lock Store).
*   **The Issue**: A leader might be connected to the Database but *disconnected* from the outside world (Load Balancer/Gateways). This results in a "Zombie Leader" that holds the lock but cannot actually serve traffic or control the data plane.

### The Solution: Physical Readiness Probes
In this simulation, before a Server Node attempts to acquire the Leader Lock:
1.  **Probe Phase**: It first sends a physical "Probe Packet" (Blue Dot 🔵) to the Load Balancer (LB).
2.  **Verification**: The packet must bounce off the LB and return to the Server.
3.  **Acquisition**: Only *after* confirming it is reachable from the LB does it attempt to write the "Lock" (Orange Packet 🟠) to the Database.

This implements a **Fencing** strategy based on network topology, ensuring that the Leader is not just "alive" but "useful".

## 🏗 Real-World Parallels

### 1. Kubernetes (Lease API)
*   **Simulation**: `db.leaseTime` and the "Leader" state.
*   **Real World**: Kubernetes Control Plane components (Scheduler, Controller Manager) use the `Lease` object in API Server to elect a leader. If the leader stops renewing the lease (process crash or network cut), a new leader is elected.

### 2. Network Partition Handling (Split-Brain Protection)
*   **Simulation**: The "Red X" (Trash) and Shockwaves when requesting during a partition.
*   **Real World**: Systems like **Zookeeper** or **Consul** use Quorums. This simulation visualizes the *client-side* impact of partitions (requests failing) and the *control-plane* impact (Leader stepping down if isolated).

### 3. Load Balancer Health Checks
*   **Simulation**: The LB stops routing packets to Gateways with "Cut Links" (dashed red lines).
*   **Real World**: AWS ALB / Nginx / Envoy performing active health checks. If a backend is unreachable (failed TCP handshake), the LB removes it from the rotation to prevent blackholing traffic.

## 🎮 How to Play / Demo

### Basic Workflow
1.  **Deploy**: Click `Deploy` in the API Panel.
2.  **Observation**: Watch the `REQ` -> `ACK` flow. A Leader is elected (Gold Ring). The Leader then syncs the configuration to the Gateways.
3.  **Traffic**: Click `Deploy` again or `Update` to generate user traffic.

### Chaos Engineering (The Fun Part)
1.  **Simulate Partition**:
    *   **Left Click** on the line connecting the Current Leader and the Load Balancer.
    *   *Result*: The link becomes "Cut" (dashed red).
    *   *Watch*: The Leader might try to renew, but if you force a re-election (Kill the leader), the next candidate will **FAIL** to win if its link is also cut.
2.  **Simulate Crash**:
    *   **Right Click** on any Node.
    *   *Result*: Node goes `DEAD` (Skull 💀). Packets routed there will explode.
3.  **Scale Out**:
    *   Use the **Cluster Scaling** buttons to add more API Servers or Gateways.
    *   Observe how the Load Balancer intelligently distributes (or fails to distribute) traffic across the new topology.

## 🎨 Visual Legend
*   **🟡 Gold Ring**: Valid Leader (Holds Lease).
*   **🔵 Blue Packet**: Connectivity Probe (Readiness Check).
*   **🟠 Orange Packet**: Lock Acquisition / Lease Renewal.
*   **🔴 Red X / Shockwave**: Discarded Request (Rejection/Drop).
*   **⚪️ White Packet**: Standard User Traffic.
*   **Dashed Red Line**: Severed Network Link.

## Technical Details
*   **Engine**: Custom HTML5 Canvas Physics Engine.
*   **Logic**: Tick-based simulation loop (60 logical ticks/sec).
*   **State Machine**: Nodes operate as independent agents with local state (`Idle`, `Candidate`, `Leader`, `Dead`).
