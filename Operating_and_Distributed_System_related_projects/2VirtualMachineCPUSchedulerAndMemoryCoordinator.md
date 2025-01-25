# Virtual Machine vCPU Scheduler and Memory Coordinator

# 1. CPU Scheduler

Run the code by `./vcpu_scheduler <interval>`

## Code Description and Algorithm (yyang998)
Assume each domain is configured with one vCPU. 
Interval used in this project is 2.

#### Data Structures and CPU Usage Calculation Method
Data used for calculating CPU usages:
- Two arrays: Length of each array = number of vCPUs = number of domains(VMs).
    - `unsigned long long *preVCpuTime`: Stores CPU time of each vCPU in previous iteration. 
    - `unsigned long long *curVCpuTime`: Stores CPU time of each vCPU in current iteration. 

CPU usage calculation: (Starts calculating usages at second iteration)
- vCPU usage% = (curVCpuTime - preVCpuTime) / interval * 100.0
- pCPU usage% = summation of usages of vCPUs pinned to this pCPU.

Data used for CPU scheduling:

- `struct VCPUInfo`
    Stores information of a vCPU and contains three fields: 
    - `double usage`: usage% of this vCPU;
    - `virDomainPtr domainNum`: domain that this vCPU is on;
    - `int pCpuNum`: pCPU# that this vCPU is pinned to.
- `struct PCPUInfo` 
    Stores information of a pCPU and contains one field: 
    - `double usage`: usage% of this pCPU.
- Two arrays: 
    - `VCPUInfo *vCPUInfoArr`: Stores `VCPUInfo` for each vCPU (length = number of domains);
    - `PCPUInfo *pCPUInfoArr`: Stores `PCPUInfo` for each pCPU (length = number of pCPUs).


#### Code Description: `void CPUScheduler(virConnectPtr conn, int interval)` 
During each iteration:
1. Get all active and running VMs(domains) using `virConnectListAllDomains()`;
2. Get total number of pCPUs using `virNodeGetInfo()`;
3. Iterate through all domains. For each domain:
    3.1. Get info (`current CPU time` and `pCPU#` this vCPU is affiliated to) of the vCPU on this domain using `virDomainGetVcpus()`.
    3.2. Store `current CPU time` in `curVCpuTime` array and `pCPU#` in `vCPUInfoArr`;
    3.3. Calculate current vCPU's usage; (formula listed in section above)
    3.4. Add current vCPU's usage to the usage of its affiliated pCPU; (formula listed in section above)
4. Perform scheduling (pin each vCPU to selected pCPU) when standard deviation of pCPU usages > 5.0 
    => refer to the Scheduling Algorithm section below for details of this algorithm
5. Copy current vCPU time array `curVCpuTime` to `preVCpuTime`;
6. Free all the allocated space except for `preVCpuTime` (need this for calculating usages in next iteration).

#### Scheduling Algorithm: (pin each vCPU to selected pCPU)
1. Calculate the standard deviation across the usages of all pCPU;
2. Sort the vCPUs in `vCPUInfoArr` in ascending order by each vCPU's `usage` using `qsort()`;
2. Assume each vCPU can only be affiliated to one pCPU;
3. If standard deviation of pCPU usages > 5.0, for each vCPU `i` in ascending order of usage:
    3.1. Calculate the no. of pCPU (`newPCPU`) that vCPU `i` should be pinned to: 
    - If `i / total_number_of_pCP` is an even number, `newPCPU` should be **`remainder of i / total_number_of_pCPUs`**;
    - If `i / total_number_of_pCP` is an odd number, `newPCPU` should be **`total_number_of_pCPUs - 1 - remainder of i / total_number_of_pCPUs`**;

    3.2. Re-pin current vCPU `i` to `newPCPU` using `virDomainPinVcpu()`;
    3.3. Update current vCPU's affiliated pCPU to be `newPCPU`;

