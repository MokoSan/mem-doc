# Mem-Doc Lite

This document presents an investigation-first no-BS approach to detecting and solving performance problems related to the .NET Garbage Collector. For a more comprehensive guide, refer to the original .NET Memory Performance Analysis document.

## Work Flow

### 1. Know Your Goal: What are you optimizing for?

| Optimization Criteria | Details  | Example  | Metric To Care About   |
| --------------------- | -------- | -------- | ---------------------- |
| Throughput            | The GC is taking a lot of time by blocking your threads and consuming CPU and you want to reduce it. | You need to run as many requests as possible during a certain amount of time and the GC is impeding your ability to do so because of a high percentage time spent in the GC. | % Pause Time in GC |
| Memory Footprint | You want to keep the size of the GC heap low. | You need to run as many instances on the same machine as possible. | GC Heap Histogram: Looking at both the heap size at the start of the GC and also at the end of it. |
| Tail Latency | Individual GCs are causing high pause times and these affect the latency of your application. | You need to meet a certain SLA for latency | Individual GC Pauses that contribute to the specific latency percentile you care about. |

NOTE: You might have more than 1 goal but the aforementioned approach will help you focus on the most important goal in mind.


### 2. Take a GC Trace

The first step to any GC performance investigation is to take a top level GC trace.

| Platform  | Command   |  Additional Arguments  | 
| --------- | --------- | ---------------------- |
| Windows   | ``perfview /GCCollectOnly /AcceptEULA /nogui collect name.etl``     | ``/MaxCollectSec`` is the maximum time in seconds to collect the trace for.                   |
| Linux     | ``dotnet trace collect -p <pid> -o <output path with .nettrace extension> --profile gc-collect``     | ``--duration <in hh:mm:ss> format>``   |

### 3. Check For Definitive Signs of Performance Problems

1. Managed Memory Leak
2. Suspension is too long
3. Random Long Unproductive GC Pauses
4. Most GCs are full blocking GCs

5. Go Through the Workflow of Solving Your Problem

### Optimizing for Throughput

#### 1. Too Many Pauses i.e., Too Many GCs?

1. Profile Allocations

2. Infer reasons for why the GC decided to collect a Generation


#### 2. Individual Long GCs?

1. Is the long GCs due to GC work or not? 
  
- Check if you have a managed memory leak. 
- Are Full Blocking Gen2 GCs, the cause: 
  - Check for High Memory Load.
  - Check for Fragmentation.
- Long BGC pauses.
  - Due to GC bug.
- Are Ephemeral Generations really doing a lot of work? Check if it's productive.
- Not related to GC Work
  - Memory Leaks.
  - Long Suspensions.
  - GC work being blocked.

1. Check for a Managed Memory Leak


2. What type of GCs are contributing to your long pauses?

- Full Blocking Gen2s are a result of: high memory load, gen2 fragmentation.

### Optimizing for Memory Footprint

1. Peak Size is too large but the after size is not.

- Indicative of too much Gen0 allocation before the next GC is triggered. 
- Configure the Gen0MaxBudget with caveat: you'll see more 

2. Is the After Size Also Too Large? 

- Check if most of your size is in Gen2 / LOH. 
- Are you mostly doing BGCs?
  - Check fragmentation.
- After Full Blocking GCs, is your after Size still too large => too much data has survived.. Managed Memory Leak check.
- Check Fragmentation -> <15% or smaller.

- Memory Leak
- Fragmented Heap
  - Check Pinned Objects

Mostly BGCs for Gen2 GCs?
  - Check fragmentation

Still want it smaller despite all the above being fine:

Reduce size of heap by configurations.

### Optimizing for Tail Latency


Long Pauses only.

#### Questions

1. What version of the runtime?
2. What's the process name?
3. What is your goal / problem?