# MapReduce Implementation Readme 
Yuan Yang (yyang998)

## Table of Contents
1. [Entire Work Flow](#entire-work-flow)
    - [Map Reduce Specifications](#map-reduce-specifications)
    - [File Sharding and Partitioning](#file-sharding-and-partitioning)
    - [grpc protocol specification](#grpc-protocol-specification)
    - [MASTER](#master)
    - [WORKER](#worker)
2. [Worker Failure Case and Slower Worker Case](#worker-failure-case-and-slower-worker-case)

### Entire Work Flow
##### Map Reduce Specifications
`mapreduce_spec.h`

- Some important specs retrieved from config file:
    - number of workers; 
    - ip addresses for the list of the workers;
    - list of the input files;
    - directory to store the output files;
    - number of output files;
    - max size of the each shard we feed into map task (input files => partitions each with size of map_kilobytes);
- Validate the specs:
    - number of workers matches size of worker address list;
    - check if all the input files exist;
    - number of workers, number of output files, and max size of the each partition should all be larger than 0;
    - output directory should exist: if not exist, create it; if exist, remove all the files in it;

##### File Sharding and Partitioning
 `file_shard.h`

- The sharding logic follows the same logic introduced in project instruction. 
- Each shard contains a list of file partitions: each file partition include {filename, start line, end line} => the files are split by "\n" to prevent splitting in the middle of a word;
    - For each line in a file, 
    - if it can be added into current shard and not exceed capacity, then add this line into the shard;
    - if adding this line would result in exceeding shard capacity, then add this shard to the vector of shards, create new shard, add this line into the new shard;
- Each shard should not exceed max size of the each shard defined in specs.
- Local test on sharding logic: It was found that smaller shard size (<= 5KB) would results in more issues. => (these are some edge cases to use for debugging)

##### grpc protocol specification
`masterworker.proto`  between master and worker
- Master is client, Workers are servers:
    - Master sends map/reduce requests to workers;
    - Workers process the request, and send map/reduce responses back to master;
- two rpc calls: 
    - map: `MapResult PerformMapTask(MapInput) `
    - reduce: `ReduceResult PerformReduceTask(ReduceInput)`
- Input for mapping is a list of shards and inputs for reducing is an intermediate file (output from map task). There are also some output-related info included.
- Reply from mapping is a list of intermediate file paths and Reply from reducing is an output file. There are also some worker-related info included.

##### **MASTER**:
`master.h`
- Implemented using grpc async client call => similar to project3
- Multiple channels and stubs are created for master to communicate with multiple workers: number of channels/stubs == number of workers => each worker corresponds to a stub;
- A queue is maintained to store workers that are available for new tasks, and the queue is protected by a mutex and condition variable.
- **shards ===Map===> intermediateFiles ===Reduce===> outputFiles**
    - number of intermediateFiles == number of outputFiles;
- Map tasks: 
    - master assigns each file shard to available worker; 
    - master receives outputs from mapping as intermediate files => store as "./intermediate/intermediate_%d": "intermediate_0", "intermediate_1",..., "intermediate_$numOfOutputFiles"
- After all the map tasks are completed, perform reduce tasks. (two booleans are used to track if all the map/reduce tasks are completed or not)
- Reduce tasks: 
    - master assigns each intermediate file to available worker (intermediate files are deleted after completion of the task)
    - master receives outputs from reducing as output files => store as "./output/output_%d": "output_0", "output_1", ..., "output_$numOfOutputFiles"
- After the completion a task, the worker would be re-added into the queue that stores the available worker.
- **Slower Worker case and Worker Failure case** are both handled by master. The details of these would be introduced at the end of this README.

##### **WORKER**
`worker.h`
- Implemented using grpc sync server service;
- Some of the essential logics related to workers are implementated in `mr_tasks.h`(Mapper/Reducer Internal implementation);
- How worker handles Map task:
    - mapper was used to perform map task to each line in the input file shards;
    - for each emit of <key, value>, use hashing function to hash the key, then take the reminder of  hashed_results/num_of_output_files to determine which intermediate file should we use to store this <key, value> pair. 
        - e.g., num_of_output_files = 8, let's say hash(key) % 8 = 3, then this pair should be written into file `./intermediate/intermediate_3`.
- How worker handles Reduce task:
    - mapper was used to perform reduce task to each <key, vector<value>> in an intermediate file;
    - for current intermediate file, pre-process the file to merge all the values coupled with the same key. (an intermediate file => an map<<key, vector<value>>>);
    - the pairs are automatically sorted by key in the map.
    - then iterate <key, vector<value>> pairs:
        - use reducer to perform reduce task on each pair;
        - for each emit, stores the reducing output (<key, value_after_reducing>) in output file.
        - if the input file is `./intermediate/intermediate_3`, then output file is named as the same number: `./output/output_3`. 
    - the pairs in output file should already be sorted alphabetically.

### Worker Failure Case and Slower Worker Case
Both of the cases are handled by master.
- Worker failure would result in failed RPC call. When the rpc status returned to be "not ok", this means the map/reduce task haven't be completed. 
    - handling method: master keeps spinning on the status of current call, 
        - if rpc is unsuccessful, grab a new worker from available_workers_queue 
        - add previous failed worker back to the queue (in case the failure is temporary => still can be used later) 
        - re-send the map/reduce request to new worker (if new worker also fails, we repeat this process)
- Slower Worker would be time-consuming. When a worker is slow and there are free workers, we can kill the slow worker and re-assign the task to a new worker.
    - handling method: set a deadline to client context and I used 5 seconds for the deadline. If master does not receive the reply from the worker after 5 seconds, the rpc status would return to be "StatusCode::DEADLINE_EXCEEDED". => then a new worker would be assigned to take over this task.
    - I used the same strategy as Worker Failure case to handle this DEADLINE_EXCEEDED exception.


