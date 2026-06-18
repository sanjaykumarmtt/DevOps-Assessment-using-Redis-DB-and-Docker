# 🚀 Redis Cluster Orchestration & Zero-Downtime Rolling Upgrade Tool

A production-grade CLI automation utility designed to provision, seed, audit, and upgrade a 6-node Redis Cluster (3 Masters, 3 Replicas) with zero client-visible downtime and 100% data integrity.

---

## 🛠️ Pre-Requisites: Cluster Infrastructure Setup & OS Network Routing

### 1️⃣ Linux / Ubuntu Native Execution
This tool runs 100% natively on Linux and Ubuntu operating systems without requiring any additional network modifications. 
Once the containers are started using the `docker compose up -d` command, the bridge network IPs (`10.10.0.11` - `10.10.0.16`) will automatically route and connect directly with the local host system.

### 2️⃣ macOS MacBook Architecture Constraint & Fix

> ⚠️ **Connection Warning:** Since Docker on macOS does not run natively and operates inside a lightweight Virtual Machine (VM), a **Connection Refused** or **Network Unreachable** error may occur when the Java tool attempts to communicate with the container IPs. 

To resolve this network routing constraint and establish a seamless connection between the Java tool and the containers, you must configure the following 3 steps in Docker Desktop upon opening it:

- ⚙️ **Option A:** Click on the **Settings** (gear icon) section located at the top of the Docker Desktop application.
- 🔀 **Option B:** Navigate to the **Resources ➡️ Network** section in the left-hand menu sidebar.
- ✅ **Option C:** Check the box for **Enable IP routing** (or **Allow inter-container and host routing**) and click the **Apply & Restart** button.

### 3️⃣ Ansible SSH Authentication & Key Management Configuration
Since the custom Ansible playbooks orchestrate container lifecycle actions and configuration management securely via standard protocols, you must configure automated SSH key-based authentication before executing any tool commands. Follow these commands to generate and structure the keys appropriately within the target workspace:

- 🔑 **Step A: Generate a clean SSH Keypair**
  Open your host terminal and run the following command to create a secure keypair without a passphrase:
  ```bash
  ssh-keygen -t rsa -b 4096 -f ~/.ssh/redis_cluster_key -N ""
  ```

- 📋 **Step B: View and Copy the SSH Public Key to the Target Location**
  Run the following command to display your generated public key in the terminal. Copy the entire output string:
  ```bash
  cat ~/.ssh/redis_cluster_key.pub
  ```
  After copying the key output, paste it precisely into a file named **authorized_keys** and place it exactly at the root of your infra folder structure:
  ```bash
        /submission/infra/authorized_keys
  ```

- 🖥️ **Step C: Register the Key with the Host System**
  Run the following command to make your host system recognize this authentication key, enabling passwordless orchestration for Ansible:
  ```bash
  cat infra/authorized_keys >> ~/.ssh/authorized_keys
  chmod 600 ~/.ssh/authorized_keys
  ```

### 4️⃣ Initialize & Build Container Infrastructure
To ensure a clean environment free of port conflicts or stale volume configurations, completely tear down any existing architecture before executing a fresh initialization. Choose the appropriate runtime option below based on the container engine available on your host system:

- 🐳 **Option A: Using Docker Engine (Standard Execution)**
  If your host system runs Docker, execute the following commands from the root submission directory to clean up and fresh-build the 6-node Redis cluster:
  ```bash
  cd /submission/infra
  docker compose down
  docker compose up -d --build
  ```

- 🦭 **Option B: Using Podman (Fully Open Source Alternative)**
  If your host system utilizes Podman as the preferred open-source container runtime, use these drop-in replacement commands to safely deploy the exact same 6-node setup:       
  ```bash
  cd /submission/infra
  podman-compose down
  podman-compose up -d --build                 
  ```

---

## 💻 Orchestrating the redis-tool CLI

Once the container infrastructure is up and running, you can orchestrate the Redis cluster deployment using the compiled `redis-tool` executable.

