# Introduction

Docker utilizes Linux namespace to isolate resource for containers like network, filesystem. Yet all containers share the same system performance metrics, which cause traditional system performnce tools report metrics for whole host instead of each container.

This project provide one solution to get system performance metric per container from inside.

# How it works

Each docker container belongs to one separate cgroup, resources consumed, CPU, Memory etc are recorded in cgroup files. Instead of reading system-wide metric sources (e.g, /proc/meminfo), calculating metrics from cgroup records results in more accurate results for each container. This is how most container monitor tools work.

What make the diffrence of this project is that it take usage of LD_PRELOAD to enable existed tools to calculating matric from cgroup records. On the initialization of inject.so, libc functions like open, lseek are hijacked. hijacked version will calculate from cgroup records once application try to read related system-wide metric sources, otherwise it will pass through request to original version.

# How to use

  * build inject.so
  
        > make

  * run a container with required volumes and get right metric
  
        > docker run -ti --rm -v <inject.so>:/usr/lib/inject.so -v /sys/fs/cgroup:/sys/fs/cgroup:ro ubuntu bash
		> # in container
		> export CONTAINER_PROC_INJECT_TARGETS=free:top:uptime
		> export LD_PRELOAD=/usr/lib/inject.so
		> uptime

# Progress
  * (DONE)open, fopen, and lseek on /proc/uptime
  * (DONE)open, fopen, and lseek on /proc/cpuinfo
  * (DONE)open, fopen, and lseek on /proc/meminfo
  * (DONE)open, fopen, and lseek on /proc/stat
  * (DONE)open, fopen, and lseek on /proc/diskstats
  * (DONE)open, fopen, and lseek on /system/devices/system/cpu/online
  * (DONE)sysinfo
  * (DONE)sysconf(\_SC\_NPROCESSORS\_ONLN)
  * (TODO)open, fopen, and lseek on /proc/loadavg

# Reference

  * lxcfs: https://linuxcontainers.org/lxcfs/introduction/
    Some metric calculating algorithms come from lxcfs
