
# Ansitool

A tool for building standalone binaries that can run Ansible recipes locally.  

## Usage

Use `ansitool new` to start a new blank standalone Ansible project.

Build your playbooks using `ansitool pack`, then copy pack.bash to the remote machine.
Run `bash pack.bash` to have your playbooks executed on the local machine.

You can also do your development of your playbooks on the target server for easy debugging
and fast iteration, then copy the playbooks into your source control once you are happy with it.

## Example

Start off with just ansitool in a directory

```
rchapman@dev1:~/ansitool# ls -l
total 8
-rwxr-xr-x 1 root root 6672 Nov  6 02:34 ansitool
```

### ansitool deps

You'll need to run `ansitool deps` to make sure all dependencies are installed.  This currently requires sudo
to install the latest ansible.  It requires Ubuntu; however, it wouldn't be very difficult to add other distributions. It's just that I don't have access to anything else at the moment, nor do I have the time.  Feel free to send me a pull request if you do add additional distros.
As long as you have ansible version 2.2 or better, you don't have to run `ansitool deps`.

```
rchapman@dev1:~/ansitool# ./ansitool deps
Nov 06 03:19:49.622836437 UTC ansitool[7303]: Checking if we need to install a version of ansible newer than 2.2
./ansitool: line 91: ansible-playbook: command not found
Nov 06 03:19:49.626255743 UTC ansitool[7303]: Checking if we need to install a version of ansible newer than 2.2: yes
No LSB modules are available.
Nov 06 03:19:49.683069349 UTC ansitool[7303]: Installing latest version of Ansible
Nov 06 03:19:49.684808979 UTC ansitool[7303]: sudo apt-get -y install software-properties-common
Reading package lists... Done
Building dependency tree
Reading state information... Done
software-properties-common is already the newest version (0.96.20.4).
The following packages were automatically installed and are no longer required:
  ieee-data python-crypto python-ecdsa python-httplib2 python-jinja2 python-markupsafe python-netaddr python-paramiko python-selinux python-yaml sshpass
Use 'sudo apt autoremove' to remove them.
0 upgraded, 0 newly installed, 0 to remove and 6 not upgraded.
Nov 06 03:19:50.719478137 UTC ansitool[7303]: sudo apt-get -y install software-properties-common returned 0
Nov 06 03:19:50.721253276 UTC ansitool[7303]: sudo apt-add-repository -y ppa:ansible/ansible
gpg: keyring `/tmp/tmplfs_8lny/secring.gpg' created
gpg: keyring `/tmp/tmplfs_8lny/pubring.gpg' created
gpg: requesting key 7BB9C367 from hkp server keyserver.ubuntu.com
gpg: /tmp/tmplfs_8lny/trustdb.gpg: trustdb created
gpg: key 7BB9C367: public key "Launchpad PPA for Ansible, Inc." imported
gpg: Total number processed: 1
gpg:               imported: 1  (RSA: 1)
OK
Nov 06 03:19:52.251178749 UTC ansitool[7303]: sudo apt-add-repository -y ppa:ansible/ansible returned 0
Nov 06 03:19:52.252952541 UTC ansitool[7303]: sudo apt-get -y update
Hit:1 http://us-central1.gce.archive.ubuntu.com/ubuntu xenial InRelease
Hit:2 http://us-central1.gce.archive.ubuntu.com/ubuntu xenial-updates InRelease
Hit:3 http://us-central1.gce.archive.ubuntu.com/ubuntu xenial-backports InRelease
Hit:4 http://apt.postgresql.org/pub/repos/apt xenial-pgdg InRelease
Hit:5 http://ppa.launchpad.net/ansible/ansible/ubuntu xenial InRelease
Hit:6 http://archive.canonical.com/ubuntu xenial InRelease
Hit:7 http://ppa.launchpad.net/ubuntugis/ppa/ubuntu xenial InRelease
Hit:8 http://ppa.launchpad.net/webupd8team/java/ubuntu xenial InRelease
Hit:9 http://security.ubuntu.com/ubuntu xenial-security InRelease
Reading package lists... Done
Nov 06 03:19:55.349270843 UTC ansitool[7303]: sudo apt-get -y update returned 0
Nov 06 03:19:55.350967974 UTC ansitool[7303]: sudo apt-get -y install ansible
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following packages were automatically installed and are no longer required:
  ieee-data python-netaddr python-selinux
Use 'sudo apt autoremove' to remove them.
The following NEW packages will be installed:
  ansible
0 upgraded, 1 newly installed, 0 to remove and 6 not upgraded.
Need to get 0 B/1,608 kB of archives.
After this operation, 12.6 MB of additional disk space will be used.
Selecting previously unselected package ansible.
(Reading database ... 97904 files and directories currently installed.)
Preparing to unpack .../ansible_2.2.0.0-1ppa~xenial_all.deb ...
Unpacking ansible (2.2.0.0-1ppa~xenial) ...
Processing triggers for man-db (2.7.5-1) ...
Setting up ansible (2.2.0.0-1ppa~xenial) ...
Nov 06 03:19:59.572028750 UTC ansitool[7303]: sudo apt-get -y install ansible returned 0
Nov 06 03:19:59.573983379 UTC ansitool[7303]: Installing latest version of Ansible: done
Nov 06 03:19:59.575930497 UTC ansitool[7303]: Checking if we need to install a version of ansible newer than 2.2: done.

rchapman@dev1:~/ansitool#
```