### 📂 Initial Workspace Directory Check
Before running any tool commands, you must ensure that your terminal is pointed exactly at the root submission folder.

> 📍 **Strict Directory Rule:** All commands must be executed strictly from the `/submission` directory where the tool binary resides.

```bash
cd /submission
```

### 🔍 Step 1 / Phase 0: Initial Infrastructure Status Verification
Before initiating any cluster provisioning or data operations, execute the status command first. This checks the health of the container infrastructure and ensures all 6 nodes are running and reachable:
```bash
./redis-tool status
```

---

### 🚀 Phase 1: Cluster Provisioning

Execute the core provisioning command from the `/submission` directory.

> ⚠️ **Strict Constraint:** The version number (e.g., **7.2.6**) is **completely dynamic** and can be customized, but all other topology flags must remain exactly as shown below:

```bash
  provision --version 7.0.15 --masters 3 --replicas-per-master 1
```

Running this command triggers our automated Ansible pipeline to execute the following 5 critical operations without any manual intervention:
- 📦 **1. Precise Version Installation:** Automatically downloads and installs the exact Redis version specified (e.g., 7.2.6 or any dynamic version) across all 6 isolated container nodes.
- ⚙️ **2. Cluster Mode Configuration:** Updates and locks the core Redis configurations on each node (`cluster-enabled yes`, custom `cluster-config-file`, optimized `cluster-node-timeout`, and correct network binds/ports).
- 🔄 **3. Automated Service Daemon:** Safely starts and monitors the active Redis server instances across the entire infrastructure.
- 🏗️ **4. Topology Formation:** Dynamically creates the cluster architecture, establishing 3 dedicated Master nodes with allocated hash slot ranges and assigning the remaining 3 nodes as their respective Replicas.
- 🖨️ **5. Live Topology Printout:** Outputs a clean visual layout of the finalized cluster structure directly in your terminal, detailing node roles, active slot ranges, and current connectivity status.

---

### 📊 Phase 2: Seed Data and Verify

This phase is used to populate the cluster with deterministic data and verify its integrity both before and after the rolling upgrade process to guarantee zero data loss.

#### 1. Populate the Cluster (Data Seed)
Execute this command from the `/submission` directory to generate and insert the initial dataset:
```bash
   data seed --keys 1000
```

> ⚠️ **Strict Constraint (Fully Dynamic):** The total number of keys (e.g., **1000**) is completely dynamic. Evaluators can customize this value to any integer size (such as `--keys 2000` or `--keys 5000`), and the CLI will dynamically scale data generation to match that exact count.

**What happens behind the scenes:**
- 🔑 **Dynamic & Deterministic Generation:** Automatically generates the exact number of key-value pairs specified via the flag using a reproducible cryptographic scheme (e.g., `key:0001` → SHA256 hash of the key) so values can be independently verified later.
- 🧩 **Smart Cluster Distribution:** Inserts the generated keys into the cluster, where they dynamically distribute across the 3 Master nodes based on Redis hash slots.
- 📝 **Execution Summary:** Prints a clean summary layout showing total keys successfully inserted, exact key distribution counts across each Master node, and any insertion failures.

#### 2. Verify Data Integrity (Data Verify)
Execute this command to check data consistency and prove that our cluster architecture retains 100% of its dataset:
```bash
redis-tool data verify
```

**What happens behind the scenes:**
- 📖 **Full Dataset Readback:** Reads back all the dynamically generated keys directly from the live cluster architecture.
- ⚖️ **Independent Integrity Compare:** Recomputes the expected deterministic values for each key on-the-fly and compares them with the values currently stored in Redis.
- 🏆 **Final Certification Printout:** Displays a direct cryptographic status report matching your exact flag input:
  - **PASS** — 1000/1000 keys verified (or your dynamic key count if configured higher)
  - **FAIL** — X keys missing, Y values mismatched (If any corruption occurs)

---

### 🔍 Phase 3: Cluster Status Verification

