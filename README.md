# cldemo-tacacs

This demo  create a base line tacacs demo that shows user authentication working.

It has two roles:
1. Deploy TACACS server package onto oob-mgmt-server
* Configure TACACS server with user account to allow login.
2. Deploy TACACS client package onto all switches in infrastructure
* Configure TACACS client to use TACACS server for authentication

There is a client called **adminuser** with password **password**. Test that tacacs is working by trying to log in with the user:
`ssh adminuser@leaf01`

# Steps to run demo
```
git clone https://github.com/CumulusNetworks/cldemo-vagrant.git
cd cldemo-vagrant
vagrant up oob-mgmt-server oob-mgmt-switch
vagrant up leaf01 leaf02 leaf03 leaf04 spine01 spine02
```

```
vagrant ssh oob-mgmt-server
git clone https://github.com/CumulusNetworks/cldemo-tacacs.git
cd cldemo-tacacs
ansible-playbook run-demo.yml
```
## Test the Adminuser
The adminuser account has the ability to use SUDO and escalate their privileges to perform anything on the switch.
```
cumulus@oob-mgmt-server:~/cldemo-tacacs$ ssh adminuser@leaf01
adminuser@leaf01's password: password

Welcome to Cumulus VX (TM)

Cumulus VX (TM) is a community supported virtual appliance designed for
experiencing, testing and prototyping Cumulus Networks' latest technology.
For any questions or technical support, visit our community site at:
http://community.cumulusnetworks.com

The registered trademark Linux (R) is used pursuant to a sublicense from LMI,
the exclusive licensee of Linus Torvalds, owner of the mark on a world-wide
basis.
adminuser@leaf01:~$
```
Verify your privilege level: 
```
adminuser@leaf01:~$ pwd
/home/tacacs15
```


Look at logs:
```
adminuser@leaf01:~$ sudo tail -f /var/log/syslog
2017-09-05T22:29:38.173712+00:00 leaf01 sshd[3029]: Accepted password for adminuser from 192.168.0.254 port 40259 ssh2
2017-09-05T22:29:38.208347+00:00 leaf01 sshd[3029]: pam_unix(sshd:session): session opened for user adminuser by (uid=0)
```

```
adminuser@leaf01:~$ groups
tacacs
```

```
adminuser@leaf01:~$ sudo getent passwd adminuser
adminuser:x:1017:1002:TACACS+ mapped user at privilege level 15,,,:/home/tacacs15:/bin/bash
```

#### See Tacacs Events in the log file.
```
cumulus@oob-mgmt-server:~$ sudo tail -f /var/log/tac_plus.acct 
Nov 30 01:27:54	192.168.0.11	adminuser	pts0	oob-mgmt-server	stop		start_time=1512005274	task_id=10780	service=shell	cmd=/bin/ls exit=0

```

