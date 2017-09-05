# cldemo-tacacs

This demo  create a base line tacacs demo that shows user authentication working.

It has two roles:
1. Deploy TACACS server package onto oob-mgmt-server
* Configure TACACS server with user account to allow login.
2. Deploy TACACS client package onto all switches in infrastructure
* Configure TACACS client to use TACACS server for authentication

There is a client called **testuser** with password **cn321**. Test that tacacs is working by trying to log in with the user:
`ssh testuser@leaf01`

#Steps to run demo
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