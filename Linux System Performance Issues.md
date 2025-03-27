# Linux System Performance Issues  -- system is running slow and we need to start troubleshooting

  ##   so system can be slow due to main 4 reason and first we need to find out and understand where is the problem 

  1. process releted issue ? processs runnig slow
  2. Disk writing issue ? Do we have external disk where system is writing or internal disk with bad block or degraded
  3. Netwroking (file transfer ) need to find out if the transfer is causing disk issue , network issue , or memory that is actually in use.
  4. Hardware issue at last

# Troubleshooting step 
  1. Check you are on right system where the issue is
  2. check disk space ( df -h or du ) -- check if disk is full
  3. check processing ( top , free, lsmem, /proc/meminfo, vmstat, pmap <PID> , dmidecode , lscpu or /proc/cpuinf)
  4. check disk issues (iostat -y 5,lsof)
  5. check networking ( tcpdump -i enps03, lsof -i -P -n | grep -i listen, netstat -plnt or ss -plnt , iftop)
  6. check system up time ( uptime)
  7. check logs
  8. check hardware ststus by loggins into system console
  9. other tool ( htop, iotop, iptraf, psacct)


# Practical

1. Check disk Space

# tmpfs is for memory file system and can be ignored 

  [root@mylinux ~]# df -h | grep -v tmpfs
  Filesystem           Size  Used Avail Use% Mounted on
  /dev/mapper/cs-root   19G  5.9G   13G  32% /
  /dev/sda1            960M  350M  611M  37% /boot
  [root@mylinux ~]#


If we see anythoing as 99% or 100% , we need to run command du -a specifying which file system to look for 

  [root@mylinux ~]# du -ah / | sort -nr | more

2. Check processing

top --> system up time - 03:51:27 up


[root@mylinux ~]# top
top - 03:51:27 up  9:43,  3 users,  load average: 1.70, 1.00, 0.39
Tasks: 222 total,   1 running, 221 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.0 us,  0.1 sy,  0.0 ni, 99.9 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
MiB Mem :   2617.6 total,    560.4 free,   1137.1 used,   1114.7 buff/cache
MiB Swap:   2236.0 total,   2223.4 free,     12.6 used.   1480.5 avail Mem

    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
    536 root      20   0       0      0      0 S   0.3   0.0   0:07.13 xfsaild/dm-0
   3441 root      20   0       0      0      0 I   0.3   0.0   0:03.28 kworker/u15:1-events_unbound
   3491 root      20   0       0      0      0 I   0.3   0.0   0:07.31 kworker/1:2-mm_percpu_wq
   3612 root      20   0  225904   4224   3456 R   0.3   0.2   0:00.06 top
      1 root      20   0  173948  18672  11080 S   0.0   0.7   0:04.37 systemd
      2 root      20   0       0      0      0 S   0.0   0.0   0:00.31 kthreadd


   # check memory utilization 
   free -m (in megabytes)

   # List memory - tell about memory block size , total online memory we have available 

   Same thoing we can check using cat /proc/meminfo
   
        [root@mylinux ~]# lsmem
        RANGE                                  SIZE  STATE REMOVABLE BLOCK
        0x0000000000000000-0x00000000b7ffffff  2.9G online       yes  0-22
        
        Memory block size:       128M
        Total online memory:     2.9G
        Total offline memory:      0B
        
        [root@mylinux ~]# cat /proc/meminfo
        MemTotal:        2680448 kB
        MemFree:          573152 kB
        MemAvailable:    1515072 kB
        Buffers:               0 kB
        Cached:           835808 kB
        SwapCached:         4512 kB

   
   # Virtual memory status - vmstat ( tell information for swap memuory 


       [root@mylinux ~]# vmstat
    procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
     r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
     0  0  12860 573404      0 1141176    0    0    29    15   22   41  0  0 100  0  0
    [root@mylinux ~]#


# to find memory information for particular process
  pmap <PID> 

  # Find details of the hardware inside system , get cpu details , hardware details 
dmidecode | more

# check CPU architecture details like memory lsmem

lscpu or can use cat /proc/cpuinfo

  
***********************************************************
3. Now we already check CPU and memory , move to disk status

So with df -h we fiund out all good but disk can be running in degraded mode with disk capacity issues but may be disk input output is degraded.

to find out that run --> iostat -y 5    ,, running command in every 5 second and gather disk stats 

***********************************************************

now lets get into networking part

First command to run is tcpdump

  to run tcp dump , first find out which interface to get tcp dump using ifconfig command

  # if my machine is talikng to DB server and i need to verify connectivity 

  tcpdump -i enp0s3 | grep db-servr-ip-address: DB port number


# If unable to ssh from one machine to our machine then on our machine we will run below command grepping port for ssh 

lsof -i -P -n | grep 22

cupsd      915       root    7u  IPv6  22841      0t0  TCP [::1]:631 (LISTEN)
cupsd      915       root    8u  IPv4  22842      0t0  TCP 127.0.0.1:631 (LISTEN)
sshd       916       root    3u  IPv4  22871      0t0  TCP *:22 (LISTEN)
sshd       916       root    4u  IPv6  22873      0t0  TCP *:22 (LISTEN)

by runnig this we can verify that our machine is listening to port 22 

And may be we have firewall that is blocking this connection from other machine

# same informatioon i can get from netstat -plnt or ss -plnt


    [root@mylinux ~]# netstat -plnt
    Active Internet connections (only servers)
    Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
    tcp        0      0 127.0.0.1:631           0.0.0.0:*               LISTEN      915/cupsd
    tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      916/sshd: /usr/sbin
    tcp        0      0 127.0.0.1:6010          0.0.0.0:*               LISTEN      2952/sshd: pushpend
    tcp6       0      0 ::1:631                 :::*                    LISTEN      915/cupsd
    tcp6       0      0 :::22                   :::*                    LISTEN      916/sshd: /usr/sbin
    tcp6       0      0 ::1:6010                :::*                    LISTEN      2952/sshd: pushpend
    [root@mylinux ~]# ss -plnt
    State      Recv-Q     Send-Q           Local Address:Port           Peer Address:Port     Process
    LISTEN     0          4096                 127.0.0.1:631                 0.0.0.0:*         users:(("cupsd",pid=915,fd=8))
    LISTEN     0          128                    0.0.0.0:22                  0.0.0.0:*         users:(("sshd",pid=916,fd=3))
    LISTEN     0          128                  127.0.0.1:6010                0.0.0.0:*         users:(("sshd",pid=2952,fd=9))
    LISTEN     0          4096                     [::1]:631                    [::]:*         users:(("cupsd",pid=915,fd=7))
    LISTEN     0          128                       [::]:22                     [::]:*         users:(("sshd",pid=916,fd=4))
    LISTEN     0          128                      [::1]:6010                   [::]:*         users:(("sshd",pid=2952,fd=8))
    [root@mylinux ~]#


Next  iftop - display bandwidth usage on an interface by host

at last check log /var/log

tao; -100 messages | grep error


[root@mylinux log]# tail -100 messages | grep error
[root@mylinux log]# cat messages | grep error
Mar 25 18:08:33 mylinux alsactl[803]: alsa-lib main.c:1554:(snd_use_case_mgr_open) error: failed to import hw:0 use case configuration -2
[root@mylinux log]#

*******************************************************************************************************
*******************************************************************************************************
*******************************************************************************************************


# ISSUE : MACHINE GOT THE IP ADDRESS BUT WE CANNOT GO OUTSIDE 


 