**The basic concept of algorithm: when re-pinning is needed, sort the existing vCPUs by their usages, and re-pin all the vCPUs to available pCPUs according the Scheduling Algorithm defined in this project. The mechanism used is further elaborate using examples below:** 
- **Example 1: # of vCPU < # of pCPU** 
```
3 vCPU and 4 pCPU

After sorting vCPUs:
  i |    vCPU   |  usage_val  | should pin to which pCPU no.
- 0 | 'aos_vm3' | usage: 30.0 | => pin to pCPU no.0
- 1 | 'aos_vm1' | usage: 40.0 | => pin to pCPU no.1
- 2 | 'aos_vm2' | usage: 50.0 | => pin to pCPU no.2

Then after re-pinning:
 pCPU no. |   pCPU   |    mapping info     |     usage_val
-    0    | 'pCPU_1' | mapping ['aos_vm3'] | Total Usage = 30.0
-    1    | 'pCPU_2' | mapping ['aos_vm1'] | Total Usage = 40.0
-    2    | 'pCPU_3' | mapping ['aos_vm2'] | Total Usage = 50.0
-    3    | 'pCPU_4' | mapping []          | Total Usage = 0.0
```

- **Example 2: # of vCPU = # of pCPU** 
```
4 vCPU and 4 pCPU

After sorting vCPUs:
  i |    vCPU   |  usage_val  | should pin to which pCPU no.
- 0 | 'aos_vm3' | usage: 10.0 | => pin to pCPU no.0
- 1 | 'aos_vm4' | usage: 20.0 | => pin to pCPU no.1
- 2 | 'aos_vm2' | usage: 30.0 | => pin to pCPU no.2
- 3 | 'aos_vm1' | usage: 40.0 | => pin to pCPU no.3

Then after re-pinning:
 pCPU no. |   pCPU   |    mapping info     |     usage_val
-    0    | 'pCPU_1' | mapping ['aos_vm3'] | Total Usage = 10.0
-    1    | 'pCPU_2' | mapping ['aos_vm4'] | Total Usage = 20.0
-    2    | 'pCPU_3' | mapping ['aos_vm2'] | Total Usage = 30.0
-    3    | 'pCPU_4' | mapping ['aos_vm1'] | Total Usage = 40.0
```

- **Example 3: # of vCPU > # of pCPU**
    - **In this example, after re-pinning, the usages across 4 pCPUs are the same.**
    - As shown in this example, after sorting, the vCPUs numbered from 0 to 3 are pinned to pCPUs in forward order from pCPU no. 0 to 3, while the vCPUs numbered from 4 to 7 are pinned to pCPUs in backward order from pCPU no. 3 to 0. In this way, the usages across pCPU can be balanced as much as possible.
    - If there are more vCPUs, for example 12 vCPUs, then vCPUs numbered from 8 to 11 should be pinned to pCPUs in forward order from pCPU no. 0 to 3 again. Moreover, if there are 16 vCPUs, then vCPUs numbered from 11 to 15 should be pinned to pCPUs in backward order from pCPU no. 3 to 0. So on so forth.

```
8 vCPU and 4 pCPU

After sorting vCPUs:
  i |    vCPU   |  usage_val  | should pin to which pCPU no.
- 0 | 'aos_vm3' | usage: 10.0 | => pin to pCPU no.0
- 1 | 'aos_vm4' | usage: 20.0 | => pin to pCPU no.1
- 2 | 'aos_vm2' | usage: 30.0 | => pin to pCPU no.2
- 3 | 'aos_vm7' | usage: 40.0 | => pin to pCPU no.3
-------------------------------------------------------
- 4 | 'aos_vm5' | usage: 50.0 | => pin to pCPU no.3
- 5 | 'aos_vm1' | usage: 60.0 | => pin to pCPU no.2
- 6 | 'aos_vm8' | usage: 70.0 | => pin to pCPU no.1
- 7 | 'aos_vm6' | usage: 80.0 | => pin to pCPU no.0 

Then after re-pinning:
 pCPU no. |   pCPU   |         mapping info          |            usage_val
-    0    | 'pCPU_1' | mapping ['aos_vm3','aos_vm6'] | Total Usage = 10.0 + 80.0 = 90.0
-    1    | 'pCPU_2' | mapping ['aos_vm4','aos_vm8'] | Total Usage = 20.0 + 70.0 = 90.0
-    2    | 'pCPU_3' | mapping ['aos_vm2','aos_vm1'] | Total Usage = 30.0 + 60.0 = 90.0
-    3    | 'pCPU_4' | mapping ['aos_vm7','aos_vm5'] | Total Usage = 40.0 + 50.0 = 90.0
```

# 2. Memory Coordinator

Run the code by `./memory_coordinator <interval>`

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



    
