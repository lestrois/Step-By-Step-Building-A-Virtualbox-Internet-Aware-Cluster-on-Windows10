##Motivatin:
     1) Building a lab cluster for team's development
     2) Too many virtual machine's LAN IP causes unstable of LAN router
     3) Easy to work from home

##Requirements:
    1) Building a 1 master, 4 slaves cluster by using VirtualBox.
     2) The system is built on Linux Ubuntu and CentOs virtual machines running on Windows 10.
     3) Only Master can be accessed by LAN/Internet computers. No Slaves can be accessed by LAN/Internet.
     4) Only Master has internet access
     5) Master can be ssh log in from Ineternet

##Software Preparations:
     1) Windows 10
     2) Virtualbox
     3) Linux Ubuntu or CentOS image/iso

###IP Address Plan:
     Master: centos-master: 192.168.0.150 (for LAN access by using Bridge-Adapter)
                  192.168.56.150 (for connecting slaves by using Host-Only)
     Slaves:      centos-slave1: 192.168.56.151
     centos-slave2: 192.168.56.152
     centos-slave3: 192.168.56.153
     centos-slave4: 192.168.56.154

###Installations:
     - Install Virtualbox on Windows 10
     - Create 5 Linux machines for above 5 names. It's better to set up SSH no password login from each other but not neccessary for finishing this plan.
     - Download Squid from  http://squid.acmeconsulting.it/download/dl-squid.html

###Configures:
#####On Virtualbox console, select centos-master then using menu Settings -> Network
     - enable Adapter1 then choose Attached to as Bridged Adapter
     - Enable Adapter2 then choose Host-only Adapter
          - At this step if the "Name" doesn't show option, go to Control Panel -> Network and Internet -> Network Connections to enable Virtualbox Host-Only Network
     - Enable Adapter3 then choose NAT
     - Click OK
     - Host-Only default gateway is 192.168.56.1. Check it on VB Console Menu File -> Preferences -> Network -> Host-Only Networkss. Do not need change it.
#####Starting centos-master VM then log in to configure the 3 network interfaces
     Now the interfaces for above 3 Adapters are eth2/eth3/eth4 in my case. it chould be eth0/eth1/eth2 in your case.
     1. Configure Bridge-Adapter to allow LAN computers' access
          Run "ifconfig -a" you would see the bridge-adapter's IP 192.168.0.X should be available which is assigned by your LAN router. Remember this interface name "ethX" (X is a number depending your machine). In my case it's eth2.
              - Make a static IP for it. Save the information in your /etc/network/interfaces (ubuntu) steps:
          sudo vi /etc/network/interfaces
          -Add following in content
          auto eth2
          iface eth2 inet static
          address 192.168.0.150
          netmask 255.255.255.0
          gateway 192.168.0.1
          -content done
          
          Make a static IP for it. Save the formation in /etc/sysconfig/network-scripts (centos) steps:
          
          sudo vi /etc/sysconfig/network-scripts/ifcfg-eth2
          -Following is the content
          DEVICE=eth2
          TYPE=Ethernet
          ONBOOT=yes
          NM_CONTROLLED=yes
          BOOTPROTO=static
          IPADDR=192.168.0.150
          NETMASK=255.255.255.0
          GATEWAY=192.168.0.1
          -Content done
          sudo ifdown eth3
          sudo ifup eth3

          - You should be able to ssh to centos-master's IP (192.168.0.150) by using putty.
     2. Configure Host-Only Adapter (eth3) to connect master/slaves each other.
          run "sudo ifconfig eth3 192.168.56.150 netmask 255.255.255.0 up" to enable it, then "ping 192.168.56.1", should work.
          - save the information in your /etc/network/interfaces (ubuntu) steps:
          sudo vi /etc/network/interfaces
          -Add following in content
          auto eth3
          iface eth3 inet static
          address 192.168.56.150
          netmask 255.255.255.0
          gateway 192.168.56.1
          -content done
          
          -Save the formation in /etc/sysconfig/network-scripts (centos) steps:
          sudo vi /etc/sysconfig/network-scripts/ifcfg-eth3
          -Following is the content
          DEVICE=eth3
          TYPE=Ethernet
          ONBOOT=yes
          NM_CONTROLLED=yes
          BOOTPROTO=static
          IPADDR=192.168.56.150
          NETMASK=255.255.255.0
          GATEWAY=192.168.56.1
          -Content done
          sudo ifdown eth3
          sudo ifup eth3
          
     
     3. Configure NAT Adapter (eth4) to enable internet access.
          - save the information in your /etc/network/interfaces (ubuntu) steps:
          sudo vi /etc/network/interfaces
          -Add following in content
          auto eth4
          iface eth3 inet dhcp
          -content done
          
          -Save the formation in /etc/sysconfig/network-scripts (centos) steps:
          sudo vi /etc/sysconfig/network-scripts/ifcfg-eth3
          -Following is the content
          DEVICE=eth4
          TYPE=Ethernet
          ONBOOT=yes
          NM_CONTROLLED=yes
          BOOTPROTO=dhcp
          -Content done
          sudo ifdown eth4
          sudo ifup eth4
          
          -Then "ping github.com" you should get internet access.
          
