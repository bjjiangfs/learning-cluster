__!!!NO USERNAME, PASSWORD HERE!!!__

* [Hadoop Cluster Requirements](#requirement)
* [Knowledge Background](#knowledge)
* [Notes About Hardware](#hd)
* [Steps to Follow](#step)
    - [Install Ubuntu](#install-ubuntu)
    - [Establsh Subnet](#install-subnet)
    - [Iptables](#iptables)
    - [Install Hadoop using Ambari](#install-hadoop)
    - [Test a MapReduce Program](#test-mapreduce)
* [Pitfalls](#pitfall)
* [How to Re-create the Cluster](#recreate-cluster)
* [Basic Network Troubleshooting](#troubleshoot)

# <a name="requirement">Hadoop Cluster Requirements</a>
- OS: CentOS 7/ Ubuntu 14.04
- Network Structure: NAT, `losalamos` need to be the NAT server. `losalamos` can be connected to the port on wall through "eno2".
- Install Choice: [Link to installation]( http://docs.hortonworks.com/HDPDocuments/Ambari-2.2.0.0/bk_Installing_HDP_AMB/bk_Installing_HDP_AMB-20151221.pdf)
- You need to install `HDFS`,`MapReduce2 + YARN`, `Ambari Metrics`, and `ZooKeeper` and you must install the package we are currently learning.
- You must keep a wiki of the necessary steps you think may be helpful to the next group here. Change of the wiki also is part of the grading.
- You have 3 whole days minus 2h for grading from 1:00 PM the first day to 11:00 AM the last day.


# Grading Criteria (30')

## Wiki (10')
- Beyond expectation, more than TA would write (12')
- Meet expectation (10')
- Basically meet expectation, missing some points that TA think is necessary for other groups (8')
- Below expectation (6')
- Less than 10 lines of wiki added/Modified, or modification makes no sense (2')

## Self designed demo with scripts to run without any intervention (10')
- Beyond expectation, test some points that TA would not think of (12')
- Meet expectation (10')
- Basically meet expectation (8')
- Below expectation, there are some essential points are not tested (6')
- Demo does not work will add another 3 points penalty on the previous grade.
- Demo requires TA intervention will add 1~2 points penalty on the previous grade depending on the time spend on intervention.

## Other (10')
- Iptables is up on losamalos and has basic protection with minimum iptables (2')
- Primary Name Node and Data Nodes should be on separate machines. (2')
- Primary Name Node and Secondary Name Nodes should be on separate machines. (2')
- NAT test, get google home page on other three machines. (2')
- Strong Password, password including at least a number and a letter and longer than 6 characters (2')
- No Alert in Ambari. (1' each alert, 2' max)

### Minimum Iptables
```
target     prot opt in     out     source               destination
ACCEPT     all  --  any    any     anywhere             anywhere             ctstate RELATED,ESTABLISHED
ACCEPT     all  --  lo     any     anywhere             anywhere
ACCEPT     icmp --  any    any     anywhere             anywhere
REJECT     all  --  any    any     anywhere             anywhere             reject-with icmp-host-prohibited
```

# <a name="knowledge">Knowledge Background</a>
- Ubuntu Command Line, Network Config (hostname, hosts, etc)
- Basic Computer Network knowledge, like DNS, Subnet (IP, Mask), Gateway;
- SSH, NAT, Forwarding
- Hadoop structure
- Basic understanding of above topics would make this work much easier

# <a name="hd">Things about hardware you should know</a>
- There are four servers to set up, but only the first one (Losalamos) has access to the Internet;
- The box connecting the servers is just a switch, not a router. So “forwarding” is needed to get the other three servers connected to the Internet;
- Every server has two network adapters, `eth0` and `eth1`, and it can only connects to the Internet by `eth1`. So please double-check the connection ports;
- Since `losalamos` uses `eth1` to connect to the Internet, it should use `eth0` for the sub network.
- Keep the roles of `eth0` and `eth1` in mind when you are configuring iptables with the linked tutorials: you may need to change the bash command given in those tutorials.

<hr>


# <a name="step">Steps to Follow</a>

Overview: Given four blank server, we need to install system and establish a subnet. Finally install the requested hortonwork components. The network should be built as this image

![image](network.png)

## <a name="install-ubuntu">Install Ubuntu</a>

Install Ubuntu (recommend 14.04) on each machine. The hard disks of four machines should already be erased. If not, press F11 when the system is starting and choose to start from the CD rom.

It may be hard to create a bootable USB stick on mac OS X. Failures occured for the following two approaches:
1. burn by command `dd` [[ref]](http://www.ubuntu.com/download/desktop/create-a-usb-stick-on-mac-osx)
2. burn by UNetbootin [[ref]](http://unetbootin.github.io/) Please update if there are methods that work. A convenient method is to install Ubuntu from CD (the CD is already provided, you can find it near the machines).

[Here](https://www.youtube.com/watch?v=P5lMuMhmd4Q) is a step-by-step installation video.

### Tips
1. In the image above, the three innet machines' hostname are `alpha`, `beta` and `gamma`. You can change them to whatever you like.

2. During the installation, we need configured network of `losalamos` with eth1 and we don't need to configure the network of three innet machines during the install process. Thus when installing Ubuntu on the three innet machines, you can either chose eth0 or eth1 during network configuration step, and it will eventually show "network autoconfiguration failed", just ignore and continue.

3. You probably want to install the OpenSSH during installation, so that you can then connect to the server using terminal in your own laptops.

4. `losamalos` should have access to the internet already after installation. Using `ping google.com` or `ping + other known IP address` to check the connection.

5. You need to choose unmount the disk partition before installation step. Choose the guided use entire disk, if there is multiple partition selections, just take the default one.

6. Sometimes the system may get stuck when reboots after completing installation, in such rare cases, just press the reboot button on the back of the server for more than 10 seconds and restart the system.

Notice: the openssh-server should be installed on all of the four machines for ssh to function properly, try `apt-get update` before install openssh-server. 

## <a name="install-subnet">Establish Subnet</a>

Notice: during the entire process (even after you finish this part), you’d better not reboot any of the four machines after you have done with following establish subnet steps, otherwise you may lose your network connection and need to install the OS once again (Welcome for the notes if you could solve this problems without reinstalling OS).

### Tips

1. Connect servers physically, through the switch and network adapter ports on each machine. Usually this step has already been done.
2. Start from the `losamalos` Up the `eth0` network of `losalamos`. using command `sudo ifconfig eth0 up`
3. Configure `eth0` in the file `/etc/network/interfaces` with `static ip = 10.0.0.2`, `netmask 255.255.255.0`, `gateway 10.0.0.2`, and `broadcast 10.0.0.255`. You can find an example [here](https://wiki.debian.org/NetworkConfiguration), in the **Configuring the interface manually** section. Since this file is read-only, you may want to edit it with sudo.
4. There are two ways to setup connection between `losamalos` and the other threes machine `alpha`, `beta` and `gamma`. （static IP is easier and safer）
5. To prevent warning for Ambari part, you can set the hosts as 'ip_address domain_name alias', each node should maintain the same copies of hosts configuration file.
6. If you want to set the hosts as 'ip_address domain_name alias'. In the file /etc/hosts, you should list all the hosts below the localhost on each machine.  It should be  10.0.0.2 losalamos.pc.cs.cmu.edu losalamos, 10.0.0.3 alpha.pc.cs.cmu.edu alpha 10.0.0.4 beta.pc.cs.cmu.edu beta and 10.0.0.5 gamma.pc.cs.cmu.edu gamma.(Each host one line.).


* Using DHCP

> * Set up a DHCP server on `losalamos` first. [Here](https://www.youtube.com/watch?v=9Vc6-0smd64) is a video tutorial about how to set up a DHCP server on Ubuntu server. Be carefule about compatability. The system we install is Ubuntu 14.01. So download the version of DHCP server which is compatable with our system. The DNS server of CMU are [here](https://www.cmu.edu/computing/partners/dept-computing/services/domain.html) And you can check [this](http://askubuntu.com/questions/140126/how-do-i-install-and-configure-a-dhcp-server) for DHCP configuration.

> * Here is some quick tips for setting up the dhcp server. After installed dhcp in `losalamos`, you need to configure it in file `/etc/dhcp/dhcpd.conf`. In this file, you need to configure an internal subnet with `subnet`, `netmask`, `range`, `domain-name-servers`, `default-lease-time` and `max-lease-time`. You can configure the other parameters, but the stuff above is considered necessary to let your dhcp server work. After configuration, run `/etc/init.d/isc-dhcp-server restart` to restart your dhcp server. As always, please use sudo.

> * Switch to innet machines, up the `eth1`, and set up each `eth1` to `dhcp`. You can check this [page](http://inside.mines.edu/CCIT-NET-SS-Configuring-a-Dynamic-IP-Address-Debian-Linux) to help.

* Using staic IP

> * No need to set up DHCP server on `losalamos`. Go straight to innet machines and set up the static IP to the three innet machine as the image above. This [page](https://help.ubuntu.com/14.04/serverguide/network-configuration.html) can help you to set up the static ip, you need to set the `address`(staic ip),`netmask`(255.255.255.0),`gateway`(the static IP of losamalos) and`dns-nameservers`(128.2.184.224) in the file `/etc/network/interfaces`

5. For slaves machine, after making the configurations above, remember the configurations will take effect only after 1) you reboot the machine **OR** 2) shut down port using `sudo ifdown eth1` and then restart using `sudo ifup eth1`. Though the command may return error information, it actually works. 
6. **DO NOT** reboot losalamos after configuration. Simply using `sudo ifdown eth0`, `sudo ifup eth0` and `sudo ifconfig eth0 up` to enable the configuration (Not eth1 for losalamos! And if it returns error information after executing second command, you can ignore it as long as the third command can be executed successfully). Otherwise you may lose your connection to external network. 
7. You should be able to ping each other now using IP.
8. Edit `/etc/hosts` files on four machines, telling them the connections between IP and domain and hostname. This [page](http://linux.die.net/man/5/hosts) can guide you how to set up. In losalamos `/etc/hosts`, remember to change the default ip address of losalamos to 10.0.0.2.
9. You should be able to ping each other now using domain or hostname.

## <a name="iptables">Iptables</a>

![iptables](http://www.system-rescue-cd.org/images/dport-routing-02.png)

For now, the machines in the subnet are unable to connect the real internet. This is because the gateway does not forward their tcp/udp requests to the outside world. Thus we use `iptables` to tell gateway forwarding them. [This page](http://www.revsys.com/writings/quicktips/nat.html) is enough as a HOWTO wiki. If you want to know more about forwarding, check [this](http://www.howtogeek.com/177621/the-beginners-guide-to-iptables-the-linux-firewall/). After configuring iptables, all four machines should be able to connect to the Internet now, you can try to ping www.google.com on all four machines to test your configuration.

You may want to confiture the iptables to block some incoming traffic and allow access only to particular protocols and ports. [Here](http://developer-should-know.tumblr.com/post/128018906292/minimal-iptables-configuration) is a quick introduction. Use `iptables -L -v` to check current valid rules. In case you wronly add a certain rule, use `iptable -D [rules]` to delete a cerain rules, check [this](https://www.digitalocean.com/community/tutorials/how-to-list-and-delete-iptables-firewall-rules) for reference. 	

### Tips

1. Read the instructions **carefully** and find out **which is incoming network port and which is outgoing**. **Do not** use the command line exactly the same as in [this](http://www.revsys.com/writings/quicktips/nat.html). You should modify the eth0 or eth1 based on the picture above. First you should postrouting eth1, then you should forward eth1 to eth0 and last you should forward eth0 to eht1.
2. When executing `echo 1 > /proc/sys/net/ipv4/ip_forward` as instructed in the HOWTO wiki page, if get a "permission denied" alert，please use this command: 
`sudo bash -c 'echo 1 > /proc/sys/net/ipv4/ip_forward'`.
3. Don't overthink it. Just type in the commands, they are not script.
4. If you cannot ping external resources on the inner machines, you can: 1) check if your server is able to ping outside or not; 2) check if the `dns-nameservers` is set in all four configuration files; or 3) check carefully the spelling of your configuration files. 4) check `/etc/sysctl.conf` is well modified.
5. When setting the iptable protection, make sure you don't block the SSH.
6. When you are setting the iptable protection, if you want to set the REJECT all -- any any anywhere anywhere reject-with icmp-host-prohibited, make sure that you should first accpet the port 22 and port 8080, or you may lose the ssh.


## <a name="install-hadoop">Install Hadoop using Ambari</a>

Ambari is a automatical deploy system for Hadoop. [Link to installation]( http://docs.hortonworks.com/HDPDocuments/Ambari-2.2.0.0/bk_Installing_HDP_AMB/bk_Installing_HDP_AMB-20151221.pdf).

For setup, configure and deploy parts, you may also refer to [This](http://blog.phaisarn.com/node/1391) and [This](https://hadoopjournal.wordpress.com/2015/08/09/hortonworks-hadoop-installation-using-apache-ambari-on-centos6/).

### Tips
* Go through the “Getting Ready” section to check and configure if you could meet with the basic environment requirements. Take care of part 1.4.
* It's better to follow the Official installation document. Link has been given above. But for the password-less SSH setting, the links behind in the tips are more detailed(although basically they are the same), you may get puzzled follow the official document.
* ##Do not## skip the 1.4 “Prepare the Environment” for the sake of less possible problems in the later installation process:
1. Do 1.4.1 Set Up Password-less SSH use links behind in the tips
2. no need do 1.4.2: there is default account
3. Do 1.4.3 NTP on all four hosts, there is no ubuntu version command in the official installation document, refer to [here](http://blogging.dragon.org.uk/setting-up-ntp-on-ubuntu-14-04/) and [here](http://blogging.dragon.org.uk/setting-up-ntp-on-ubuntu-14-04/)
4. no need do 1.4.4: Offitial installation document gives hosts name and network setting on redhat and centOS. for ubuntu, hostname and network are set in etc/network/interfaces already in the "Establish Subnet" process。
5. no need for 1.4.5: detailed iptable setting guide has been given above.
6. Do 1.4.6 Ubuntu 14 has no selinux pre-installed. Follow the instruction to set umask.
7. You can set ulimit at /etc/security/limits.conf, make sure you change the ulimit of the ACCOUNT YOU USE(e.g root) to install Ambari. **Do not** reboot the system when you finish the ulimit installation. If do, you may need to reinstall the machine.

* [This](http://posidev.com/blog/2009/06/04/set-ulimit-parameters-on-ubuntu/) will help you when setting `ulimit`. Notice that in this instruction, `user` means `[user]`. Thus you need to replace it with your system username.
* Set up the SSH carefully. After this part being done, you can remotely control those four machines with your own laptop. If you did not install OpenSSH during installation, you can install it using `apt-get install openssh-server`. You can only directly SSH into `losalamos` from the outside, but you can SSH into other machines within `losalamos` (like Inception!).
* You need to set up password-less SSH during the process:
	- Overview for password-less SSH: produce a pair of public key and private key on one host, copy the public key to other hosts, then you could visit those hosts without inputting password. It's like give away your public key to others, you have the access to them.
	- The goal is that you can ssh from any one of the four machines to the root of other three without typing in password manually.
	- One way to achieve password-less SSH is that: for each node, login as root user by su and put the same copy of rsa key pair in the /.ssh directory of root user account. 
	- Ubuntu system has no pre-set password for root user, in order to login as root user, you need to set password first, use command -'sudo passwd'
	- The manual from Hortonworks have covered the basic steps. You can also check [this](http://www.linuxproblem.org/art_9.html) and [this](http://askubuntu.com/questions/497895/permission-denied-for-rootlocalhost-for-ssh-connection) if you need more help (However, be careful that you should still use `ssh-keygen` while generating key pairs, otherwise it could not ssh the root properly later).
	- You need to use root permission to set up password-less SSH. To set the root password see [this](http://askubuntu.com/questions/155278/how-do-i-set-the-root-password-so-i-can-use-su-instead-of-sudo).
	- If you change the ssh configuration, you may need to restart ssh by `service ssh restart`.
	- Make sure the password-less SSH works in both directions among four machines: scp the private and public key to the .ssh folder of four machines and modify authorized_key file.
	- If something goes wrong with the password-less SSH, you may get timeout error in building cluster. Then try Installing Ambari Agents Manually, look at [this](https://ambari.apache.org/1.2.0/installing-hadoop-using-ambari/content/ambari-chap6-1.html). For Ubuntu, use apt-get instead of yum.
	- You may generate the public key or private key from the user account which is not root, check it carefully or you may not be able to automatically install the hadoop system. The public key and private key is under the file /root/.ssh. .ssh file is invisible file there.
	- When input the private key in the Ambari installation, don't forget to include the first line and last line.
	
* If you come accross failure in registering four machines, check:
	- If you set the ssh correctly, and can login in other machine from root@losalamos without password.
	- Use the private key: `id_rsa`. Copy this with `scp` to your laptop beforehand. You could use this [link](http://www.hypexr.org/linux_scp_help.php) for reference.
	- All machine, /etc/hosts need to have their FQDN inside. Also, according to Install Documentation, check `hostname -f` is return its FQDN.
* Before Install the services, better to carefully handle the warning from the registeration section. Check whether NTP is intalled.If you meet warings when confirms hosts which said ntp services error, you may check whether you have already started up the ntp on each machine, if not, use this command line 'sudo service ntp start'.
* The services you need to install are `HDFS`, `MapReduce2`, `Yarn`, `ZooKeeper` and  `Ambari Metrics`. Some other services may fail so do not install services that you do not need.
* You need to install both `ambari-server` and `ambari-agent` on `losamalos`, and you only need to install `ambari-agent` on three innet machine,
* But if everything goes smoothly, you only have to manully install `ambari-server` on `losalamos`, and everything else can be doen through the [Ambari Web](http://losalamos.pc.cs.cmu.edu:8080) in web browser.
* While installing `ambari-server` on `losamalos`, java 1.8 will be installed with your choice during the process, but you need to configure the environment variables by yourself this [page](http://stackoverflow.com/questions/9612941/how-to-set-java-environment-path-in-ubuntu) will help on your configurations.
* Your java directory should be under `/usr/jdk64/`. Carefully set it to your configuration file.
* While going through the Ambari Install Wizard, there are several parts you should watch out: 
	- Make sure password-less ssh is correctly set up, which will let you ssh from any one of the four machines to other three without typing in password manually. Otherwise if may gave you failure when registering three inner machines.
	- When choosing services to install, only choose those are required. One safe way to do this is to first install only `HDFS`, `MapReduce2`, `Yarn`, `ZooKeeper` and  `Ambari Metrics`. And go back to install other required services after confirming your hadoop can run correctly by runing a MapReduce task.
	- When assinging master, go through the `Grading Criteria` in `Requirements` section carefully.
	- When install extra service, you should not omit the warning. You need to handle it one by one.
	- Restart the service before runing Demo
* You should be aware of that `losalamos` should be one of the clients since it is the only interface to run Hadoop programs from outside.
* Select default setting when installing Ambari Server.
* If you come across errors when starting the server, Check [ this](https://community.hortonworks.com/articles/16944/warning-setpgid31734-0-failed-errno-13-permission.html).
* Once the cluster is installed, make sure [this page](http://losalamos.pc.cs.cmu.edu:8080/#/main/hosts) shows each host has correct IP address (10.0.0.x).s If the IP address is 127.0.0.1 that's not correct, check whether the four `/etc/hosts` files are same with each other. Modify `/etc/hosts` if necessary, then restart both ambari-server and all ambari-clients.
* If something goes wrong, check your firewall settings or you may find causes by looking at log files under `/var/log`
* If run into Transparent Huge Pages error, check out [this](https://docs.mongodb.org/manual/tutorial/transparent-huge-pages/).
[this](https://access.redhat.com/solutions/46111).
* For installing Ambari (except for logging into node for debug), you DON'T need to install python2.6, Ambari is compatible with python2.6 or later version.
* If you decide to install python yourself, actually for anything, DO NOT use any personal repository, use official ones. Otherwise it may lead to cluster building failure and probably reinstallation of OS.

## <a name="test-mapreduce">Test a MapReduce Program</a>

If everything is green on the dashboard of Ambari, you can follow [this](http://www.joshuaburkholder.com/blog/2014/05/15/how-to-run-ava-mrv2-using-hadoop/) to run a mapreduce job on the machines.

### Steps

1. Create a input directory under the user of `hdfs`
2. Write the test MapReduce program (eg. wordcount)
3. Compile the java files to class files with `javac` and archive the class files into `jar`
4. Use command `yarn` to run the project and remember to set the output directory of your project or you will hard to find it
5. Run the program under the user `hdfs` (HADOOP_USER_NAME=hdfs).
6. If you want to move the files to HDFS via Ambari UI, you could follow the steps mentioned [here](https://developer.ibm.com/hadoop/blog/2015/10/22/browse-hdfs-via-ambari-files-view/). Also, it is better to create a separate user 'hdfs' instead of 'admin' in Ambari if you follow this approach and give it root permissions in Ambari.

### Tips

- If you meet any permission problem of `hdfs`, check [this](http://stackoverflow.com/a/20002264/2580825)
- Log in through SSH to `losalamos` and perform all you tests here since this server should be the only interface;
- Switch to other Hadoop users (ex. hdfs, but you can still create a new one) and upload or create your files on HDFS;
- The output folder of your map reduce program should not exist when executing the jar program.
- If there's any "permission" problem, try using su (root), or `sudo` in each command;
- Remember that in MapReduce 2.0, you should use the command `yarn` but not `hadoop`.
- If you have trouble running your wordcount program, you may need to install the Java Jre before. You can choose the default one. 
- If you have already run the wordcount program successfully and want to run it again, make sure to remove two things. The first one is the output folder. Using hdfs 'dfs -rm -r StartsWithCount/output'. And anther one is the previous version's result. Or you may meet problems say 'File exits'.

# <a name="pitfall">Pitfalls you should pay attention to</a>
- Make sure the physical connection is correct;
- You should down/up network adapters or reboot machines to make your network configurations work;
- Make sure your configurations are permanent, otherwise they will remain unchanged after reboot, like iptables;
- Ambari Server should be installed on `losalamos` since it is the only server you can get access to from outside the subnet;
`losalamos` should also hold a Ambari Agent to be part of the cluster;
- Keep in mind that `losalamos` should be one of the clients;
- Make sure you use `ulimit` to change file descriptors limit before installing Ambari, or you may encounter problems in running the cluster.
- If by any chance you mapped the History Server incorrectly, you can change it using the steps given [here](https://cwiki.apache.org/confluence/display/AMBARI/Move+Mapreduce2+History+Server) instead of re-doing everything.
- **Do not** reboot losalamos after installing the OS. If you reboot the losalamos after install the ulimit and lose ssh and net connection, you don't need to restart all the machine. Just the losalamos. Install it from the beginning.
- Edit this instruction file with carefulness, wrong tips can lead to a huge waste of time of other people.

# <a name="recreate-cluster">How to Re-create the Cluster</a>
In case anything you configured wrong, you might want to rebuild the cluster again. Please follow the below steps.

##(Two groups have indicated that the following 5 steps may cause components of ambari not being able to install and the ambari to fail rebooting, so be careful if you need to reconfigure the server)
1. Stop all services from Ambari first, both in losalamos and 3 slave machines. On slave machines, `sudo ambari-client stop`. On losalamos, do `sudo ambari-client stop` and `sudo ambari-server stop`.
2. Clean installed services on all four machines
`python /usr/lib/python2.6/site-packages/ambari_agent/HostCleanup.py`
3. Stop Ambari Server `sudo ambari-server stop`
3. Reset Ambari Server `sudo ambari-server reset`
4. Start Ambari Server again `sudo ambari-server start`
5. Login to Ambari webpage and create the cluster


If you cann't create iptables by following the steps above, you can refer to this script created by Hsueh-Hung Cheng [Here](https://gist.github.com/xuehung/8859e7162466918aac82), make sure you understand each line of script (it may not work). When you make use of this script, if there is permission denied alert, try to add `sudo` at the head of most of the lines and refer to the tips in Iptables above to modify the rest one.

# <a name="trubleshoot">Basic Network Troubleshooting</a>
## Troubleshooting Checklist

* Is the interface configured correctly? (Related command or files: ifconfig, /etc/network/interfaces, lspci, lsmod, dmesg)
* Is DNS/hostnames configured correctly? (Related command or files: /etc/hosts, /etc/resolv.conf, bind)
* Are the ARP tables correct? (arp -a)
* Can you ping the localhost? (ping localhost/127.0.0.1)
* Can you ping other local hosts (hosts on the local network) by IP address? How about hostname? (Related command: ping)
* Can you ping hosts on another network (Internet)? (Related command: ping)
All your are doing is going either up or down the network model layers.

## Explanations about several useful command

* `route -n`: To see your routing tables. `-n` means return numeric output
* `ping`: Ping your computer (by address, not host name) to determine that TCP/IP is functioning. You can also use option `-c` to determine how many packets you'are sending.
* `ifconfig`: Tell you everything about the network interface
* `iptables -L -v` Check current valid rule in iptable