If your system doesn't need ansible updated, you see something like this:

```
rchapman@dev1:~/ansitool# ./ansitool deps
Nov 06 03:20:44.129356568 UTC ansitool[8210]: Checking if we need to install a version of ansible newer than 2.2
Nov 06 03:20:44.365667927 UTC ansitool[8210]: Checking if we need to install a version of ansible newer than 2.2: not needed.

rchapman@dev1:~/ansitool#
```

### ansitool new

Create a new project

```
rchapman@dev1:~/ansitool# ./ansitool new
Nov 06 03:00:58.866700157 UTC ansitool[3350]: New ansible application with role main_role created.

rchapman@dev1:~/ansitool# ls -l
total 20
-rwxr-xr-x 1 root root 6672 Nov  6 02:34 ansitool
-rw-r--r-- 1 root root   48 Nov  6 03:00 hosts
-rw-r--r-- 1 root root   48 Nov  6 03:00 main.yml
drwxr-xr-x 3 root root 4096 Nov  6 03:00 roles
rchapman@dev1:~/ansitool#
```

### ansitool run

Run the project.  By default, the main_role role has a task that prints out host variables if the verbosity level is 3 or greater.
The default verbosity is 0, so `ansitool run` doesn't print host variables:

```
rchapman@dev1:~/ansitool# ./ansitool run
Nov 06 03:07:06.244846439 UTC ansitool[4598]: ansible-playbook --inventory-file ./hosts main.yml

PLAY [localhost] ***************************************************************

TASK [setup] *******************************************************************
ok: [localhost]

TASK [main_role : Display all variables for localhost] *************************
skipping: [localhost]

PLAY RECAP *********************************************************************
localhost                  : ok=1    changed=0    unreachable=0    failed=0

Nov 06 03:07:07.145924622 UTC ansitool[4598]: ansible-playbook --inventory-file ./hosts main.yml returned 0

rchapman@dev1:~/ansitool#
```

### ansitool run debug

Running `ansitool run debug` sets the verbosity level to 3, which causes host variables to print:

```
rchapman@dev1:~/ansitool# ./ansitool run debug
Nov 06 03:09:19.483271341 UTC ansitool[4964]: ansible-playbook --inventory-file ./hosts main.yml -vvv
Using /etc/ansible/ansible.cfg as config file

PLAYBOOK: main.yml *************************************************************
1 plays in main.yml

PLAY [localhost] ***************************************************************

TASK [setup] *******************************************************************
Using module file /usr/lib/python2.7/dist-packages/ansible/modules/core/system/setup.py
<localhost> ESTABLISH LOCAL CONNECTION FOR USER: root
<localhost> EXEC /bin/sh -c '( umask 77 && mkdir -p "` echo $HOME/.ansible/tmp/ansible-tmp-1478401759.98-111063585839069 `" && echo ansible-tmp-1478401759.98-111063585839069="` echo $HOME/.ansible/tmp/ansible-tmp-1478401759.98-111063585839069 `" ) && sleep 0'
<localhost> PUT /tmp/tmp0_o5He TO /home/rchapman/.ansible/tmp/ansible-tmp-1478401759.98-111063585839069/setup.py
<localhost> EXEC /bin/sh -c 'chmod u+x /home/rchapman/.ansible/tmp/ansible-tmp-1478401759.98-111063585839069/ /home/rchapman/.ansible/tmp/ansible-tmp-1478401759.98-111063585839069/setup.py && sleep 0'
<localhost> EXEC /bin/sh -c '/usr/bin/python /home/rchapman/.ansible/tmp/ansible-tmp-1478401759.98-111063585839069/setup.py; rm -rf "/home/rchapman/.ansible/tmp/ansible-tmp-1478401759.98-111063585839069/" > /dev/null 2>&1 && sleep 0'
ok: [localhost]

TASK [main_role : Display all variables for localhost] *************************
task path: /home/rchapman/ansitool/roles/main_role/tasks/main.yml:2
ok: [localhost] => {
    "hostvars[inventory_hostname]": {
        "ansible_all_ipv4_addresses": [
            "10.128.0.2"
        ],

[...]

        "playbook_dir": "/home/rchapman/ansitool"
    }
}

PLAY RECAP *********************************************************************
localhost                  : ok=2    changed=0    unreachable=0    failed=0

Nov 06 03:09:20.312657635 UTC ansitool[4964]: ansible-playbook --inventory-file ./hosts main.yml -vvv returned 0

rchapman@dev1:~/ansitool#
```