#####On Virtualbox console, select centos-slave1 then using menu Settings -> Network
     - Enable Adapter2 then choose Host-only Adapter
          - At this step if the "Name" doesn't show option, go to Control Panel -> Network and Internet -> Network Connections to enable Virtualbox Host-Only Network
         - Click OK
     - Starting centos-slave1 then log in. Find out the right interface name. In my case it's eth2
     - save the information in your /etc/network/interfaces (ubuntu) steps:
          sudo vi /etc/network/interfaces
          #Add following in content
          auto eth2
          iface eth2 inet static
          address 192.168.56.151
          netmask 255.255.255.0
          gateway 192.168.56.1
          #content done
     - Save the formation in /etc/sysconfig/network-scripts (centos) steps:
          sudo vi /etc/sysconfig/network-scripts/ifcfg-eth2
          #Following is the content
          DEVICE=eth2
          TYPE=Ethernet
          ONBOOT=yes
          NM_CONTROLLED=yes
          BOOTPROTO=static
          IPADDR=192.168.56.151
          NETMASK=255.255.255.0
          GATEWAY=192.168.56.1
          #Content done
          sudo ifdown eth2
          sudo ifup eth2
     
     - ping 192.168.56.150 it should work. No internet by checking "ping github.com".
#####Repeating above steps to finsh other 3 slaves.
#####On all master/slaves edit /etc/hosts to add the following.
     192.168.56.150 master centos-master
     192.168.56.151 slave1 centos-slave1
     192.168.56.152 slave2 centos-slave2
     192.168.56.153 slave3 centos-slave3
     192.168.56.154 slave4 centos-slave4
#####Connect From Internet
     - Now Let's Forward Master address to be accessed by Internet. Login your LAN Router (TPLink series) http://192.168.0.1
     - Go to Forwarding -> Virtual Servers Add SSH Port and click "OK" as following
          Service Port: 5822
          Internal Port: 22
          IP Address: 192.168.0.150
     - Get your Internet IP of your router, try putty to ssh XXX.XXX.XXX.XXX -p 5822 to verify you ssh it from internet

