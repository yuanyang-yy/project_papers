
## Project Instruction Updates:

1. Complete the function MemoryScheduler() in memory_coordinator.c
2. If you are adding extra files, make sure to modify Makefile accordingly.
3. Compile the code using the command `make all`
4. You can run the code by `./memory_coordinator <interval>`

### Notes:

1. Make sure that the VMs and the host has sufficient memory after any release of memory.
2. Memory should be released gradually. For example, if VM has 300 MB of memory, do not release 200 MB of memory at one-go.
3. Domain should not release memory if it has less than or equal to 100MB of unused memory.
4. Host should not release memory if it has less than or equal to 200MB of unused memory.
5. While submitting, write your algorithm and logic in this Readme.

## Code Description and Algorithm (yyang998)
Assume each domain is configured with one vCPU. 
Interval used in this project is 2.

#### Data Structures
- `struct VCpuMemInfo`
    Stores memory related information of a vCPU and contains five fields: 
    - `domainNum`: domain that this vCPU is on;
    - `actualMem`: actual memory;
    - `unusedMem`: unused memory;
    - `availableMem`: available memory;
    - `usableMem`: usable memory.
- Two arrays: Length of each array = number of vCPUs = number of domains(VMs).
    - `VCpuMemInfo* vCpuMemInfoArr`: Stores memory related data of each vCPU in current iteration;
    - `VCpuMemInfo* prevVCpuMemInfoArr`: Stores memory related data of each vCPU in previous iteration.

#### Metrics and Thresholds used for Memory Allocation
Metrics:
1. Changes in consumed memory between iterations for each vCPU:
    - **Consumed Memory = Actual Memory - Unused Memory**
    - **Delta = Consumed Memory at Current Iteration - Consumed Memory at Previous Iteration**
2. Value of unused memory at current iteration for each vCPU;
3. Value of host free memory at current iteration for each vCPU.

Thresholds:
1. `threshold` = 5MB: only release/allocate memory from/to a vCPU when absolute value of `Metric 1` > `threshold`;
2. `LOWER_BOUND_DOMAIN` = 100MB: for a domain, if its unused memory < 100MB, then it should not release memory at all;
3. `LOWER_BOUND_HOST` = 200MB: if host's unused memory < 200MB, then any domain should not acquire extra memory (host should not release memory);
4. `MAX_TOTAL_MEM` = 2048MB: if a domain need allocate extra memory, then the actual memory after allocating should not exceed 2048MB.
5. There are some other thresholds used in this project, please refer to Allocating Algorithm section for details.

#### Code Description: `void MemoryScheduler(virConnectPtr conn, int interval)` 
During each iteration:
1. Get all active and running VMs(domains) using `virConnectListAllDomains()`;
2. Set memory stats collection period for each domain using `virDomainSetMemoryStatsPeriod()`: 
    - value of `period` set to 1 instead of interval value of 2;
    - reason: when `period = 2`, I noticed lagging in data collection, causing unused memory to be larger than actual memory at some points. Changing `period` value from 2 to 1 solved this issue.
3. Iterate through all domains. For each domain:
    3.1. Get memory info of the vCPU on this domain using `virDomainMemoryStats()`.
    3.2. Iterate through obtained statistics and get actual, unused, available, and usable memory data for current domain and store in `vCpuMemInfoArr`;
4. Get host's free memory value using `virNodeGetMemoryStats()`;
4. Perform memory allocation and de-allocation for each domain. For each domain:
    => refer to the Allocating Algorithm section below for details of this algorithm.
5. Copy current vCPU memory info array `vCpuMemInfoArr` to `prevVCpuMemInfoArr`;
6. Free all the allocated space in code except for `prevVCpuMemInfoArr` (need this for calculating changes in consumed memory in next iteration).

#### Allocating Algorithm: 
For each domain: 
1. Calculate the consumed memory for current and previous iterations (`curConsumedMem` and `preConsumedMem`) based on data in `vCpuMemInfoArr` and `prevVCpuMemInfoArr`: actual - unused;
2. Calculate `delta`: absolute value of the consumed memory changes = `abs(curConsumedMem - preConsumedMem)`;
2. Three different cases:
    - Case 0 => If this vCPU consumed more memory than previous iteration AND the change `delta` is larger than `threshold`, this means the vCPU becomes busier and more memory should be allocated to it. 
        - Only allocate memory when host has free memory larger than `LOWER_BOUND_HOST`;
        - Use `virDomainSetMemory()` to allocate additional `delta * 1.5` amount of memory to this domain. (Memory allocation should be faster, so using 1.5)
    - Case 1 => If this vCPU consumed less memory than previous iteration AND the change `delta` is larger than `threshold`, this means the vCPU becomes freer and memory should be released from it. 
        - Only release memory when this domain has unused memory larger than `LOWER_BOUND_DOMAIN`;
        - Use `virDomainSetMemory()` to release `delta * 0.05` amount of memory from this domain. (Memory de-allocation should be gradually and slower, so using 0.05)
    - Case 2 => If this vCPU consumed similar memory comparing to previous iteration, memory should be released from this vCPU if its unused memory > 350MB (to prevent wasting).   
        - If its unused memory > 350MB, use `virDomainSetMemory()` to release 100MB memory from this domain.

**The basic concept of this algorithm: use the change in consumed memory to decide if allocating more memory OR release memory OR do nothing. Then allocate/de-allocate amount of memory that proportional to the change in consumed memory. Also utilize unused memory value to determine if memory should be released when the change in consumed memory is not obvious. Moreover, multiple thresholds are used to help making decisions.** 
