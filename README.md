# Fast DDS QoS and Security Experiment

A C++ project using **eProsima Fast DDS** to study DDS Quality of Service (QoS) configurations, message latency, and the performance impact of DDS Security.

## 1. Project Overview

This project implements a simple Fast DDS publisher/subscriber application using the `HelloWorld` data type.

The application can be used to:

- Run a Fast DDS publisher and subscriber.
- Measure end-to-end message latency.
- Observe message payload size.
- Measure received data throughput.
- Experiment with different DDS QoS policies.
- Compare performance with and without DDS Security.

The project currently uses:

- **Fast DDS:** 3.6.0
- **Fast DDS-Gen:** 4.3.0
- **C++:** C++11
- **CMake:** 3.20 or newer
- **Fast CDR**

---

## 2. Repository Structure

```text
FastDDS-Project/
│
├── CMakeLists.txt
├── HelloWorld.idl
│
├── HelloWorld.hpp
├── HelloWorldCdrAux.hpp
├── HelloWorldCdrAux.ipp
├── HelloWorldPubSubTypes.hpp
├── HelloWorldPubSubTypes.cxx
├── HelloWorldTypeObjectSupport.hpp
├── HelloWorldTypeObjectSupport.cxx
│
├── HelloWorldApplication.hpp
├── HelloWorldApplication.cxx
├── HelloWorldPublisherApp.hpp
├── HelloWorldPublisherApp.cxx
├── HelloWorldSubscriberApp.hpp
├── HelloWorldSubscriberApp.cxx
├── HelloWorldmain.cxx
│
├── security/
│   ├── ca.pem
│   ├── governance.xml
│   ├── permissions.xml
│   ├── pub.pem
│   ├── pub.csr
│   ├── sub.pem
│   └── sub.csr
│
└── .gitignore
```

Private key files (`*.key`) and the build directory are intentionally excluded from Git.

---

## 3. Requirements

### Operating System

The tested setup is:

- Ubuntu/Linux

Fast DDS is cross-platform, but the commands in this README are written for Linux.

### Software

Install or make sure the following are available:

```bash
g++ --version
cmake --version
```

Fast DDS and Fast CDR must be installed and discoverable by CMake.

Check Fast DDS:

```bash
fastdds --version
```

Example:

```text
Fast DDS version: 3.6.0.0
```

> **Note:** Fast DDS-Gen is not required just to build this repository because the generated C++ type-support files are already included. It is only needed if you want to regenerate the source files from `HelloWorld.idl`.

---

## 4. Clone the Repository

Clone the project:

```bash
git clone https://github.com/Luckygoyal765/FastDDS-Project.git
```

Enter the project directory:

```bash
cd FastDDS-Project
```

---

## 5. Build the Project

Create a build directory:

```bash
mkdir -p build
```

Configure the project:

```bash
cmake -S . -B build
```

Build:

```bash
cmake --build build -j$(nproc)
```

After a successful build, the executable will be:

```text
build/HelloWorld
```

---

## 6. Run the Publisher and Subscriber

The application requires one argument:

```text
publisher
```

or

```text
subscriber
```

Both processes use:

- **DDS Domain ID:** `0`
- **Topic:** `HelloWorldTopic`

### Terminal 1 — Start Subscriber

From the project directory:

```bash
./build/HelloWorld subscriber
```

You should see:

```text
subscriber running...
```

### Terminal 2 — Start Publisher

Open another terminal and run:

```bash
cd FastDDS-Project
./build/HelloWorld publisher
```

You should see:

```text
publisher running...
```

The subscriber should then begin receiving messages.

---

## 7. Understanding the Output

The subscriber prints latency measurements in the following format:

```text
<latency>,LATENCY,<message_size>
```

Example:

```text
131241,LATENCY,1000
```

Here:

- `131241` = measured latency in nanoseconds
- `LATENCY` = measurement label
- `1000` = message payload size in bytes

The publisher currently sends a message payload consisting of 1000 `A` characters.

Therefore:

```text
message_size = 1000 bytes
```

The subscriber also periodically prints:

```text
throughput,<value>
```

The throughput value is calculated as received bytes per second.

---

## 8. Current QoS Configuration

The current publisher and subscriber are configured with:

### Reliability

```cpp
RELIABLE_RELIABILITY_QOS
```

Reliable communication requests that samples are delivered reliably.

### Durability

