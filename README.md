# Project Papers/Reports
**Author: Yuan Yang**
Linkedin: https://www.linkedin.com/in/yuan-yang-yyang/

FYI: Due to my university's rule prohibiting us from sharing implementation details of the projects, this public repo only contains the reports and papers associated with my projects. If details are further needed, please email me at yuanyang.uu@gmail.com.

Table of Contents:
- [Machine Learning related projects](#machine-learning-related-projects) (6 projects)
- [Operating and Distributed Systems related projects](#operating-and-distributed-system-related-projects) (7 projects)
- [Software Development related projects](#software-development-related-projects) (2 projects)

---

### Machine Learning related projects: 
0. [Hateful Meme Detection (Python):](Machine_Learning_related_projects/0MultimodalLearningwithTransformers_HatefulMemeDetection.pdf)
    - Deep Learning and Natural Language Processing;
    - Multi-modal Framework Development with State-of-Arts **Visual-Linguistic Transformers**.
1. [Overcooked (Python):](Machine_Learning_related_projects/1CooperativeMultiAgentDeepReinforcementLearning_Overcooked.pdf)
    - Cooperative **Multi-Agent Deep Reinforcement Learning** in a multi-player game environment(Overcooked-AI);
    - Q-Learning with Neural Network: Deep Q-Learning (DQN) and Double Deep Q-Learning (DDQN).
2. [Optimal Rocket Landing (Python):](Machine_Learning_related_projects/2QLearningwithDeepNeuralNetwork_OptimalRocketLanding.pdf)
    - Reinforcement **Q-learning with a Neural Network**;
    - In OpenAI gym's Lunar Lander environment.
3. [Replication of Sutton’s Temporal Difference Learning Paper (Python):](Machine_Learning_related_projects/3ReinforcementLearning_ReplicationofSuttonTemporalDifferenceLearningPaper.pdf)
    - **Reinforcement Learning**;
    - Implemented the temporal difference(TD) methods, and compared with supervised-learning methods in a dynamical system (bounded random walk system).
4. [Stereo Correspondence (Python):](Machine_Learning_related_projects/4ComputerVision_StereoCorrespondence.pdf)
    - **Computer Vision**;
    - Addressed stereo correspondence problem using Window-based Sum of Squared Difference (SSD) method and Energy Minimization Loopy Belief Propagation (LBP) method.
5. [Key-Value Memory Network for Question-Answering System (Python):](Machine_Learning_related_projects/5NLP_project_key_value_memory_network.ipynb)
    - **Natural Language Processing**;
    - Implemented Key-Value Memory Network for Question-Answering System with multiple datasets preprocessing mechanisms.

### Operating and Distributed System related projects:
0. [MapReduce Library Implementation (C++):](Operating_and_Distributed_System_related_projects/0MapReduceLibraryImplementation.md)
    - **Big Data Framework Implementation**：processing and generating large-scale data sets；
    - Enable seamless parallelization of user-defined programs (Mapper&Reducer) across distributed clusters;
    - Fault tolerance and straggler mitigation.
1. [Distributed File System (DFS) with Remote Procedure Call (C++):](Operating_and_Distributed_System_related_projects/1DistributedFileSystemDevelopmentWithRPC.pdf)
    - **Distributed File System (DFS)**;
    - File transfer protocols (e.g., fetch, upload, delete, list status) using gRPC and Protocol Buffers;
    - Automatically detect and sync file modifications.
2. [Virtual Machine vCPU Scheduler and Memory Coordinator (C):](Operating_and_Distributed_System_related_projects/2VirtualMachineCPUSchedulerAndMemoryCoordinator.md)
    - A vCPU scheduler and a memory coordinator to dynamically allocate resources across multiple virtual machines on host physical machines;
    - Evaluated the performance on the cloud (Azure) under various system configurations. 
3. [Distributed Price Querying Service for an E-Commerce Store (C++):](Operating_and_Distributed_System_related_projects/3DistributedPriceQueryingServiceForAnECommerceStore.txt)
    - Distributed service for communication between clients and vendors in an e-commerce store;
    - Built a robust ThreadPool library to efficiently manage and process concurrent client requests.
4. [Barrier Synchronization Algorithms for Parallel Systems (C):](Operating_and_Distributed_System_related_projects/4BarrierSynchronizationAlgorithmsForParallelSystems.pdf)
    - Implemented sense-reversing, dissemination, tournament, and tree-based (with 4-Ary arrival) barriers using OpenMP and MPI parallelization frameworks, also an OpenMP-MPI combined barrier;
    - Evaluated barriers on multicore, distributed, and multicore + distributed systems, achieving performance comparable to built-in OpenMP and MPI barriers on a CC-NUMA cluster.
5. [Multi-threaded Online File Request System (C):](Operating_and_Distributed_System_related_projects/5MultiThreadedOnlineFileRequestSystem.pdf)
    - Created a multi-threaded User-Proxy-Cache-Web system for efficient file retrieval from internet servers;
    - IPC mechanisms with POSIX APIs.
6. [Implementation of a GetFile Server (C):](Operating_and_Distributed_System_related_projects/6ImplementationOfGetFileServer.pdf)
    - GetFile protocol and Multi-thread GetFile Server.

### Software Development related projects:
0. [Hemkraft (React, Spring, RESTful APIs with MySQL):](Software_Development_related_projects/0FullStackSoftwareDevelopment_Hemkraft) 
    - **Full-Stack Web Software Development**;
    - Collect users' household information and allow users to request reports of analyzed data (e.g., averaged household statistics within a specified postal code radius);
    - **Emphasized on the database design**: EER, IFD, EER to Relational Mapping
    - **Designed MySQL schemas** to automate data analysis within database via queries triggered by user requests, thereby enhancing efficiency.
1. [Android Job Comparison App (Android Studio with Java & SQLite, JUnit5 and Espresso):](Software_Development_related_projects/1AndroidAppDevelopment_JobComparison)
    - **Full-Stack Android App Development**;
    - Designed and implemented white box and black box test suites.



