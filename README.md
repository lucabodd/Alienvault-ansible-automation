# Alienvault-hids-autodeploy

Offering a golang program to automate ossec-hids deployment on an entire subnet (or single host).
The program will be setting up an agent that allows hosts to connect to alienvault sensors via port 1514 (UDP) and send system events.
Before starting the automatic deployment of ossec-hids you will need to tune up your alienvault USM appliance and install the golang requirements for "scanner" host. The following Docs will guide you trough the process.

## "Scanner" Setup
In order to setup the scripts on your deploy "delegated" host you will just need to run:
```
curl https://raw.githubusercontent.com/lucabodd/Alienvault-hids-autodeploy/master/setup/hids-autodeploy-install.sh | bash
```
this script will setup golang 1.13, install requirements and install golang binary itself, the program will be then located in $PATH and you will be able to run the scripts with
```
Alienvault-hids-autodeploy [Options]
```
read documentation below for program usage examples.

## SIEM or Sensor Setup
In the following doc I will refer to "Siem or Sensor" as "sensor" as it didn't make any difference; in fact a Sensor is a subsystem of USM appliance
### Sensor Automatic Setup
For automatically setting up a sensor for deployment you will just need to log in via ssh, "jailbreak" the system (option 2) and run:
```
curl https://raw.githubusercontent.com/lucabodd/Alienvault-hids-autodeploy/master/setup/sensor-setup.sh | bash
```
If, for any reason, you don't trust running scripts, read the following "Sensor Manual Setup" section, as the script is doing the exact same thing.

### Sensor Manual Setup
you need to "jailbreak" the system by default setting up .bashrc as follow.

```
# Automatically generated by ossim-reconfig scripts. DO NOT TOUCH!
# ~/.bashrc: executed by bash(1) for non-login shells.

#File changed: condition from != to ==
if [ "$jailbreak" == "yes" ];then
if [[ $- =~ "i" ]];then
        ossim-setup
exit
fi
fi
export PS1='\h:\w\$ '
umask 022
export LS_OPTIONS='--color=auto'
eval "`dircolors`"
alias ls='ls $LS_OPTIONS'
alias ll='ls $LS_OPTIONS -l'
alias l='ls $LS_OPTIONS -lA'
alias du='du -kh'
alias df='df -kTh'
alias grep='grep --color=auto'
alias egrep='egrep --color=auto'
alias fgrep='fgrep --color=auto'
alias rgrep='rgrep --color=auto'
export LC_ALL='C'
export HISTTIMEFORMAT='%F %T '
```

please, make sure to backup the original file in order to restore it in future if needed.

create a script under /usr/local/bin named cluster-delete-agent and copy the following code:
```
vi /usr/local/bin/cluster-delete-agent
```
and copy
```
if [ "$#" -ne 1 ]; then
	echo "Illegal number of parameters"
	echo "You need to provide onli cluster name or hostname as parameter"
fi
for i in $(/var/ossec/bin/manage_agents -l | grep $1 | awk '{ print $2 }' | tr -d ','); do
	/var/ossec/bin/manage_agents -r $i;
done  
/var/ossec/bin/ossec-control restart;
```
add exec permissions
```
chmod a+x /usr/local/bin/cluster-delete-agent
```

## Usage
the program can be configured with the following flags:
