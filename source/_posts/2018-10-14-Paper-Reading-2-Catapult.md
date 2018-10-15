---
title: 'Paper Reading: A Reconfigurable Fabric for Accelerating Large-Scale Datacenter Services'
date: 2018-10-14 21:47:00
comments: true
tags: ["FPGA", "Datacenter", "HPC"]
categories: Paper
---

https://www.microsoft.com/en-us/research/publication/a-reconfigurable-fabric-for-accelerating-large-scale-datacenter-services/

This is one of the series of papers from Microsoft's [Project Catapult](https://www.microsoft.com/en-us/research/project/project-catapult/),
which studies leveraging reconfigurable devices (FPGA, etc.) to accelerate data center, from very specific
accelerating algorithms like page ranking for Bing search engine, to more sophisticated machine
learning frameworks like DNN.

This is one of their early publications, which introduces the basic design and implementation
of the FPGA accelerated datacenter. It covers the very fundamental details of all aspects of
server design, from hardware, network topology, FPGA core design, fault-tolerant cluster management
software design, workload scheduling algorithm, and etc..

<!-- more -->

Some highlights and takeaways:

## Hardware

Catapult hardware is integrated with existing server-grade blades, which takes the space on the
PCIe of the motherboard, through a daughter board with one single high-end FPGA card.

## Network Design

The daughter-board
cards connects each other with a fast secondary network, independent of the CPU network. The secondary
networks form a 6x8 torus topology network (see more details in paper), which gives fast inter-FPGA communications,
good routability but not too much cabling complexity. The CPU network connects to a 48-port switch for
each pod.

## Software Interface

On FPGA space has been divided into _Shell_ and _Role_. _Shell_ manages the common libraries or functionalities
like memory management, serial link, or PCIe, reconfiguration logics, etc.. The _Role_ space is responsible for
actual acceleration algorithm, which will be reloaded for each reconfiguration when the FPGA functionality needs
to be updated.

FPGAs will be reconfigured from time to time and certain software must be designed to ensure to take the FPGA
completely offline and ignored by neighbors, to ensure correct operations.

Debugging might be hard to achieve through typical JTAG hardware debugging facilities, considering the scale
of the datacenter. The paper presents an 'always-on' data collector that captures the key components and
saves them to a circular-buffer log.

The software interfaces divides the algorithm into 7-stages and distributed them to a 8-node FPGA group, with
one for redundancy. The paper describes how the network accelerates the 'Feature Extraction', which produces
a single score at the last stage, indicating how close the document is to the search key word.

All the queries are queued in memory in DRAM. The Queue Manager takes documents from each queue, then sends them
down the pipeline. It also manages the model reloads in the pipeline, which calculates different feature 'scores' for
queries.

## Evaluation

The Catapult project, according to the paper, 'reduces the worst-case latency by 29% in the 95 percentile distribution'
in their evaluation environment, and provides 95% gain in throughput relative to software.
