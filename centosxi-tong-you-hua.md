**vi /etc/sysctl.conf 输入**

net.ipv4.tcp\_max\_tw\_buckets = 5000

net.ipv4.tcp\_syncookies = 1

net.ipv4.tcp\_max\_syn\_backlog = 1024

net.ipv4.tcp\_synack\_retries = 2

vm.overcommit\_memory=1

vm.panic\_on\_oom=0

kernel/panic=10

kernel/panic\_on\_oops=1

net.ipv4.conf.default.rp\_filter=0

net.ipv4.conf.all.rp\_filter=0

net.ipv4.conf.eth0.rp\_filter=0

net.ipv4.conf.default.arp\_announce=2

net.ipv4.conf.all.arp\_announce=2

kernel.pid\_max=65535

