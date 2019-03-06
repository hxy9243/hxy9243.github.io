title: |
    Paper Reading: Large-scale cluster management at Google with Borg
date: 2019-02-27 11:05:34
comments: true
tags: ['Borg', 'PaperReading', 'DistributedSystems']
categories: Paper
---

Link: https://ai.google/research/pubs/pub43438

About: Borg is Google's large cluster workload scheduling and management system, which handles Google's most service and batch job workloads on a cluster on scale of thousands of machines. It hides users from burdens of management of cluster, and provides high-availability features that handles failures.

The now very famous and popular open-source docker orchestration tool Kubernetes, is an open source successor to Borg, and keeps borrowing ideas from Borg (see [kubernetes](https://kubernetes.io/blog/2015/04/borg-predecessor-to-kubernetes/)).

<!-- more -->

## Concepts

### Workloads

There are heterogeneous workloads on the cluster, that could mainly be categorized as

* long-running services: that responds to user requests.
* batch jobs: computation work that might take long time to finish.

### Cluster and cells

A cell is a collection of machines in a datacenter. A cluster hosts one large cell or several smaller cells for testing.

### Jobs and tasks

A job is made of one of multiple tasks. Tasks can:

* have constraints on what OS, what IP, processor it requires,
* run inside a container with resources (CPU, memory, disk) limits with command-line flags.

Users can operate by jobs with RPCs to Borg.

### Allocs

- Alloc: is a reserved set of resources on a machine for one or more tasks to be run.
- Alloc Set: a set of Allocs on multiple machines. Once an Alloc Set is created, a job can be scheduled to run on it.

### Priority, Quota and Admission Control

Every job has a priority, and the scheduler schedule them ranking by the priority.

Quota is assigned to/purchased by the user. It's defined by resources at a certain priority. Quota is managed by admission control, and a job/user is over quota, the job is immediately rejected.

### Naming and monitoring

Borg names and monitors tasks with:

- "Borg name service", that assigns each task a name and a DNS name, so that a task can be reachable at a certain DNS address.
- Chubby consistency service: a task writes its info to Chubby upon creation, and updates when there's a change in health.
- Almost every task has an HTTP endpoint that exposes health metrics that can be queried by Borg health monitoring service.
- Records all job submission and task events, resource usage metrics in a database for future query.

## Architecture

### Borgmaster and Borglet

Borg master records all the job status and manages state machines to all the objects in the system (machines, tasks, allocs, etc). And the data is saved in a Paxos-enabled Chubby store.

Borglet is a local Borg agent that resides on every machine in a cell, which manages tasks on a single machine, and sends heartbeats to the master.

### Scheduler

Borgmaster records jobs to Paxos store and pending queue, which is picked up by the scheduler, and gets scheduled. The scheduler uses an algorithm "E-PVM" for scoring, (sometimes called "worst fit"), or an algorithm that packs the tasks to minimal number of machines (sometimes called "best fit").

### Scalability

Borg uses the following techniques for scalability:

- Scheduler uses a separate process, to operate in parallel with the other Borgmaster.
- A scheduler operates on a cached copy of the cell state.
- Uses separate threads to talk to Borglets and respond to read-only RPCs.
- Shards (partitioned) functions across five Borgmaster replicas.


## Availability

Failures are normal and applications run on Borg on expected to handle failures, and automatically rescheduled when evicted due to failure, eviction, preemption, and etc.

## Conclusion

Borg serves as an important example for the design of all other large-scale distributed scheduling systems, which performs in the challenges of functionality, scalability and availability, and high utilization of the cluster resources.
