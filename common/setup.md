# Machine setup
## Java setup
Download and unpack java to some directory, ```/usr/lib/jvm/``` for example
```
$ ls -ld /usr/lib/jvm/jd*
lrwxrwxrwx 1 root root   12 May 13 20:13 /usr/lib/jvm/jdk -> jdk1.8.0_131
drwxr-xr-x 8 root root 4096 Mar 15  2017 /usr/lib/jvm/jdk1.8.0_131
```

## Environment
Environment, something like this (JAVA_HOME most important):
```
$ cat /etc/environment
JAVA_HOME="/usr/lib/jvm/jdk"
SCALA_HOME="/usr/lib/jvm/scala"
GRADLE_HOME="/usr/lib/jvm/gradle"
GROOVY_HOME="/usr/lib/jvm/groovy"
SBT_HOME="/usr/lib/jvm/sbt"
PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/lib/jvm/jdk/bin:/usr/lib/jvm/scala/bin:/usr/lib/jvm/gradle/bin:/usr/lib/jvm/groovy/bin:/usr/lib/jvm/mvn/bin:/usr/lib/jvm/sbt/bin"
LANG="en_US.UTF-8"
LC_ALL="en_US.UTF-8"
```

## Create hadoop user
Create user ```hadoop``` in group ```hadoop```
```
sudo -H useradd -m -d /home/hadoop -s /bin/bash hadoop
```

## Set up /etc/hosts
Why to set up ```/etc/hosts```:
https://web-beta.archive.org/web/20140104070155/http://blog.devving.com/why-does-hbase-care-about-etchosts

```/etc/hosts``` modifications (for Ubuntu 16.04):
```
hadoop@machinename:/etc$ diff -U100 hosts.orig hosts
--- hosts.orig	2017-09-22 18:08:15.345496162 +0300
+++ hosts	2017-09-22 18:09:17.662002725 +0300
@@ -1,9 +1,9 @@
-127.0.0.1	localhost
+127.0.0.1	localhost master.local
 127.0.1.1	machinename

 # The following lines are desirable for IPv6 capable hosts
 ::1     ip6-localhost ip6-loopback
 fe00::0 ip6-localnet
 ff00::0 ip6-mcastprefix
 ff02::1 ip6-allnodes
 ff02::2 ip6-allrouters
```

## Create SSH keys
```hadoop``` user walks by SSH to ```localhost``` and another machines
```
$ sudo -H su - hadoop
$ ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
$ chmod 0600 ~/.ssh/authorized_keys
$ ls -l .ssh/
-rw------- 1 hadoop hadoop  396 сен 18 22:19 authorized_keys
-rw------- 1 hadoop hadoop 1675 сен 18 22:19 id_rsa
-rw-r--r-- 1 hadoop hadoop  396 сен 18 22:19 id_rsa.pub
```
Put ```/home/hadoop/.ssh/authorized_keys``` to all cluster machines

## Configure local SSH options
```
$ echo "StrictHostKeyChecking no" >> /home/hadoop/.ssh/config
$ cat /home/hadoop/.ssh/config
StrictHostKeyChecking no
```

## Install openssh server (even if you have single machine), set server config options
```
sudo -H apt-get -s install openssh-server
```

SSH server config diff:
```
yamert@newship:/etc/ssh$ diff -U0 sshd_config.orig sshd_config
--- sshd_config.orig	2017-09-23 05:11:33.352922296 +0300
+++ sshd_config	2017-09-23 05:19:06.315729712 +0300
@@ -28 +28 @@
-PermitRootLogin prohibit-password
+PermitRootLogin no
@@ -42 +42 @@
-#IgnoreUserKnownHosts yes
+IgnoreUserKnownHosts yes
@@ -52 +52 @@
-#PasswordAuthentication yes
+PasswordAuthentication no
```

Restart sshd
```
sudo -H systemctl restart sshd
```

Check ssh access:
```
$ sudo -H su - hadoop
hadoop@machinename:~$ ssh localhost whoami
hadoop
hadoop@machinename:~$ ssh master.local whoami
hadoop
```
