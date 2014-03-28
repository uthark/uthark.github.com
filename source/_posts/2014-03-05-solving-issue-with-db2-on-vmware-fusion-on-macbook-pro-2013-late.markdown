---
layout: post
title: "Solving issue with DB2 on VMWare Fusion on MacBook Pro 2013 Late"
date: 2014-03-05 14:41:54 -0500
comments: true
categories: 
    - development
    - macbook
    - vmware fusion
    - db2
---

Today I found issue running DB2 10.1 under VMWare Fusion 6.0.2 on my MacBook - all management services for `db2` are starting except for the db2 instance itself.

When I tried to start DB2 using `db2start` it just crashed without any additional information. 

After some googling I found out that it is due to the fact that current version of DB2 can't handle more than 4 responses to CPUID query.

The workaround is to update VMWare configuration file for the virtual machine.

``` ini Changes to *.vmx file
monitor_control.enable_fullcpuid = TRUE
cpuid.4.4.eax = "0000:0000:0000:0000:0000:0000:0000:0000"
```

The answer was found on [VMWare Community Forums](https://communities.vmware.com/thread/461857)