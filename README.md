# cldemo-tacacs

This demo  create a base line tacacs demo that shows user authentication working.

It has two roles:
1. Deploy TACACS server package onto oob-mgmt-server
* Configure TACACS server with user account to allow login.
2. Deploy TACACS client package onto all switches in infrastructure
* Configure TACACS client to use TACACS server for authentication

There is a client called **testuser** with password **cn321**. Test that tacacs is working by trying to log in with the user:
`ssh testuser@leaf01`

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

```
cumulus@oob-mgmt-server:~/cldemo-tacacs$ ssh testuser@leaf01
testuser@leaf01's password: cn321

Welcome to Cumulus VX (TM)

Cumulus VX (TM) is a community supported virtual appliance designed for
experiencing, testing and prototyping Cumulus Networks' latest technology.
For any questions or technical support, visit our community site at:
http://community.cumulusnetworks.com

The registered trademark Linux (R) is used pursuant to a sublicense from LMI,
the exclusive licensee of Linus Torvalds, owner of the mark on a world-wide
basis.
testuser@leaf01:~$
```

Look at logs:
```
testuser@leaf01:~$ sudo tail -f /var/log/syslog
2017-09-05T22:29:38.173712+00:00 leaf01 sshd[3029]: Accepted password for testuser from 192.168.0.254 port 40259 ssh2
2017-09-05T22:29:38.208347+00:00 leaf01 sshd[3029]: pam_unix(sshd:session): session opened for user testuser by (uid=0)
```

```
testuser@leaf01:~$ groups
tacacs
```

```
testuser@leaf01:~$ sudo getent passwd testuser
testuser:x:1017:1002:TACACS+ mapped user at privilege level 15,,,:/home/tacacs15:/bin/bash
```

#### See Tacacs Events in the log file.
```
cumulus@oob-mgmt-server:~$ sudo tail -f /var/log/tac_plus.acct 
Nov 30 01:27:54	192.168.0.11	testuser	pts0	oob-mgmt-server	stop		start_time=1512005274	task_id=10780	service=shell	cmd=/bin/ls exit=0

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