#### See Connection Events in TACACS Server Syslog
```
cumulus@oob-mgmt-server:~$ sudo grep tac_plus /var/log/syslog
2017-11-29T19:25:02.300619+00:00 oob-mgmt-server tac_plus[26873]: Reading config
2017-11-29T19:25:02.302364+00:00 oob-mgmt-server tac_plus[26873]: Version F4.0.4.27a Initialized 1
2017-11-29T19:25:02.304692+00:00 oob-mgmt-server tac_plus[26874]: Reading config
2017-11-29T19:25:02.305469+00:00 oob-mgmt-server tac_plus[26874]: Version F4.0.4.27a Initialized 1
2017-11-29T19:25:02.313412+00:00 oob-mgmt-server tac_plus[26877]: socket FD 0 AF 2
2017-11-29T19:25:02.314641+00:00 oob-mgmt-server tac_plus[26877]: socket FD 2 AF 10
2017-11-29T19:25:02.315544+00:00 oob-mgmt-server tac_plus[26877]: uid=0 euid=0 gid=0 egid=0 s=-20254704
2017-11-29T19:25:08.862814+00:00 oob-mgmt-server ansible-stat: Invoked with checksum_algorithm=sha1 get_checksum=True path=/etc/tacacs+/tac_plus.conf checksum_algo=sha1 follow=False get_md5=False get_mime=True get_attributes=True
2017-11-29T19:25:09.368141+00:00 oob-mgmt-server ansible-copy: Invoked with directory_mode=None force=True remote_src=None owner=None follow=False local_follow=None group=None unsafe_writes=None setype=None content=NOT_LOGGING_PARAMETER serole=None dest=/etc/tacacs+/tac_plus.conf selevel=None original_basename=tac_plus.conf regexp=None validate=None src=/home/cumulus/.ansible/tmp/ansible-tmp-1511983508.22-98887147948235/source seuser=None delimiter=None mode=None attributes=None backup=False
2017-11-29T19:25:10.299916+00:00 oob-mgmt-server tac_plus[26877]: Received signal 15, shutting down
2017-11-29T19:25:10.316821+00:00 oob-mgmt-server tac_plus[27403]: Reading config
2017-11-29T19:25:10.318210+00:00 oob-mgmt-server tac_plus[27403]: Version F4.0.4.27a Initialized 1
2017-11-29T19:25:10.320373+00:00 oob-mgmt-server tac_plus[27404]: Reading config
2017-11-29T19:25:10.320966+00:00 oob-mgmt-server tac_plus[27404]: Version F4.0.4.27a Initialized 1
2017-11-29T19:25:10.323448+00:00 oob-mgmt-server tac_plus[27407]: socket FD 0 AF 2
2017-11-29T19:25:10.323944+00:00 oob-mgmt-server tac_plus[27407]: socket FD 2 AF 10
2017-11-29T19:25:10.324486+00:00 oob-mgmt-server tac_plus[27407]: uid=0 euid=0 gid=0 egid=0 s=-796471744
2017-11-29T19:30:01.270130+00:00 oob-mgmt-server tac_plus[27696]: connect from 192.168.0.14 [192.168.0.14]
2017-11-29T19:30:01.284004+00:00 oob-mgmt-server tac_plus[27698]: connect from 192.168.0.14 [192.168.0.14]
2017-11-29T19:30:01.591892+00:00 oob-mgmt-server tac_plus[27702]: connect from 192.168.0.11 [192.168.0.11]
2017-11-29T19:30:01.595533+00:00 oob-mgmt-server tac_plus[27703]: connect from 192.168.0.11 [192.168.0.11]
2017-11-29T19:30:01.719082+00:00 oob-mgmt-server tac_plus[27704]: connect from 192.168.0.13 [192.168.0.13]
2017-11-29T19:30:01.723899+00:00 oob-mgmt-server tac_plus[27705]: connect from 192.168.0.13 [192.168.0.13]
2017-11-29T19:30:01.766067+00:00 oob-mgmt-server tac_plus[27707]: connect from 192.168.0.12 [192.168.0.12]
2017-11-29T19:30:01.769761+00:00 oob-mgmt-server tac_plus[27708]: connect from 192.168.0.12 [192.168.0.12]
2017-11-29T19:30:01.881050+00:00 oob-mgmt-server tac_plus[27709]: connect from 192.168.0.21 [192.168.0.21]
```

## Try the Basicuser
The basicuser is a limited privilege account designed to provide troubleshooting abilities to NOC users. This account cannot make configuration chages or use sudo aside from for commands related to troubleshooting activities such as performing a packetcapture with TCPdump or collecting a cl-support file or looking at lldpctl output.
```
cumulus@oob-mgmt-server:~/cldemo-tacacs$ ssh basicuser@leaf01
basicuser@leaf02's password: password
```

### Notice the Limited Privileges