```cpp
TRANSIENT_LOCAL_DURABILITY_QOS
```

The publisher retains samples according to the durability policy so that late-joining subscribers can receive retained data when applicable.

### History

```cpp
KEEP_ALL_HISTORY_QOS
```

The middleware attempts to keep all samples subject to the applicable resource limits.

The publisher configuration is conceptually:

```cpp
DataWriterQos writer_qos = DATAWRITER_QOS_DEFAULT;

writer_qos.reliability().kind =
    ReliabilityQosPolicyKind::RELIABLE_RELIABILITY_QOS;

writer_qos.durability().kind =
    DurabilityQosPolicyKind::TRANSIENT_LOCAL_DURABILITY_QOS;

writer_qos.history().kind =
    HistoryQosPolicyKind::KEEP_ALL_HISTORY_QOS;
```

The subscriber uses matching reliability, durability, and history settings.

---

## 9. Experimenting With QoS

The project can be modified to compare different QoS configurations.

Examples of policies that can be studied include:

- Reliable vs Best Effort reliability
- Volatile vs Transient Local durability
- Keep Last vs Keep All history
- Different history depths
- Different message publication rates
- Different payload sizes

When changing QoS, make sure the publisher and subscriber use compatible policies.

After modifying the source code, rebuild:

```bash
cmake --build build -j$(nproc)
```

Then run the subscriber and publisher again.

---

## 10. Latency Measurement

The subscriber calculates latency using the timestamp included in each message.

The basic calculation is:

```text
latency = current_time - publisher_timestamp
```

The timestamp is represented in nanoseconds.

The output therefore reports latency in **nanoseconds**.

For example:

```text
50000,LATENCY,1000
```

means an observed latency of approximately:

```text
50,000 ns = 50 µs
```

---

## 11. DDS Security

The repository contains DDS Security-related configuration and certificate files under:

```text
security/
```

These include:

- `governance.xml`
- `permissions.xml`
- CA certificate
- publisher certificate
- subscriber certificate
- certificate signing request files

The governance configuration covers Domain 0 and specifies protected DDS/RTPS communication and access-control behavior.

The permissions configuration contains permissions for the publisher and subscriber identities.

### Important Security Note

The presence of the XML and certificate files **does not by itself activate DDS Security**.

The current C++ source does not contain the complete DDS Security property configuration needed to automatically enable security.

Therefore, if you clone this repository, do not assume that simply running:

```bash
./build/HelloWorld subscriber
```

and:

```bash
./build/HelloWorld publisher
```

will run the application with DDS Security enabled.

To reproduce a secured experiment, the security configuration must be explicitly connected to the Fast DDS participant configuration and the corresponding private keys must be available on the machine.

For a new experiment, it is recommended to generate and use your own certificates and private keys rather than sharing private keys through Git.

---

## 12. Security Files

The repository intentionally excludes private key files through `.gitignore`:

```gitignore
build/
*.key
```

This is important because private keys should not be committed to a public Git repository.

If you configure DDS Security on another system, place the required private keys in the appropriate local security directory and configure Fast DDS to use them.

---

## 13. Running on Another Computer

To reproduce the basic publisher/subscriber experiment on another Linux machine:

### Step 1 — Install dependencies

Install:

- C++ compiler
- CMake
- Fast DDS
- Fast CDR

Verify:

```bash
g++ --version
cmake --version
fastdds --version
```

### Step 2 — Clone the repository

```bash
git clone https://github.com/Luckygoyal765/FastDDS-Project.git
cd FastDDS-Project
```

### Step 3 — Build

```bash
mkdir -p build
cmake -S . -B build
cmake --build build -j$(nproc)
```

### Step 4 — Start subscriber

Terminal 1:

```bash
./build/HelloWorld subscriber
```

### Step 5 — Start publisher

Terminal 2:

```bash
./build/HelloWorld publisher
```

The two applications can communicate when they are configured for the same DDS domain and can discover each other on the network.

---

## 14. Running Publisher and Subscriber on Different Machines

For a distributed experiment:

1. Install Fast DDS on both machines.
2. Clone the repository on both machines.
3. Build the project on both machines.
4. Use the same DDS Domain ID.
5. Make sure the network allows DDS discovery and data traffic.
6. Start the subscriber on one machine.
7. Start the publisher on the other machine.

Example:

### Machine A

```bash
./build/HelloWorld subscriber
```

