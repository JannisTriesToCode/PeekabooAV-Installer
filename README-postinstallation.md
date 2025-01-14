Things to do after the installer finishes
=========================================

## Installer
run the installer or mimic its actions

## Configure Cuckoo and VBox control
become peekaboo user to configure cuckoo and vbox control

`su peekaboo`

vboxmanage is a wrapper script that connect either over SSH
 to the virtualisation host (linux) or vboxmanageAPI.py (windows)
 
`vim vboxmanage.conf`

depending on the chosen method it is necessary to either run
the vboxmanageAPI.py or configure ssh-key authentication on
the virtualisation host
```
smbclient ... vbox/vboxmanageAPI.py ...
 PS> $env:Path += ";C:\Program Files\Oracle\VirtualBox\"
 PS> C:\Python27\python.exe .\vboxmanageAPI.py
```
or
```
scp vmhost/remote-command.sh ...
scp vmhost/authorized_keys.sample ...
scp /var/lib/peekaboo/.ssh/id_ec25519.pub ...
```

try running vboxmanage it should display its help

`vboxmanage`

## Configuration of Cuckoo

`cd /var/lib/peekaboo/.cuckoo/conf`

configure the available machines.
Make sure to set the correct IP address for cuckoo agent to
connect back to `resultserver_ip`

`vim virtualbox.conf`

Depending on the size of your installation you might want to
adjust the number of cuckoo processors (`$n`)

`vim /opt/peekaboo/bin/cuckooprocessor.sh`

You can now start peekaboo
```
systemctl start peekaboo
systemctl status peekaboo
```

Use socat to connect to peekaboo and expect the greeting
```
su -s /bin/bash amavis
socat STDIN UNIX-CONNECT:/var/run/peekaboo/peekaboo.sock
```

At this point it's already possible to check files
type the following into the previous command to check the
file.

`/var/lib/peekaboo/vboxmanage.conf`

## Amavis configuration
Set `$myhostname, $mydomain, $virus_admin, $notify_method`
and `$forward_method` according to your needs.
If postfix runs remotely `$inet_socket_bind` and `@inet_acl`
have to be adjusted.

`vim /etc/amavis/conf.d/50-user`

## Configure MDA (here described for the stand alone - demo setup)
```
cp main.cf /etc/postfix/
cp master.cf /etc/postfix/
vim /etc/postfix/main.cf
vim /etc/postfix/master.cf
echo "peekabooav-demo.int" > /etc/mailname
```

Restart postfix
```
systemctl restart postfix
systemctl status postifx
```

It's time to analyse the first email
```
./utils/checkFileWithPeekaboo.py README.md
systemctl status peekaboo
```

For the exact Demo / stand alone setup run the following commands
```
cp hosts /etc/hosts
cp interfaces /etc/network/interfaces
systemctl restart networking
```

Manually start the first virtual machine and check if its available

`ping 192.168.56.101`

Now start analysing with behaviour analysis
```
su peekaboo
```

The following command will return an ID, spinn up a VM and
ultimately produce a report

`cuckoo submit /usr/share/icons/cab_view.png`

## Configure and start Cuckoo Web UI
Adjust the IP address it binds to

`vim /etc/systemd/system/cuckoohttpd.service`

and start
```
systemctl daemon-reload
systemctl start cuckoohttpd.service
```

Use your favourite browser and connect to port 8000 and 
check "Recent" for your analysis.

## Add Dovecot and Thunderbird

For quick demo results change
`disable_plaintext_auth to no` in
`vim /etc/dovecot/confg.d/10-auth.conf`

Add user to group mail

`gpasswd -a felix mail`

Configure thunderbird:
```
felix@peekabooav-demo.int
imap 143 no connection security plain password
smpt 25 no connection security no authentication
```

## Analyse
```
Analyse: Now send an email to yourself
send another one with an attachment
have fun
goto Analyse
```