Notice that you cannot make configuration changes, only use show commands:
```
basicuser@leaf02:~$ net show interface
State  Name           Spd  MTU    Mode          LLDP                    Summary
-----  -------------  ---  -----  ------------  ----------------------  -------------------------
UP     lo             N/A  65536  Loopback                              IP: 127.0.0.1/8
       lo                                                               IP: 10.255.255.2/32
       lo                                                               IP: ::1/128
UP     eth0           1G   1500   Mgmt          oob-mgmt-switch (swp7)  IP: 192.168.0.12/24(DHCP)
UP     swp1           1G   1500   BondMember    server01 (eth2)         Master: SERVER01(UP)
UP     swp2           1G   1500   BondMember    server02 (eth2)         Master: SERVER02(UP)
UP     swp49          1G   1500   BondMember    leaf01 (swp49)          Master: peerlink(UP)
UP     swp50          1G   1500   BondMember    leaf01 (swp50)          Master: peerlink(UP)
UP     SERVER01       1G   1500   LACP                                  Master: bridge(UP)
       SERVER01                                                         Bond Members: swp1(UP)
UP     SERVER02       1G   1500   LACP                                  Master: bridge(UP)
       SERVER02                                                         Bond Members: swp2(UP)
UP     bridge         N/A  1500   Bridge/L2
UP     peerlink       2G   1500   LACP                                  Master: bridge(UP)
       peerlink                                                         Bond Members: swp49(UP)
       peerlink                                                         Bond Members: swp50(UP)
UP     peerlink.4094  2G   1500   SubInt/L3                             IP: 169.254.1.2/30
UP     vlan10         N/A  1500   Interface/L3                          IP: 10.0.10.3/24
UP     vlan10-v0      N/A  1500   Interface/L3                          IP: 10.0.10.1/24
UP     vlan20         N/A  1500   Interface/L3                          IP: 10.0.20.3/24
UP     vlan20-v0      N/A  1500   Interface/L3                          IP: 10.0.20.1/24

basicuser@leaf02:~$ net add bgp autonomous-system 65000
ERROR: You do not have permission to execute that command.
basicuser@leaf02:~$ tail /var/log/syslog
2018-05-04T18:00:51.990777+00:00 leaf02 phc2sys: [271230.677] Waiting for ptp4l...
2018-05-04T18:00:51.991442+00:00 leaf02 phc2sys: [271230.677] uds: sendto failed: No such file or directory
2018-05-04T18:00:52.992084+00:00 leaf02 phc2sys: [271231.678] Waiting for ptp4l...
2018-05-04T18:00:52.992547+00:00 leaf02 phc2sys: [271231.678] uds: sendto failed: No such file or directory
2018-05-04T18:00:53.993434+00:00 leaf02 phc2sys: [271232.680] Waiting for ptp4l...
2018-05-04T18:00:53.993839+00:00 leaf02 phc2sys: [271232.680] uds: sendto failed: No such file or directory
2018-05-04T18:00:54.994669+00:00 leaf02 phc2sys: [271233.681] Waiting for ptp4l...
2018-05-04T18:00:54.995078+00:00 leaf02 phc2sys: [271233.681] uds: sendto failed: No such file or directory
2018-05-04T18:00:55.995926+00:00 leaf02 phc2sys: [271234.682] Waiting for ptp4l...
2018-05-04T18:00:55.996381+00:00 leaf02 phc2sys: [271234.682] uds: sendto failed: No such file or directory

```
## Try  the timeuser
The Timeuser in this case has very limited access to the system, and can only access the system to get current time information and ntp server information. This is example of Cumulus TACACS+ Per-command Authorization feature.
```
cumulus@oob-mgmt-server:~$ ssh timeuser@leaf02
timeuser@leaf02's password:
timeuser@leaf02:~$ pwd
/home/tacacs1
timeuser@leaf02:~$ 
timeuser@leaf02:~$ 
timeuser@leaf02:~$ 
timeuser@leaf02:~$ net show time 
Mon Oct 14 20:00:28 UTC 2019 
timeuser@leaf02:~$ net show time json
{"time": "Mon Oct 14 20:00:31 UTC 2019"} 
timeuser@leaf02:~$ net show time ntp servers
remote           refid      st t when poll reach   delay   offset  jitter
==============================================================================
*oob-mgmt-server 204.2.134.162    3 u  309 1024  377    3.135    2.727   4.105 
```
### No access to other net show commands 
```
timeuser@leaf02:~$ net show route 
net not authorized by TACACS+ with given arguments, not executing
timeuser@leaf02:~$ net show inter
net not authorized by TACACS+ with given arguments, not executing
timeuser@leaf02:~$ 
timeuser@leaf02:~$ 

```
#### Tacacs Server logs for timeuser
```
Oct 14 20:00:28	192.168.0.12	timeuser	pts0	oob-mgmt-server	start		start_time=1571083228	task_id=17831	service=shell	cmd=/usr/bin/python2.7 /usr/bin/net show time
Oct 14 20:00:28	192.168.0.12	timeuser	pts0	oob-mgmt-server	start		start_time=1571083228	task_id=17831	service=shell	cmd=/usr/bin/python2.7 /usr/bin/net show time
Oct 14 20:00:28	192.168.0.12	timeuser	pts0	oob-mgmt-server	stop		start_time=1571083228	task_id=17831	service=shell	cmd=/usr/bin/python2.7 exit=0
Oct 14 20:00:28	192.168.0.12	timeuser	pts0	oob-mgmt-server	stop		start_time=1571083228	task_id=17831	service=shell	cmd=/usr/bin/python2.7 exit=0
Oct 14 20:00:30	192.168.0.12	timeuser	pts0	oob-mgmt-server	start		start_time=1571083230	task_id=17833	service=shell	cmd=/usr/sbin/tacplus-auth show time json
Oct 14 20:00:30	192.168.0.12	timeuser	pts0	oob-mgmt-server	start		start_time=1571083230	task_id=17833	service=shell	cmd=/usr/sbin/tacplus-auth show time json
Oct 14 20:00:30	192.168.0.12	timeuser	pts0	oob-mgmt-server	start		start_time=1571083230	task_id=17833	service=shell	cmd=/usr/bin/python2.7 /usr/bin/net show time json
Oct 14 20:00:30	192.168.0.12	timeuser	pts0	oob-mgmt-server	start		start_time=1571083230	task_id=17833	service=shell	cmd=/usr/bin/python2.7 /usr/bin/net show time json
Oct 14 20:00:31	192.168.0.12	timeuser	pts0	oob-mgmt-server	stop		start_time=1571083231	task_id=17833	service=shell	cmd=/usr/bin/python2.7 exit=0
Oct 14 20:00:31	192.168.0.12	timeuser	pts0	oob-mgmt-server	stop		start_time=1571083231	task_id=17833	service=shell	cmd=/usr/bin/python2.7 exit=0
Oct 14 20:00:35	192.168.0.12	timeuser	pts0	oob-mgmt-server	start		start_time=1571083235	task_id=17835	service=shell	cmd=/usr/sbin/tacplus-auth show time ntp servers
Oct 14 20:00:35	192.168.0.12	timeuser	pts0	oob-mgmt-server	start		start_time=1571083235	task_id=17835	service=shell	cmd=/usr/sbin/tacplus-auth show time ntp servers
Oct 14 20:00:35	192.168.0.12	timeuser	pts0	oob-mgmt-server	start		start_time=1571083235	task_id=17835	service=shell	cmd=/usr/bin/python2.7 /usr/bin/net show time ntp servers
Oct 14 20:00:35	192.168.0.12	timeuser	pts0	oob-mgmt-server	start		start_time=1571083235	task_id=17835	service=shell	cmd=/usr/bin/python2.7 /usr/bin/net show time ntp servers
Oct 14 20:00:35	192.168.0.12	timeuser	pts0	oob-mgmt-server	stop		start_time=1571083235	task_id=17835	service=shell	cmd=/usr/bin/python2.7 exit=0
Oct 14 20:00:35	192.168.0.12	timeuser	pts0	oob-mgmt-server	stop		start_time=1571083235	task_id=17835	service=shell	cmd=/usr/bin/python2.7 exit=0
Oct 14 20:00:39	192.168.0.12	timeuser	pts0	oob-mgmt-server	start		start_time=1571083239	task_id=17848	service=shell	cmd=/usr/sbin/tacplus-auth show route
Oct 14 20:00:39	192.168.0.12	timeuser	pts0	oob-mgmt-server	start		start_time=1571083239	task_id=17848	service=shell	cmd=/usr/sbin/tacplus-auth show route
Oct 14 20:00:39	192.168.0.12	timeuser	pts0	oob-mgmt-server	stop		start_time=1571083239	task_id=17848	service=shell	cmd=/usr/sbin/tacplus-auth exit=1
Oct 14 20:00:39	192.168.0.12	timeuser	pts0	oob-mgmt-server	stop		start_time=1571083239	task_id=17848	service=shell	cmd=/usr/sbin/tacplus-auth exit=1
Oct 14 20:04:51	192.168.0.12	timeuser	pts0	oob-mgmt-server	start		start_time=1571083491	task_id=18371	service=shell	cmd=/usr/sbin/tacplus-auth show inter
Oct 14 20:04:51	192.168.0.12	timeuser	pts0	oob-mgmt-server	start		start_time=1571083491	task_id=18371	service=shell	cmd=/usr/sbin/tacplus-auth show inter
Oct 14 20:04:51	192.168.0.12	timeuser	pts0	oob-mgmt-server	stop		start_time=1571083491	task_id=18371	service=shell	cmd=/usr/sbin/tacplus-auth exit=1
Oct 14 20:04:51	192.168.0.12	timeuser	pts0	oob-mgmt-server	stop		start_time=1571083491	task_id=18371	service=shell	cmd=/usr/sbin/tacplus-auth exit=1
root@oob-mgmt-server:~/cldemo-tacacs# 

```