Execute this command from the `/submission` directory to audit the live architecture and inspect the real-time runtime configurations of all 6 nodes:
```bash
redis-tool status
```

**What happens behind the scenes:**
- 📡 **Comprehensive Metadata Audit:** Dynamically queries the cluster to extract each node's exact IP, port, specific Redis version, and runtime role.
- 🩺 **Master Node Diagnostics:** Maps out the dedicated hash slot ranges and calculates the exact live key distribution counts across all active Masters.
- 🗺️ **Replica Mapping:** Identifies replication links to explicitly show which Master node each Replica is currently tracking.
- 📈 **Global Health & Resource Metrics:** Evaluates the overall cluster state (`ok` or `fail`) and pulls active memory consumption metrics individually per node.

#### 📊 Structured Output Example Layout:
The CLI processes raw cluster state data and outputs a highly clean, human-readable console dashboard separated by node roles:

```text
🚀 [Initiating] Check Redis Cluster Status...

        =========================================================================
                        REDIS CLUSTER STATUS DASHBOARD                     
        =========================================================================

        [ MASTERS ]
        +-----------------+--------+----------+----------------+-------+---------+
        | IP Address      | Port   | Version  | Slots          | Keys  | Memory  |
        +-----------------+--------+----------+----------------+-------+---------+
        | 10.10.0.14      | 6379   | v7.2.6   | 10923-16383    | 343   | 1.94M   |
        | 10.10.0.15      | 6379   | v7.2.6   | 0-5460         | 333   | 1.94M   |
        | 10.10.0.16      | 6379   | v7.2.6   | 5461-10922     | 325   | 1.94M   |
        +-----------------+--------+----------+----------------+-------+---------+

        [ REPLICAS ]
        +-----------------+--------+----------+-----------------------+---------+
        | IP Address      | Port   | Version  | Replicating Master    | Memory  |
        +-----------------+--------+----------+-----------------------+---------+
        | 10.10.0.11      | 6379   | v7.2.6   | 10.10.0.15:6379       | 1.85M   |
        | 10.10.0.12      | 6379   | v7.2.6   | 10.10.0.16:6379       | 1.85M   |
        | 10.10.0.13      | 6379   | v7.2.6   | 10.10.0.14:6379       | 1.85M   |
        +-----------------+--------+----------+-----------------------+---------+
```

---

### 🔄 Phase 4: Zero-Downtime Rolling Upgrade (The Core Challenge)

This is the most critical orchestration workflow of the tool. Executing this single command automates the entire rolling upgrade across the 6-node cluster, ensuring zero client-visible downtime and maintaining an uninterrupted `ok` cluster state throughout the process.

> ⚠️ **Strict Constraint:** The target version (e.g., **7.2.6**) is **completely dynamic** and can be pointed to any valid production release.

```bash
upgrade --target-version 7.2.6
```

🌍 **Fully Dynamic Version Fetching:** The `--target-version` parameter is completely dynamic. Evaluators can pass any valid production Redis version string here. The tool is architected to dynamically fetch, download, and compile the requested version on-the-fly, ensuring the system never relies on static or pre-downloaded dependency packages.

#### 1. Pre-Flight Health Checks
Before altering any node, the tool runs a safety sweep to lock in an integrity baseline:
- ✅ Verifies global cluster state is healthy (`cluster_state:ok`) and all 6 containers are fully reachable.
- ✅ Validates that the active live version actually differs from the requested target version.
- ✅ Automatically executes a `data verify` operation to secure a pre-upgrade data integrity snapshot.

#### 2. Replica Node Upgrades (One-by-One Sequence)
To prevent traffic interruption, the tool systematically upgrades the isolated Replica nodes first:
- 🔁 **The Routine:** Gracefully stops Redis → Installs the target version (7.2.6) → Restarts Redis with identical configurations → Waits for the node to safely rejoin and complete data resynchronization.
- 🔒 **Strict Validation:** Assures the global cluster state is strictly `ok` before initiating the next node.
- 📺 **Live Console Progress:** Prints clean sequential milestone updates, e.g., `[2/6] Upgraded replica 10.10.0.15 — cluster: ok.`

