---
published: true
title: Mapping PPP lines to their configuartion
layout: post
---
I had an interesting use-case of PPP in my prevision job. We used to have multiple PPP interfaces configured on our Linux(CentOS) servers each connecting to a DSL line. In some cases the PPP configuration of all the interfaces were the same, which was easy to manage. However, In most cases we had PPP configuration different for each interface. This was because we used DSL’s from multiple ISP’s and so on. I won’t go into why we were using multiple ISP’s here.
Add to this we also need to bounce interfaces at certain interval. Also some interfaces will go down due ISP issues.

The problem was how to track the line down to the specific ISP DSL account(in /etc/sysconfig/network-scripts/ifcfg-pppNUM) and open a case with them. Since Linux did dynamic network interface naming the PPP interfaces will take the lowest possible interface number. One way to track the interfaces that were having issues was to shut all the lines and bring one-by-one up and down checking the connectivity.

On searching a lot about this. I found a better way to track it.

The PID file /var/run/ppp-ppp”CURRENT_NUM”.pid where CURRENT_NUM is the PPP interface number currently, always contains a pointer to the original configuration file (/etc/sysconfig/network-scripts/ifcfg-pppNUM).

Also wrote a script to map current interface number to original configuration showing the interfaces that are down.

PPP Mapping Script

```bash
#!/bin/bash
Total_PPP=$(ls /etc/sysconfig/network-scripts/ifcfg-ppp* |wc -l)
echo -e “PPP orig. config(ifcfg-pppN)  -> current PPP interface\tIPAddr”
for num in $(seq 0 $((Total_PPP-1)) )
do
if [ -f /var/run/ppp-ppp”$num”.pid ]
then
current_ino=$(sed -n ‘2p’ /var/run/ppp-ppp”$num”.pid)
IPAddr=$(/sbin/ifconfig “$current_ino” |awk ‘/inet/ { print $2}’|cut -d: -f2)
echo -e “$num -> $current_ino\t$IPAddr”
else
echo “$num -> DOWN”
fi
done
```

References :
[PppConnectionNaming](https://utcc.utoronto.ca/~cks/space/blog/linux/PppConnectionNaming)
[ How force pppd tu use same ppp interface name](https://www.mail-archive.com/linux-ppp@vger.rutgers.edu/msg03064.html)