###Testing:
     Assume you have installed Hadoop/Spark on it
     [hadoop@centos-master ~]$ start-dfs.sh
     16/05/26 10:28:30 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
     Starting namenodes on [centos-master]
     centos-master: Warning: Permanently added the RSA host key for IP address '192.168.56.150' to the list of known hosts.
     centos-master: starting namenode, logging to /usr/local/hadoop/logs/hadoop-hadoop-namenode-centos-master.out
     yes
     centos-slave3: starting datanode, logging to /usr/local/hadoop/logs/hadoop-hadoop-datanode-centos-slave3.out
     centos-slave2: starting datanode, logging to /usr/local/hadoop/logs/hadoop-hadoop-datanode-centos-slave2.out
     centos-slave1: starting datanode, logging to /usr/local/hadoop/logs/hadoop-hadoop-datanode-centos-slave1.out
     centos-slave4: starting datanode, logging to /usr/local/hadoop/logs/hadoop-hadoop-datanode-centos-slave4.out
     Starting secondary namenodes [0.0.0.0]
     0.0.0.0: starting secondarynamenode, logging to /usr/local/hadoop/logs/hadoop-hadoop-secondarynamenode-centos-master.out
     16/05/26 10:28:50 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
     [hadoop@centos-master ~]$ jps
     5504 SecondaryNameNode
     5648 Jps
     5336 NameNode
     [hadoop@centos-master ~]$ slaves.sh jps
     centos-slave1: 4846 Jps
     centos-slave4: 4690 DataNode
     centos-slave3: 4761 Jps
     centos-slave1: 4768 DataNode
     centos-slave4: 4768 Jps
     centos-slave3: 4683 DataNode
     centos-slave2: 4762 Jps
     centos-slave2: 4684 DataNode
     [hadoop@centos-master ~]$ start-yarn.sh
     starting yarn daemons
     starting resourcemanager, logging to /usr/local/hadoop/logs/yarn-hadoop-resourcemanager-centos-master.out
     centos-slave1: starting nodemanager, logging to /usr/local/hadoop/logs/yarn-hadoop-nodemanager-centos-slave1.out
     centos-slave3: starting nodemanager, logging to /usr/local/hadoop/logs/yarn-hadoop-nodemanager-centos-slave3.out
     centos-slave4: starting nodemanager, logging to /usr/local/hadoop/logs/yarn-hadoop-nodemanager-centos-slave4.out
     centos-slave2: starting nodemanager, logging to /usr/local/hadoop/logs/yarn-hadoop-nodemanager-centos-slave2.out
     [hadoop@centos-master ~]$ slaves.sh jps
     centos-slave4: 4690 DataNode
     centos-slave2: 4812 NodeManager
     centos-slave4: 4818 NodeManager
     centos-slave1: 4896 NodeManager
     centos-slave3: 4811 NodeManager
     centos-slave4: 4854 Jps
     centos-slave2: 4848 Jps
     centos-slave3: 4683 DataNode
     centos-slave1: 4932 Jps
     centos-slave3: 4890 Jps
     centos-slave2: 4684 DataNode
     centos-slave1: 4768 DataNode
     [hadoop@centos-master ~]$ start-master.sh
     starting org.apache.spark.deploy.master.Master, logging to /usr/local/spark-1.6.1/logs/spark-hadoop-org.apache.spark.deploy.master.Master-1-centos-master.out
     [hadoop@centos-master ~]$ jps
     5504 SecondaryNameNode
     6085 Jps
     6031 Master
     5729 ResourceManager
     5336 NameNode
     [hadoop@centos-master ~]$ start-slaves.sh
     centos-slave1: starting org.apache.spark.deploy.worker.Worker, logging to /usr/local/spark-1.6.1/logs/spark-hadoop-org.apache.spark.deploy.worker.Worker-1-centos-slave1.out
     centos-slave4: starting org.apache.spark.deploy.worker.Worker, logging to /usr/local/spark-1.6.1/logs/spark-hadoop-org.apache.spark.deploy.worker.Worker-1-centos-slave4.out
     centos-slave2: starting org.apache.spark.deploy.worker.Worker, logging to /usr/local/spark-1.6.1/logs/spark-hadoop-org.apache.spark.deploy.worker.Worker-1-centos-slave2.out
     centos-slave3: starting org.apache.spark.deploy.worker.Worker, logging to /usr/local/spark-1.6.1/logs/spark-hadoop-org.apache.spark.deploy.worker.Worker-1-centos-slave3.out
     [hadoop@centos-master ~]$ slaves.sh jps
     centos-slave3: 4811 NodeManager
     centos-slave2: 5020 Jps
     centos-slave2: 4812 NodeManager
     centos-slave3: 5019 Jps
     centos-slave1: 5067 Worker
     centos-slave2: 4983 Worker
     centos-slave3: 4982 Worker
     centos-slave2: 4684 DataNode
     centos-slave3: 4683 DataNode
     centos-slave1: 5101 Jps
     centos-slave4: 4690 DataNode
     centos-slave1: 4896 NodeManager
     centos-slave1: 4768 DataNode
     centos-slave4: 4818 NodeManager
     centos-slave4: 4989 Worker
     centos-slave4: 5026 Jps