#### 3. Master Node Upgrades (Graceful Failover Sequence)
Upgrading active Masters requires automated role promotion to maintain 100% uptime:
- 🚀 **Orchestrated Promotion:** Triggers a controlled `CLUSTER FAILOVER` on the corresponding (already upgraded) Replica node, promoting it instantly to the new Master.
- 🔁 **The Routine:** Once failover completes, the old Master transitions into a Replica role. The tool then halts its service, installs the target version (7.2.6), restarts it, and waits for it to rejoin the topology smoothly.
- 📺 **Live Console Progress:** Prints a validation checkpoint after each successful Master role migration.

#### 4. Post-Upgrade Integrity Certification
Once all 6 nodes are cycled, the tool certifies the deployment:
- 🛡️ Automatically triggers a final `data verify` scan—confirming all dynamically seeded keys are 100% present and untampered.
- 🔎 Re-audits the global infrastructure status to ensure every node explicitly registers the new version.
- 🏆 Prints the final project victory banner: `UPGRADE COMPLETE — all nodes on v7.2.6, data integrity verified`

---

### 🛡️ Phase 5: Full Verification (Post-Upgrade Health Audit)

After completing the rolling upgrade workflow, execute the comprehensive full verification command from the `/submission` directory to perform a deep-dive production-ready health audit on the finalized architecture:                                        

```bash
redis-tool verify --full
```

When triggered, the CLI executes a strict, multi-layered diagnostic suite covering 5 critical infrastructure metrics:

- 🔐 **1. Cryptographic Data Integrity:** Validates the entire dynamically seeded dataset (similar to `data verify`), ensuring every single key is fully readable and matches its expected deterministic SHA256 baseline value.
- 🏷️ **2. Version Consistency Audit:** Scans all 6 isolated containers simultaneously to certify that every single Master and Replica node successfully reports the exact same updated target Redis version.
- 🗺️ **3. Topology & Slot Health Map:** Audits the cluster routing engine to guarantee that all 16,384 Redis hash slots are completely covered, and strictly verifies that every single Master node has a healthy, designated Replica.
- 🔒 **4. Global Cluster State Lock:** Directly queries the active runtime configurations to confirm that the overarching `cluster_state` strictly returns an `ok` status.
- ⚡ **5. Zero Replication Lag Check:** Inspects the distributed synchronization links across the backend network, ensuring every single upgraded Replica reports a live `master_link_status:up` connection with absolutely zero packet drop.

---

## 🧠 Section 6: Engineering Strategy, Assumptions, and Limitations

### The Rolling Upgrade Strategy Implemented & Why
To achieve true zero client-visible downtime, the orchestration engine implements a highly tactical **Replica-First, Failover-Driven Master Migration** strategy. The core architecture behaves exactly like a real-world vehicle utilizing a live backup tyre (Stepney) while remaining in continuous motion:

- **Phase A: Upgrading the Child (Replica) Nodes First**
  - **The Logic:** Master nodes are the primary "Parent" nodes handling active live traffic, while Replicas act as passive "Child" nodes that act as backups. Since Child nodes are under-utilized during normal operations, the tool safely isolates, stops, and upgrades all 3 Replica nodes to the target Redis version first.
  - **Why:** This ensures the primary Parent nodes remain completely untouched, keeping 100% of the live cluster operations running without a single millisecond of slow-down for the end-user.

- **Phase B: The Failover Handshake (CLUSTER FAILOVER)**
  - **The Logic:** Once all 3 Child nodes are fully upgraded, the tool dispatches a strict automated `CLUSTER FAILOVER` command directly to each upgraded Child node.
  - **The Magic:** This command acts as an orchestration handshake. The upgraded Child node securely signals its respective Parent, telling it: *"I am now running the updated version; step down to a Replica role, and I will instantly take over as the active Master server."*