### Machine B

```bash
./build/HelloWorld publisher
```

Network/firewall configuration may be required depending on the environment.

---

## 15. Rebuilding From a Clean State

If you want to remove the generated build files and rebuild:

```bash
rm -rf build
mkdir build
cmake -S . -B build
cmake --build build -j$(nproc)
```

**Do not run this command if you need to preserve the existing build directory.**

---

## 16. Regenerating Fast DDS Type-Support Code

`HelloWorld.idl` defines the data type:

```idl
struct HelloWorld
{
    long index;
    string message;
    long long timestamp;
};
```

The generated files include:

```text
HelloWorld.hpp
HelloWorldPubSubTypes.hpp
HelloWorldPubSubTypes.cxx
HelloWorldCdrAux.hpp
HelloWorldCdrAux.ipp
HelloWorldTypeObjectSupport.hpp
HelloWorldTypeObjectSupport.cxx
```

If Fast DDS-Gen is installed, the IDL can be used to regenerate the corresponding type-support code.

However, regeneration is not required for a normal clone/build of this repository because the generated files are already included.

---

## 17. Troubleshooting

### `./HelloWorld: No such file or directory`

Make sure you are running the executable from the correct location:

```bash
./build/HelloWorld subscriber
```

or:

```bash
cd build
./HelloWorld subscriber
```

### CMake cannot find Fast DDS

Check:

```bash
fastdds --version
```

Also verify that Fast DDS and Fast CDR are installed and available to CMake.

If they were installed in a custom location, CMake may need the appropriate installation prefix/path.

### Publisher and subscriber do not communicate

Check:

- Both processes use Domain ID `0`.
- Both use the topic `HelloWorldTopic`.
- Both are running at the same time.
- The machines are on a network that allows DDS discovery/data traffic.
- Firewall rules are not blocking the required DDS traffic.
- QoS policies are compatible.

### Permission/security errors

If experimenting with DDS Security, verify:

- Certificates exist.
- Private keys exist locally.
- File paths are correct.
- Governance and permissions XML files are valid.
- Certificate identities match the permissions configuration.
- The security properties have actually been applied to the Fast DDS participant.

---

## 18. Research Measurements

For meaningful performance comparisons, do not rely on a single latency sample.

A stronger experiment should collect multiple runs and report statistics such as:

- Mean latency
- Median / p50
- p95
- p99
- Minimum
- Maximum
- Standard deviation
- Jitter
- Throughput

Experiments can also vary:

- QoS configuration
- Payload size
- Publication rate
- Security enabled/disabled
- Network conditions
- CPU load

For comparisons, keep the experimental conditions identical except for the variable being studied.

---

## 19. Example Research Comparison

A typical experiment can compare:

```text
Configuration A: Reliable
Configuration B: Best Effort
Configuration C: Balanced/custom QoS
Configuration D: Security disabled
Configuration E: Security enabled
```

For each configuration, collect multiple samples under the same workload and calculate the required statistics.

When reporting security overhead, use matched experimental conditions and matched sample counts wherever possible.

---

## 20. GitHub

Repository:

urlFastDDS-Project on GitHubhttps://github.com/Luckygoyal765/FastDDS-Project

---

## 21. References

- eProsima Fast DDS documentation:
  https://fast-dds.docs.eprosima.com/

- Fast DDS installation documentation:
  https://fast-dds.docs.eprosima.com/en/3.x/installation/sources/sources_linux.html

- Fast DDS simple C++ application:
  https://fast-dds.docs.eprosima.com/en/3.x/fastdds/getting_started/simple_app/simple_app.html

- Fast DDS-Gen documentation:
  https://fast-dds.docs.eprosima.com/en/latest/fastddsgen/usage/usage.html

- OMG DDS Security specification:
  https://www.omg.org/spec/DDS-SECURITY/1.1

---

## 22. Quick Start

For someone who only wants to run the project:

```bash
git clone https://github.com/Luckygoyal765/FastDDS-Project.git
cd FastDDS-Project
mkdir -p build
cmake -S . -B build
cmake --build build -j$(nproc)
```

Then open two terminals.

**Terminal 1:**

```bash
./build/HelloWorld subscriber
```

**Terminal 2:**

```bash
./build/HelloWorld publisher
```

The subscriber will print latency measurements and periodic throughput measurements.

---

## License

Add the appropriate license for your project before publishing or distributing the repository.