### ansitool pack

Running `ansitool pack` builds pack.bash, which is just a bash script with an embedded tar.gz file in it

```
rchapman@dev1:~/ansitool# ./ansitool pack
Nov 06 03:12:48.142904365 UTC ansitool[5526]: Building pack in /tmp/XXX-HPi.tar.gz
Nov 06 03:12:48.145028680 UTC ansitool[5526]: if [[ -f pack.bash ]]; then rm -f pack.bash; fi
Nov 06 03:12:48.147025395 UTC ansitool[5526]: if [[ -f pack.bash ]]; then rm -f pack.bash; fi returned 0
Nov 06 03:12:48.148856829 UTC ansitool[5526]: tar zcvf /tmp/XXX-HPi.tar.gz .
./
./hosts
./.ansitool.swp
./ansitool
./roles/
./roles/main_role/
./roles/main_role/tasks/
./roles/main_role/tasks/main.yml
./roles/main_role/files/
./main.yml
Nov 06 03:12:48.153494005 UTC ansitool[5526]: tar zcvf /tmp/XXX-HPi.tar.gz . returned 0
Nov 06 03:12:48.156132072 UTC ansitool[5526]: cat /tmp/XXX-HPi.tar.gz >> pack.bash
Nov 06 03:12:48.158575819 UTC ansitool[5526]: cat /tmp/XXX-HPi.tar.gz >> pack.bash returned 0
Nov 06 03:12:48.160204657 UTC ansitool[5526]: rm -f /tmp/XXX-HPi.tar.gz
Nov 06 03:12:48.162659036 UTC ansitool[5526]: rm -f /tmp/XXX-HPi.tar.gz returned 0
Nov 06 03:12:48.164388768 UTC ansitool[5526]: Building pack in /tmp/XXX-HPi.tar.gz: done

rchapman@dev1:~/ansitool#
```

### Copy pack.bash to remote machine 

Now, copy the pack.bash file to the remote machine you want to run the playbooks:

```
rchapman@dev1:~/ansitool# scp pack.bash rchapman@dev2:
rchapman@dev2's password:
tput: No value for $TERM and no -T specified
pack.bash                                                                                                                                        100% 5373     5.3KB/s   00:00
rchapman@dev1:~/ansitool#
```

### bash pack.bash

On the remote machine where you want to run your playbooks, execute `bash pack.bash`
Note that it will check to see if ansible is at least 2.2 and attempt to install it 
if it is not.  As long as you are running as root on Ubuntu, it should all just work (hopefully).

```
rchapman@dev2:~/tmp# ls -l
total 8
-rw-r--r-- 1 root root 5373 Nov  6 03:12 pack.bash
rchapman@dev2:~/tmp#
rchapman@dev2:~/tmp#
rchapman@dev2:~/tmp# bash pack.bash
Extracting files into /tmp/k2gd9s
./
./hosts
./.ansitool.swp
./ansitool
./roles/
./roles/main_role/
./roles/main_role/tasks/
./roles/main_role/tasks/main.yml
./roles/main_role/files/
./main.yml
cd /tmp/k2gd9s
skipping update of ansible, not needed (you have at least version 2.2)
sudo bash ansitool run
Nov 06 03:24:22.484859720 UTC ansitool[8782]: ansible-playbook --inventory-file ./hosts main.yml

PLAY [localhost] ***************************************************************

TASK [setup] *******************************************************************
ok: [localhost]

TASK [main_role : Display all variables for localhost] *************************
skipping: [localhost]

PLAY RECAP *********************************************************************
localhost                  : ok=1    changed=0    unreachable=0    failed=0

Nov 06 03:24:23.297940364 UTC ansitool[8782]: ansible-playbook --inventory-file ./hosts main.yml returned 0

cd /root/tmp
Cleaning up directory /tmp/k2gd9s
rchapman@dev2:~/tmp#
```

### Done

That's it.  You have a simple python based tool to deploy infrastructure without complicated
setup and with no dependencies on having a master server :)

Enjoy.  Please send feedback.