- **Phase C: Upgrading the Demoted Nodes & Maintaining Zero Downtime**
  - **The Logic:** Because this role-swapping happens instantaneously at the Redis cluster-routing layer, the user experiences absolutely zero latency or disruption. The old Parent nodes—now safely demoted to Child/Replica roles—are subsequently targeted, fetched dynamically via the automated pipeline, and upgraded to match the target version.

🚗 **The Stepney Analogy:**
Our deployment views Replica nodes exactly like a car's active backup Stepney. The moment a primary runtime component needs a structural transition (upgrade), the architecture gracefully swaps the load over to the ready backup structure (Stepney) on-the-fly and keeps moving forward seamlessly, proving a resilient, zero-downtime pipeline.

### Any Assumptions or Trade-offs You Made

- ⚖️ **Language Trade-off: Python vs. Java (Prioritizing Velocity & Core Expertise):**
  - **The Trade-off:** Initially, Python was considered for developing the CLI due to its native script-handling capabilities. However, since my core advanced expertise lies deeply within the Java ecosystem—having only basic academic exposure to Python during my second year—learning Python syntax from scratch would have significantly slowed down development velocity.
  - **The Decision:** I chose to prioritize execution speed and technical depth by sticking with a Java-Ansible architecture. This allowed me to immediately build the core logic, execute automated shell operations, and call the underlying Ansible playbooks directly from Java with 100% confidence.

- 🏗️ **Architectural Refactoring: From Monolithic Code to MVP Pattern:**
  - **The Process:** In the initial "Proof of Concept" phase, the Java automation logic and CLI methods were tightly coupled and unstructured. Once the functional workflow was working, I proactively initiated a heavy refactoring phase.
  - **The Structure:** To prevent a messy codebase, I adapted the architectural **MVP (Model-View-Presenter)** design pattern tailored for a Console/CLI Environment. This cleanly separated the data models, terminal views, and execution logic, turning the tool into a highly maintainable, production-ready enterprise application.

- 🎨 **UI/UX Enhancement (Terminal Aesthetics):**
  - **The Detail:** The initial version generated plain text terminal printouts. Recognizing that a premium CLI tool should mimic an authentic production console experience, I strategically refactored the presentation layer to support custom formatting and modern terminal status icons. This trade-off required extra string-parsing logic but dramatically elevated the tool’s professional look and user experience during live evaluation.       

### Known Limitations

- 🐧 **1. OS Compatibility Constraints (Linux/macOS vs. Windows Environment):**
  - **The Limitation:** The tool and its underlying automated Ansible orchestration engine are architected specifically to exploit native Unix shell behaviors. Therefore, the application executes flawlessly and is fully verified across Linux and macOS distributions. However, running the application natively within a pure Windows environment is not fully supported and may experience runtime issues unless executed via Linux-subsystem compatibility layers like WSL (Windows Subsystem for Linux).

- 🌐 **2. Mandatory Network/Internet Reliance for Dynamic Fetching:**
  - **The Limitation:** Because the tool avoids using bulky, pre-bundled binaries and instead dynamically fetches any official Redis version string on-the-fly, the host evaluation system strictly requires an active, stable internet connection during the upgrade window. If the host network drops, the automated compilation sequence will fail.

- 🔗 **3. Tight Coupling with Container Runtime Namespaces:**
  - **The Limitation:** The Java-Ansible orchestration layer relies heavily on a fixed and predictable container naming convention and IP layout inside the docker environment. Any external, manual modifications made to the container namespaces or network subnets during the live deployment window will interrupt the automation target paths.

- ⚠️  **4. Single-Node Availability Risk During the Sync Window:**
  - **The Limitation:** While a Master node is undergoing its rolling transition (immediately after being gracefully demoted to a Replica role), its newly promoted Master node temporarily operates without an active backup replica for a brief resynchronization window. A host-level hardware or container failure during this specific narrow window could risk temporary cluster degradation.
