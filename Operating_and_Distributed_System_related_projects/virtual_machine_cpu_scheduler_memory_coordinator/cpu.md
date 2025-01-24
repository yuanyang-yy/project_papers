## Project Instruction Updates:

1. Complete the function CPUScheduler() in vcpu_scheduler.c
2. If you are adding extra files, make sure to modify Makefile accordingly.
3. Compile the code using the command `make all`
4. You can run the code by `./vcpu_scheduler <interval>`
5. While submitting, write your algorithm and logic in this Readme.

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





    
