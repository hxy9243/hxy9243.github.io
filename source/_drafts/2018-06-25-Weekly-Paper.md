title: Weekly Paper Reading 06-25
date: 2018-06-25 00:45:00
comments: true
tags: ['Paper', 'HPC', 'FPGA']
categories: Reading
---

Link to paper: https://www.microsoft.com/en-us/research/wp-content/uploads/2016/10/Cloud-Scale-Acceleration-Architecture.pdf

From Microsoft research, this paper presents the ['Catapult'](https://www.microsoft.com/en-us/research/project/project-catapult/) project from Microsoft - a scalable, flexible FPGA network deployment in datacenters, to accelerate large scale computation (e.g. Bing Search Engine, and etc.). Each server node uses one FPGA board, which is connected to the host through PCIe, and build a mesh network together with all other FPGAs with a 'torus' topology.

This paper presents some interesting design ideas on hardware, network topology, and management software, and design decisions to maximize the performance output of this large-scale deployment of accelerated datacenter.



This is an interesting project to follow for anyone who's interested in HPC and heterogeneous computations, as it provides an outstanding design in all key parts of HPC datacenter, from hardware selection, network and software management and programming interface. Unfortunately it didn't elaborate on the details of software side, which is the part I'm mostly interested in.
