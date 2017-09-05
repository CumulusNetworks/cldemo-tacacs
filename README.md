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
ssh testuser@leaf01
cn321
```

Look at logs:
```

```