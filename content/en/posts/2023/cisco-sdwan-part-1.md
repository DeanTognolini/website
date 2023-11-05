---
title: "Cisco SD-WAN Part 1"

date: 2023-11-05
url: /cisco-sdwan-part-1/
categories:
  - Networking
tags:
  - Cisco
  - SD-WAN
draft: false
-----

I will be using this series of posts as an aide for teaching myself about Cisco SD-WAN. These posts are not meant to be used as a tutotial or any kind of learning guide for others and therefore may not following logical steps or may be missing crucial information!

# What is SD-WAN?
Software-Defined Wide Area Network (SD-WAN), is a networking approach that enhances the management and operation of WAN connections by decoupling the networking hardware from its control mechanism, using software for intelligent path control, optimizing application performance, and increasing agility while reducing costs compared to traditional WAN infrastructure. It allows for the dynamic routing of traffic across the most efficient and reliable pathways.

# ChatGPT's Perspective of Why We Need SD-WAN

I enjoyed this analogy so thought I'd include it here.

*Me: Tell me why we need SD-WAN and explain it like a student!*

*ChatGPT: Imagine you're playing a video game where you have to manage a bunch of roads connecting different cities—this is kind of like what SD-WAN does for computer networks. In a traditional network (without SD-WAN), you might have a single road to get information from one city to another, and if there's a traffic jam or a roadblock, your data gets stuck or delayed.*

*Now, with SD-WAN, it's like having a super smart GPS that can instantly reroute traffic, finding the best path for data to travel based on current conditions, like avoiding roadblocks (network congestion) or taking shortcuts (faster pathways). It makes sure that important information, like a live video call or a big file you're downloading, doesn't get stuck in a traffic jam.*

*For businesses, this is super important because it keeps their data moving smoothly and quickly across long distances, between different offices or even cloud services, and it helps save money because they don't have to rely on expensive, high-maintenance roads (traditional network lines). Plus, they can easily change routes if they need to without having to build new roads from scratch.*

# Cisco SD-WAN Infrastructure Components

## Edge Devices
These are the endpoints of the SD-WAN fabric, they are the physical or virtual routers, located at sites, that manage the traffic entering and existing the SD-WAN. Cisco's offerings include both the Cisco vEdge and Cisco IOS XE Catalyst SD-WAN routers.

## SD-WAN Controller
This is the brains of the SD-WAN fabric. The Controller is the central control plane element that sets policy and orchestrates traffic flow to direct traffic along the best route possible given network conditions and Service Level Agreement (SLA) policies. Cisco's offering here is the Cisco vSmart Controller.

## SD-WAN Validator
This is the connectivity orchestrator that ensures Edge Devices are able to locate and communicate with the SD-WAN Controller. SD-WAN Controllers may be behind a NAT, the Validator facilitates NAT-Traversal to enable Edge Devices to establish initial communications with the Controller and build their own communications path. Cisco's offering here is the Cisco vBond Orchestrator.

## SD-WAN Manager
This is a centralized network management system that lets you configure and manage the entire overlay network from a simple graphical dashboard.

# References
* https://www.cisco.com/c/en/us/td/docs/routers/sdwan/configuration/sdwan-xe-gs-book/system-overview.html
