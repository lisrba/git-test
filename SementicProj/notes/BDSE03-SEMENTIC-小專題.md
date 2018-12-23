# BDSE03-SEMENTIC-小專題

## TODO

* 建置n台linux系統，每台要安裝hadoop, java等軟體，並且先建立一個sudouser帳號，可以連線到每一台電腦(方便用這個帳號用ssh協定連線到每一台去做設定)

#### IP, HOSTNAME

每人管2台，4GB RAM/台

HOSTNAME: semantic.example.org

翁：
192.168.33.221  semantic01   #NameNode(standby) #DataNode #QJM #Zookeeper
192.168.33.222  semantic02   #ResourceManager #DataNode
PEI
192.168.33.121  semantic03   #NameNode #QJM #Zookeeper
192.168.33.122  semantic04   #DataNode
桓:
192.168.33.161  semantic05   #DataNode #QJM #Zookeeper #ResourceManager(standby)
192.168.33.162  semantic06   #DataNode
曹:
192.168.33.201  semantic07   #DataNode
192.168.33.202  semantic08   #DataNode #Ganglia Collector
易：
192.168.33.81   semantic09   #DataNode #Ganglia Server
192.168.33.82   semantic10   #DataNode
隋:
192.168.33.131  semantic11   #DataNode
192.168.33.132  semantic12   #JobHistoryServer, #DataNode

#### 刪除之前上課Virtualbox裡的六個虛擬機器以節省C槽硬碟空間
```
C:\Users\Student>cd hadoop_cluster

C:\Users\Student\hadoop_cluster>vagrant destroy
    slave5: Are you sure you want to destroy the 'slave5' VM? [y/N] y
==> slave5: Destroying VM and associated drives...
    slave4: Are you sure you want to destroy the 'slave4' VM? [y/N] y
==> slave4: Discarding saved state of VM...
==> slave4: Destroying VM and associated drives...
    slave3: Are you sure you want to destroy the 'slave3' VM? [y/N] y
==> slave3: Destroying VM and associated drives...
    slave2: Are you sure you want to destroy the 'slave2' VM? [y/N] y
==> slave2: Destroying VM and associated drives...
    slave1: Are you sure you want to destroy the 'slave1' VM? [y/N] y
==> slave1: Destroying VM and associated drives...
    master: Are you sure you want to destroy the 'master' VM? [y/N] y
==> master: Destroying VM and associated drives...
```

#### Vagrant: git bash

```bash
vagrant box add ubuntu/trusty64  #新增映像檔 ubuntu/trusty64 至Vagrant
vagrant init ubuntu/trusty64     #用該映像檔開機
vagrant ssh                      #連線至該映像檔
``` 

#### 移除原本hadoop_node，加入老師的box後，確認
```
vagrant box remove iiiedu/hadoop_node
vagrant box add --name iiiedu/hadoop_node 路徑/hadoop_node.box
vagrant box list 
```

#### 建hadoop_classroom目錄，複製老師的Vagrantfile_classroom過來，更名為Vagrantfile
```
C:\Users\Student\hadoop_classroom
```

#### 修改Vagrantfile：設定兩台虛擬機，名稱semantic01和semantic02，修改memory至4096

```
  config.vm.define "semantic01" do |semantic01| 
    semantic01.vm.hostname = "semantic01" 
	semantic01.vm.network :public_network, ip: "192.168.33.221"
    semantic01.vm.provider "virtualbox" do |v| 
      v.name = "semantic01" 
      v.cpus = 1
      v.memory = 4096 
    end
  end 
  
  config.vm.define "semantic02" do |semantic02| 
    semantic02.vm.hostname = "semantic02" 
	semantic02.vm.network :public_network, ip: "192.168.33.222"
    semantic02.vm.provider "virtualbox" do |v| 
      v.name = "semantic02" 
      v.cpus = 1
      v.memory = 4096 
    end
  end 
```

#### 啟動虛擬機：用Vagrantfile Line15的hadoop_node產生兩台虛擬機
```
cd hadoop_classroom
vagrant up
```
#### 確認虛擬機狀態：在PC ping VM的IP，登入VM(不用密碼)
```
ping 192.168.33.XX
vagrant ssh semanticXX 
```
#### 複製老師的hosts和sethosts.sh到C:\Users\Student\hadoop_classroom\scripts，修改hosts
```
(IP              FQDN(完整網域名稱)        alias)
127.0.0.1       localhost
192.168.33.221  semantic01.example.org  semantic01
192.168.33.222  semantic02.example.org  semantic02
192.168.33.121  semantic03.example.org  semantic03
192.168.33.122  semantic04.example.org  semantic04
192.168.33.161  semantic05.example.org  semantic05
192.168.33.162  semantic06.example.org  semantic06
192.168.33.201  semantic07.example.org  semantic07
192.168.33.202  semantic08.example.org  semantic08
192.168.33.81   semantic09.example.org  semantic09
192.168.33.82   semantic10.example.org  semantic10
192.168.33.131  semantic11.example.org  semantic11
192.168.33.132  semantic12.example.org  semantic12
```
#### 在Vagrantfile設定每次重新開機自動啟動 sethosts.sh，/vagrant/scripts/hosts蓋掉原本/etc/hosts
```
    end
  semantic03.vm.provision "shell", path: "scripts/sethosts.sh", run: "always"
end
```
#### 重新啟動帶入，進入semanticXX確認hosts有看到所有VM的IP名稱
```
vagrant reload
vagrant ssh semanticXX
> cat /etc/hosts
> ping -c 4 semanticXX(.example.org)
```
翁大秘技：ping所有機器，看是否所有機器都上線
```bash
 for j in 01 02 03 04 05 06 07 08 09 10 11 12; do ping -c 4 semantic$j.example.org; done
```

#### 進入虛擬機確認是否有hadoop這個帳號，以及Java是否安裝好
```
> cat /etc/passwd | grep 'hadoop'
> java -version
> javac -version
> update-alternatives --display java
```

#### 測試是否所有人都有$JAVA_HOME, $HADOOP_HOME環境變數(有問題，待修正)
```bash
for j in 01 02 03 04 05 06 07 08 09 10 11 12; do ssh semantic$j echo $JAVA_HOME;echo $HADOOP_HOME; done
```

若無：
```bash
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update && sudo apt-get install oracle-java7-set-default
```

```bash
adduser hadoop            #新增使用者(名稱：hadoop)
gpasswd -a hadoop sudo    #將hadoop變成sudo user
```

#### 下載hadoop-2.7.3.tar.gz，放進hadoop_classroom，確認
```
> cd
> wget http://apache.stu.edu.tw/hadoop/common/hadoop-2.7.3/hadoop-2.7.3.tar.gz 
> df -h
> ll /vagrant
```
#### 至每台機器解壓縮，更名為hadoop，修改權限，確認目錄及裡面檔案的權限
```
sudo tar -xvf /vagrant/hadoop-2.7.3.tar.gz -C /usr/local/
sudo mv /usr/local/hadoop-2.7.3 /usr/local/hadoop
ls /usr/local/
sudo chown -R hadoop:hadoop /usr/local/hadoop  #(把使用者從root改為hadoop)
df -h  #(可看掛載的檔)
ls -d /usr/local
cd /usr/local/hadoop
ll
```

#### 確認是否有公鑰私鑰，能否無密碼登入其他台(01~12)虛擬機

```
su - hadoop
ls -l ~/.ssh 
ssh semanticXX.example.org

```
#### 設定環境變數，將環境變數貼到.bashrc檔案最下面
```
nano ~/.bashrc

# Set HADOOP_HOME
export HADOOP_HOME=/usr/local/hadoop
# Set JAVA_HOME 
export JAVA_HOME=/usr/lib/jvm/java-8-oracle
# Add Hadoop bin and sbin directory to PATH
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin

將環境變數導入
source ~/.bashrc
env
```

#### 進入hadoop目錄內
```
cd /usr/local/hadoop/etc/hadoop/
```

#### 更新hadoop-env.sh檔案內容，將25跟33行的部分更新(因為預設的方法會抓不到)

```
nano /usr/local/hadoop/etc/hadoop/hadoop-env.sh
export JAVA_HOME=/usr/lib/jvm/java-8-oracle
export HADOOP_CONF_DIR=/usr/local/hadoop/etc/hadoop
```
#### 設定core-site.xml檔案 
```
nano /usr/local/hadoop/etc/hadoop/core-site.xml 

將以下貼到<configuration></configuration>中間 PS去掉#註記
	<property>
       <name>hadoop.tmp.dir</name>
       <value>/home/hadoop/tmp</value>
       <description>Temporary Directory.</description>
    </property>

    <property>
       <name>fs.defaultFS</name>
       <value>hdfs://semantic03.example.org</value>	
        #PORT:54310(hadoop1.X) 8020(hadoop2.X預設) 9000(hadoop2.X 流行)
       <description>Use HDFS as file storage engine</description>
    </property>
```
#### 複製mapred-site.xml.template(範本)為mapred-site.xml，設定
定義
(1) hadoop使用yarn做工作分配
(2) 指定jobhistory server存放執行過的工作
```
cp /usr/local/hadoop/etc/hadoop/mapred-site.xml.template
       /usr/local/hadoop/etc/hadoop/mapred-site.xml
	
nano /usr/local/hadoop/etc/hadoop/mapred-site.xml

貼上
<property>
       <name>mapreduce.framework.name</name>
       <value>yarn</value>
</property>
<property>
	   <name>mapreduce.jobhistory.address</name>
	   <value>semantic12.example.org:10020</value>
</property>
<property>
	   <name>mapreduce.jobhistory.webapp.address</name>
	   <value>semantic12.example.org:19888</value>
</property>
```

#### 設定yarn-site.xml
定義
(1) ResourceManager的機器
(2) NodeManager資源使用情況
```
nano /usr/local/hadoop/etc/hadoop/yarn-site.xml

    <property>
       <name>yarn.nodemanager.aux-services</name>
       <value>mapreduce_shuffle</value>
    </property>
	<property>
       <name>yarn.nodemanager.resource.memory-mb</name>
       <value>4096</value> 
    </property>
    <property>
       <name>yarn.nodemanager.resource.cpu-vcores</name>
       <value>1</value>
    </property>
    <property>
	   <name>yarn.resourcemanager.hostname</name>
	   <value>semantic02.example.org</value>
    </property>	
```

#### 設定slaves：定義DataNode(NodeManager)[只有NameNode的機器不要設為slave，其他機器都可以設為slave去利用資源]

```
semantic01.example.org
semantic02.example.org
semantic04.example.org
semantic05.example.org
semantic06.example.org
semantic07.example.org
semantic08.example.org
semantic09.example.org
semantic10.example.org
semantic11.example.org
semantic12.example.org
```

### 順序：

```
1.先起hdfs
2.再起yarn
3.啟動History Server

1.先停History Server
2.再停yarn
3.停用hdfs
```

#### 失敗經驗：

````
原因:
原設為NameNode的semantic01，誤ssh到預定為DataNode的semantic03做了格式化以及啟動　hdfs，導致DataNode與SecondaryNameNode雖然能成功啟動，但NameNode消失了

處置1：
回到semantic01再格式化以及啟動一次，結果導致有兩台SecondaryNameNode，沒有NameNode

處置2：砍掉重架
````



#### NameNode：格式化後重啟，jps確認
```
hdfs namenode -format
start-dfs.sh
stop-dfs.sh
start-dfs.sh
jps 
```

#### ResourceManager： 
```
start-yarn.sh
jps
```

#### 從browser看NameNode ResourceManager狀態：

```
C:\Windows\System32\drivers\etc 
編輯hosts，加入：
127.0.0.1 localhost
192.168.33.221 semantic01.example.org semantic01
192.168.33.222 semantic02.example.org semantic02
192.168.33.121 semantic03.example.org semantic03
192.168.33.122 semantic04.example.org semantic04
192.168.33.161 semantic05.example.org semantic05
192.168.33.162 semantic06.example.org semantic06
192.168.33.201 semantic07.example.org semantic07
192.168.33.202 semantic08.example.org semantic08
192.168.33.81 semantic09.example.org semantic09
192.168.33.82 semantic10.example.org semantic10
192.168.33.131 semantic11.example.org semantic11
192.168.33.132 semantic12.example.org semantic12

http://semantic03.example.org:50070/ (NameNode)
http://semantic02.example.org:8088/ (ResourceManager)

查port number
lsof -nPi

```
#### jobhistory server啟動
```	
    mr-jobhistory-daemon.sh start historyserver
    jps
    lsof -nPi
```
#### 測試mapreduce
```
hadoop jar /usr/local/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.3.jar pi 30 100
```

#### 設定hdfs-site.html

<property>
     <name>dfs.nameservices</name>
     <value>hadoop-ha</value>
   </property>
   <property>
     <name>dfs.ha.namenodes.hadoop-ha</name>
     <value>semantic03,semantic01</value>
   </property>
   <property>
     <name>dfs.namenode.rpc-address.hadoop-ha.semantic03</name>
     <value>semantic03.example.org:8020</value>
   </property>
   <property>
     <name>dfs.namenode.http-address.hadoop-ha.semantic03</name>
     <value>semantic03.example.org:50070</value>
   </property>
   <property>
     <name>dfs.namenode.rpc-address.hadoop-ha.semantic01</name>
     <value>semantic01.example.org:8020</value>
   </property>
   <property>
     <name>dfs.namenode.http-address.hadoop-ha.semantic01</name>
     <value>semantic01.example.org:50070</value>
   </property>
   <property>
     <name>dfs.namenode.shared.edits.dir</name>
     <value>qjournal://semantic03.example.org:8485;semantic01.example.org:8485;semantic05.example.org:8485/hadoop-ha</value>
   </property>
   <property>
     <name>dfs.journalnode.edits.dir</name>
     <value>/home/hadoop/journalnode</value>
   </property>
   <property>
     <name>dfs.ha.fencing.methods</name>
     <value>sshfence</value>
   </property>
   <property>
     <name>dfs.ha.fencing.ssh.private-key-files</name>
     <value>/home/hadoop/.ssh/id_rsa</value>
   </property>
    <property>
     <name>dfs.client.failover.proxy.provider.hadoop-ha</name>
     <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
   </property>


#### 由semantic03散佈設定檔至其他電腦
```bash
#!/bin/bash

coresite=/usr/local/hadoop/etc/hadoop/core-site.xml
yarnsite=/usr/local/hadoop/etc/hadoop/yarn-site.xml
mpdsite=/usr/local/hadoop/etc/hadoop/mapred-site.xml
slaves=/usr/local/hadoop/etc/hadoop/slaves

for j in 01 02 04 05 06 07 08 09 10 11 12
do
  scp $coresite semantic$j:$coresite
  scp $yarnsite semantic$j:$yarnsite
  scp $mpdsite semantic$j:$mpdsite
  scp $slaves semantic$j:$slaves
done
```



 測試mapreduce
 

確認可以連線到所有機器(機器地址是FQDN)
```bash
#!/bin/bash

for j in 01 02 03 04 05 06 07 08 09 10 11 12
do
ssh semantic$j.example.org
done
```
 
 
--- 



# Day 2 

## 設定NameNode HA

#### 設定hdfs-site.xml
```
<property>
     <name>dfs.nameservices</name>
     <value>hadoop-ha</value>
   </property>
   <property>
     <name>dfs.ha.namenodes.hadoop-ha</name>
     <value>semantic03,semantic01</value>
   </property>
   <property>
     <name>dfs.namenode.rpc-address.hadoop-ha.semantic03</name>
     <value>semantic03.example.org:8020</value>
   </property>
   <property>
     <name>dfs.namenode.http-address.hadoop-ha.semantic03</name>
     <value>semantic03.example.org:50070</value>
   </property>
   <property>
     <name>dfs.namenode.rpc-address.hadoop-ha.semantic01</name>
     <value>semantic01.example.org:8020</value>
   </property>
   <property>
     <name>dfs.namenode.http-address.hadoop-ha.semantic01</name>
     <value>semantic01.example.org:50070</value>
   </property>
   <property>
     <name>dfs.namenode.shared.edits.dir</name>
     <value>qjournal://semantic03.example.org:8485;semantic01.example.org:8485;semantic05.example.org:8485/hadoop-ha</value>
   </property>
   <property>
     <name>dfs.journalnode.edits.dir</name>
     <value>/home/hadoop/journalnode</value>
   </property>
   <property>
     <name>dfs.ha.fencing.methods</name>
     <value>sshfence</value>
   </property>
   <property>
     <name>dfs.ha.fencing.ssh.private-key-files</name>
     <value>/home/hadoop/.ssh/id_rsa</value>
   </property>
   <property>
     <name>dfs.client.failover.proxy.provider.hadoop-ha</name>
     <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
   </property>

```

#### 由semantic03散佈hdfs-site.xml(用來設定NameNode High Availability)
```bash
#!/bin/bash

hdfssite=/usr/local/hadoop/etc/hadoop/hdfs-site.xml

for j in 01 02 04 05 06 07 08 09 10 11 12
do
  scp $hdfssite semantic$j:$hdfssite
done
```

#### 由semantic03散佈slaves(用來確定哪幾台是DataNode)
```bash
#!/bin/bash

slaves=/usr/local/hadoop/etc/hadoop/slaves

for j in 01 02 04 05 06 07 08 09 10 11 12
do
  scp $slaves semantic$j:$slaves
done
```

#### 失敗經驗：
````
原因：在HA還沒設定好就停掉hadoop服務再重啟，導致JobHistory Server無法啟動，其他成員則連不到hdfs（hdfs dfs -ls / 會error）

處置：把原本設好的hdfs-site.xml清空重新啟動蓋掉原本檔案，還原至一開始的狀態，再重新設定，成功解決(土法煉鋼還是有用的嘛～)
````
#### 設定core-site.xml

```
 <property>
     <name>fs.defaultFS</name>
     <value>hdfs://hadoop-ha</value>
   </property>
```

#### 散佈core-site.xml
```bash
#!/bin/bash

coresite=/usr/local/hadoop/etc/hadoop/core-site.xml

for j in 01 02 04 05 06 07 08 09 10 11 12
do
  scp $coresite semantic$j:$coresite
done
```

#### 發現secondaryNamenode停不掉，所以試著用到sbin裡找指令
```
cd /usr/local/hadoop/sbin
ll
hadoop-daemon.sh stop SecondaryNamenode
```
#### 發現還是無法關掉，所以使用優雅得kill掉他(預設15)
#### 題外話：在測試zookeeper是否正常運作時，無法使用hadoop-daemon.sh stop的方法可能是因為大小寫錯誤( NameNode(X), namenode(O) )
````
透過jps去看SecondaryNamenode的process Number
kill XXXX (預設是15)

再不行就溫柔勸說(-4)：你活著是沒用的去死好了XDDDDDDD
不要用9，檔案會損毀。
````

#### 散佈簡易防火牆，防止其他組的人來我們家
(/etc/hosts.allow 和 /etc/hosts.deny)
```bash
#!/bin/bash

hostsallow=/etc/hosts.allow
hostsdeny=/etc/hosts.deny
for j in 02 03 04 05 06 07 08 09 10 11 12
do
  scp $hostsallow semantic$j:/home/hadoop/
  scp $hostsdeny semantic$j:/home/hadoop/
done

for j in 02 03 04 05 06 07 08 09 10 11 12
do
  ssh -t semantic$j sudo cp /home/hadoop/hosts.allow /etc/hosts.allow
  ssh -t semantic$j sudo cp /home/hadoop/hosts.deny /etc/hosts.deny
done
```

#### 第一次啟動：
**First time** start services

##### Start journalnode service on semantic01,03,05
```bash
hadoop-daemon.sh start journalnode
```

##### On semantic03:(要設定namenode的機器)
Format NameNode hdfs - **only once**, if it's a new cluster
```bash
hdfs namenode -format
```
If NameNode hdfs already formatted
```bash
hdfs namenode -initializeSharedEdits
# 在hdfs加入日誌功能
```
Start semantic03 NameNode  (個別啟動)
```bash
hadoop-daemon.sh start namenode
```

##### On semantic01:
Copy the contents of Active NameNode metadata
```bash
hdfs namenode -bootstrapStandby
# 同步與namenode的日誌
```
Start semantic01 NameNode(個別啟動)
```bash
hadoop-daemon.sh start namenode
```
##### On semantic03:(要設定namenode的機器)
```bash
stop-dfs.sh
start-dfs.sh
```
#### Check HA Status
````bash
hdfs haadmin -getServiceState semantic01
hdfs haadmin -getServiceState semantic03
#接著會發現，兩台都是standby，所以需要透過手動的去啟動NameNode
````

#### Set semantic03 to active, needed by manually starting failover
```bash
hdfs haadmin -transitionToActive semantic03
```
#### Start yarn and history server
````bash
start-yarn.sh
mr-jobhistory-daemon.sh start historyserver
````
---

## Install and configure Zookeeper：semantic01,03,05(QJM)

目的：自動切換standby/active 
zookeeper 使用2n+1的法則
n=1(保證一台NN running)
zookeeper= 2X1+1=3

Install ZooKeeper on all namenodes
Download and unpack zookeeper

```bash
sudo -i #switch to root
wget http://apache.stu.edu.tw/zookeeper/zookeeper-3.4.9/zookeeper-3.4.9.tar.gz 
tar -xvf zookeeper-3.4.9.tar.gz -C /usr/local
mv /usr/local/zookeeper-3.4.9 /usr/local/zookeeper
chown -R hadoop:hadoop /usr/local/zookeeper
exit
```
#### 切換到hadoop Configure zoo.cfg file on all zookeeper(在設定為NameNode的三台semantic01,03,05(QJM))
```bash
su - hadoop

cd /usr/local/zookeeper/conf
cp zoo_sample.cfg zoo.cfg
nano zoo.cfg

#於zoo.cfg內設定以下內容：
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/usr/local/zookeeper/zoodata
dataLogDir=/usr/local/zookeeper/logs
clientPort=2181

server.1=semantic03.example.org:2888:3888
server.2=semantic01.example.org:2888:3888
server.3=semantic05.example.org:2888:3888
```
#### 建立 "zoodata" directory and a file named 'myid' in it (semantic03,01,05)
```bash
cd /usr/local/zookeeper
mkdir logs
mkdir zoodata
echo "1" > zoodata/myid #(on semantic03)
echo "2" > zoodata/myid #(on semantic01)
echo "3" > zoodata/myid #(on semantic05)
```
#### Configure environment variables (semantic03,01,05)
```bash
nano ~/.bashrc
   
export ZOOKEEPER_HOME=/usr/local/zookeeper
export PATH=\$PATH:$ZOOKEEPER_HOME/bin 
   
source ~/.bashrc
```

#### Starting ZooKeeper on all namenodes(semantic03,01,05)
```bash
zkServer.sh start
jps #zoopkeeper可看到QuorumPeerMain
lsof -nPi #看port number 2181 & 3888
```   

#### Configure Automatic Failover with Zookeeper

1) Shutdown hadoop cluster
```bash
hadoop-daemon.sh stop historyserver #on semantic12
stop-yarn.sh #on semantic02
stop-dfs.sh #on semantic03
``` 

2) Configure hdfs-site.xml (on all nodes in cluster)
```bash
cd /usr/local/hadoop/etc/hadoop/
nano hdfs-site.xml #加入以下XML部分
```
```xml
<property>
<name>dfs.ha.automatic-failover.enabled</name>
<value>true</value>
</property>
```
3) Configure core-site.xml
```
<property>
<name>ha.zookeeper.quorum</name>   <value>semantic03.example.org:2181,semantic01.example.org:2181,semantic05.example.org:2181</value>
</property>
```
4) Initializing HA state in ZooKeeper
```
hdfs zkfc -formatZK (on master1)
```
5) Starting the cluster 
```bash
start-dfs.sh
(automatically start a ZKFC daemon on NameNode
automatically select one NameNode to become active)
```
6) Check HA Status  
```bash
hdfs haadmin -getServiceState semantic03
hdfs haadmin -getServiceState semantic01
   
#http://semantic03.example.org:50070
#http://semantic01.example.org:50070
#到Active的那台semantic01關閉NameNode服務 測試zookeeper是否自動啟動另一台NameNode
hadoop-daemon.sh stop namenode
```
7) Start yarn and history server(on master3)
```
start-yarn.sh
mr-jobhistory-daemon.sh start historyserver

```


```bash
hdfssite=/usr/local/hadoop/etc/hadoop/hdfs-site.xml

for j in 01 02 04 05 06 07 08 09 10 11 12
do
  scp $hdfssite semantic$j:$hdfssite
done
```

```bash
coresite=/usr/local/hadoop/etc/hadoop/core-site.xml

for j in 01 02 04 05 06 07 08 09 10 11 12
do
  scp $coresite semantic$j:$coresite
done
```

```bash
yarnsite=/usr/local/hadoop/etc/hadoop/yarn-site.xml

for j in 01 02 04 05 06 07 08 09 10 11 12
do
  scp $yarnsite semantic$j:$yarnsite
done
```


#### Initializing HA state in ZooKeeper (on semantic03)
```
hdfs zkfc -formatZK 
# zkfc = zookeeper failover controller
start-dfs.sh
```
發現zookeeper啟動了SecondaryNameNode為NameNode，先停掉

#### on semantic01
```
hadoop-daemon.sh stop namenode
```

#### check NameNode active

確認semantic01停掉時semantic03自動變成active：簡報重點
```
hdfs haadmin -getServiceState semantic03
hdfs haadmin -getServiceState semantic01
```
若要semantic01啟動為SecondaryNameNode虛手動啟動semantic01。

#### 關機

```
1.停JobHistoryServer
2.停yarn
3.停hdfs (含停JournalNode,ZKFC)
4.停Zookeeper


```
#### 看看所有人的jps

```bash
for j in 01 02 03 04 05 06 07 08 09 10 11 12
do
  echo semantic${j}:
  ssh hadoop@semantic$j jps
  echo ====================
done
```

---


## ResourceManager的HA設定：

on semantic05：確認可以無密碼登入semantic01~12

#### 設定yarn-site.xml
加入
``` 
   <property>
      <name>yarn.resourcemanager.ha.enabled</name>
      <value>true</value>
   </property>
   <property>
      <name>yarn.resourcemanager.cluster-id</name>
      <value>rm_ha</value>
   </property>
   <property>
      <name>yarn.resourcemanager.ha.rm-ids</name>
      <value>semantic02,semantic05</value>
   </property>
   <property>
      <name>yarn.resourcemanager.hostname.semantic02</name>
      <value>semantic02.example.org</value>
   </property>
   <property>
      <name>yarn.resourcemanager.hostname.semantic05</name>
      <value>semantic05.example.org</value>
   </property>
   <property>
      <name>yarn.resourcemanager.recovery.enabled</name>
      <value>true</value>
   </property>
   <property>
      <name>yarn.resourcemanager.store.class</name>
      <value>org.apache.hadoop.yarn.server.resourcemanager.recovery.ZKRMStateStore</value>
   </property>
   <property>
      <name>yarn.resourcemanager.zk-address</name>
      <value>semantic03.example.org:2181,semantic01.example.org:2181,semantic05.example.org:2181</value>
   </property>
```
註解：
```
<!--<property>
        <name>yarn.resourcemanager.hostname</name>
        <value>master3.example.org</value>
   </property>-->
```
#### 停掉Jobhistory跟yarn 
```
mr-jobhistory-daemon.sh stop historyserver
stop-yarn.sh
```
#### 啟動yarn on semantic02與semantic05
```
start-yarn.sh 
```
#### 確認 active or standby on all node in cluster
```
yarn rmadmin -getServiceState semantic02
yarn rmadmin -getServiceState semantic05
```
#### 停掉ResourceManager on semantic02
```
yarn-daemon.sh stop resourcemanager
```
#### 確認semantic05是否自動替補為ResourceManager：簡報重點
```
yarn rmadmin -getServiceState semantic02
yarn rmadmin -getServiceState semantic05
```
#### 再啟動ResourceManager on semantic02
```
yarn-daemon.sh start resourcemanager
```
#### 從瀏覽器測試：簡報重點
```
http://semantic02.example.org:8088 
--> 自動導向到http://semantic05.example.org:8088(活著的ResourceManager)
```
#### 測試mapreduce
```
hadoop jar /usr/local/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.3.jar pi 30 100
```
---

## SPARK

#### 確認大家時間是否同步
```
for j in 01 02 03 04 05 06 07 08 09 10 11 12
do
   ssh semantic$j date
done
```

#### 修改時間，每台不要差超過10分鐘
```
sudo -i
(date MMDDhhmm)
date 12270711 
# UTC 格林威治時間 = CTS(中原標準時間) -8小時
```

#### 安裝spark，把老師的spark檔案貼入hadoop_classroom
```
sudo -i
tar -xvf /vagrant/spark-2.0.2-bin-hadoop2.7.tgz -C /usr/local
mv /usr/local/spark-2.0.2-bin-hadoop2.7 /usr/local/spark
chown -R hadoop:hadoop /usr/local/spark
exit

su - hadoop
```
SPARK不要放在PATH，因為hadoop的指令跟Spark的指令重疊度很高
#### 設定spark路徑
```
nano ~/.bashrc
  
# Set SPARK_HOME
export SPARK_HOME=/usr/local/spark
 
source ~/.bashrc
echo $SPARK_HOME
```
#### 讓spark知道hadoop的設定檔在哪裡
```
cd /usr/local/spark/conf
cp spark-env.sh.template spark-env.sh
   
nano spark-env.sh
export HADOOP_CONF_DIR=/usr/local/hadoop/etc/hadoop
```

#### 測試 spark-submit

```
cd $SPARK_HOME

./bin/spark-submit --class org.apache.spark.examples.SparkPi \
    --master yarn \
    --deploy-mode cluster \
    --driver-memory 1g \
    --executor-memory 1g \
    --executor-cores 1 \
    --queue default \
    examples/jars/spark-examples*.jar \
    100
```

#### 測試結果

```
http://semantic05.example.org:8088/cluster

> History > logs > stdout : Total file length is 33 bytes. 
> Pi is roughly 3.1419671141967114
```

Spark create a zip file with all jars under $SPARK_HOME/jars and upload it to HDFS each time you run spark-submit. You can avoid doing this copy in the following way:

#### Manually upload the Spark assembly JAR file to HDFS on semantic07
```
hdfs dfs -mkdir -p /user/spark/share/jars
hdfs dfs -put $SPARK_HOME/jars/* /user/spark/share/jars/
hdfs dfs -ls /user/spark/share/jars
```
#### 簡報重點：每次執行upload浪費時間，可以先upload上去節省執行時間

```
cd /usr/local/spark/conf
cp spark-defaults.conf.template spark-defaults.conf
nano spark-defaults.conf
spark.yarn.jars hdfs://hadoop-ha/user/spark/share/jars/*
```

#### 安裝scala
```
tar -xvf /vagrant/scala-2.11.8.tgz -C /usr/local
mv /usr/local/scala-2.11.8 /usr/local/scala
chown -R hadoop:hadoop /usr/local/scala
exit

# Switch to hadoop user
su - hadoop

# Set Scala environment variables
nano ~/.bashrc
   
# Set SCALA_HOME
export SCALA_HOME=/usr/local/scala
   
# Add Scala bin directory to PATH
export PATH=$PATH:$SCALA_HOME/bin

source ./.bashrc 

echo $SCALA_HOME

# Check if scala is ready
scala -version
```

#### 載入spark shell 

one node in the cluster
```
hdfs dfs -put /usr/local/spark/README.md /user/spark/share
hdfs dfs -ls /user/spark/share
```

```
cd $SPARK_HOME
./bin/spark-shell --master yarn --deploy-mode client
```
**簡報重點**：master = yarn 代表spark有連到ResourceManager，deploy-mode = client才能與shell互動 

#### 互動：Run scala script
can not use file:///usr/local/spark/README.md, because not all datanodes have this file
因為工作由ResourceManager分派，若指定的worker本機沒有這個檔案則無法執行，所以必須事先上傳至hdfs
```
scala> val textFile=sc.textFile("/user/spark/share/README.md")
textFile: org.apache.spark.rdd.RDD[String] = /user/spark/share/README.md MapPartitionsRDD[3] at textFile at <console>:24

scala> textFile.count
res0: Long = 99

scala> textFile.filter(_.contains("the")).count
res1: Long = 22

scala> sys.exit  
```

#### 互動：Run scala script
```
cd
nano WordCount.scala

import org.apache.spark.SparkConf
import org.apache.spark.SparkContext

object WordCount {
  def main(args: Array[String]){
    val fileRDD = sc.textFile("/user/spark/share/README.md")
    val wordsRDD = fileRDD.flatMap(line => line.split("\\W"))
    val mapRDD = wordsRDD.map(word => (word , 1))
    val reduceRDD = mapRDD.reduceByKey((value1, value2) => value1+value2)
    
    reduceRDD.take(10).foreach(println)
    sc.stop
  }
}
   
cd $SPARK_HOME
./bin/spark-shell --master yarn --deploy-mode client
   
scala>:load /home/hadoop/WordCount.scala
Loading /home/hadoop/WordCount.scala...
import org.apache.spark.SparkConf
import org.apache.spark.SparkContext
defined module WordCount

scala>WordCount.main(null)
   (scala,1)
   (package,3)
   (this,1)
   (Because,1)
   (Python,4)
   (its,1)
   (guide,1)
   (general,2)
   (have,1)
   (locally,3)

scala>sys.exit
```
#### 互動：pyspark
```
cd $SPARK_HOME
./bin/pyspark --master yarn --deploy-mode client  
   
>>> textFile=sc.textFile("/user/spark/share/README.md")
>>> textFile.count()
99
   
>>> textFile.first()
u'# Apache Spark'
   
>>> textFile.filter(lambda line: "the" in line).count()
22

>>> >>> exit()
```
spark2.0.2整合yarn，yarn重啟動，spark的資訊會留在yarn的網頁

#### 結論：spark-submit、spark-shell、pyspark都可以整合hadoop yarn 

#### 關機

```
1.停Jobhistory server
2.停兩台yarn(HA)
3.停hdfs
4.停zookeeper 
```

#### 安裝Hive
````
1) Download and unpack Hive
   sudo -i (switch to root)
   wget http://apache.stu.edu.tw/hive/stable-2/apache-hive-2.1.1-bin.tar.gz
   tar -xvf apache-hive-2.1.1-bin.tar.gz -C /usr/local
   mv /usr/local/apache-hive-2.1.1-bin /usr/local/hive
   chown -R hadoop:hadoop /usr/local/hive
   exit
````

#### Create folders in Hadoop file system

```
hdfs dfs -ls /
hdfs dfs -mkdir /tmp (if not exist)
hdfs dfs -mkdir -p /user/hive/warehouse
hdfs dfs -chmod g=rwx /tmp
hdfs dfs -chmod g=rwx /user/hive/warehouse
hdfs dfs -ls /user/hive
```
#### Change Hive metastore to Derby 
```
    cd $HIVE_HOME/conf
    cp hive-default.xml.template hive-site.xml
    vim hive-site.xml  
   
    <property>
        <name>javax.jdo.option.ConnectionURL</name>
        <value>jdbc:derby://localhost:1527/metastore_db;create=true</value>
        <description>metadata is stored in a derby server</description>
    </property>
    <property>
        <name>javax.jdo.option.ConnectionDriverName</name>
        <value>org.apache.derby.jdbc.ClientDriver</value>
        <description>derby server JDBC driver class</description>
    </property>
    <property>
        <name>javax.jdo.option.ConnectionUserName</name>
        <value>APP</value>
        <description>user name for connecting to derby server</description>
    </property>
    <property>
        <name>javax.jdo.option.ConnectionPassword</name>
        <value>>mine</value>
        <description>password for connecting to derby server</description>
    </property>
```

#### 設定允許遠端連入HiveServer2
```
          <property>
             <name>hive.server2.enable.doAs</name>
             <value>false</value>
          </property>
```


#### 建目錄：存放hive運作時的暫存資料
```
 mkdir $HIVE_HOME/iotmp
 nano $HIVE_HOME/conf/hive-site.xml
```

```
mkdir $HIVE_HOME/iotmp
	vim $HIVE_HOME/conf/hive-site.xml
	
	<property>
       <name>hive.exec.local.scratchdir</name>
       <value>/usr/local/hive/iotmp</value>
       <description>Local scratch space for Hive jobs</description>
    </property>
    <property>
       <name>hive.downloaded.resources.dir</name>
       <value>/usr/local/hive/iotmp</value>
       <description>Temporary local directory for added resources in the remote file      system.</description>
    </property>
    <property>
       <name>hive.querylog.location</name>
       <value>/usr/local/hive/iotmp</value>
       <description>Location of Hive run time structured log file</description>
    </property>
	<property>
       <name>hive.server2.logging.operation.log.location</name>
       <value>/usr/local/hive/iotmp/operation_logs</value>
       <description>Top level directory where operation logs are stored if logging f     unctionality is enabled</description>
    </property>
```

#### Copy Derby Jar Files：讓HIVE能找到DERBY
```
cp $DERBY_HOME/lib/derbyclient.jar $HIVE_HOME/lib
cp $DERBY_HOME/lib/derbytools.jar $HIVE_HOME/lib
```
#### 測試hive
```
hive
show databases;
exit;
```

#### 安裝Derby
```
sudo -i (switch to root)
wget http://apache.stu.edu.tw/db/derby/db-derby-10.13.1.1/db-derby-10.13.1.1-bin.tar.gz
tar -xvf db-derby-10.13.1.1-bin.tar.gz -C /usr/local
mv /usr/local/db-derby-10.13.1.1-bin /usr/local/derby
chown -R hadoop:hadoop /usr/local/derby
exit
```

#### Configure derby environment variables for hadoop account
```   
   su - hadoop
   nano ~/.bashrc
   
   export DERBY_HOME=/usr/local/derby
   export PATH=$PATH:$DERBY_HOME/bin
   
   source ~/.bashrc
```
#### Starting Derby
```
mkdir $DERBY_HOME/data
# in order to store Metastore data

cd $DERBY_HOME/data
nohup $DERBY_HOME/bin/startNetworkServer -h 0.0.0.0 &
# no hang up 使用者離開系統，服務照常執行
# 0.0.0.0 代表本機所有的IP
# & 背景執行

lsof -nPi | grep '1527'
java    27914 hadoop   35u  IPv4 724442      0t0  TCP *:1527 (LISTEN)
# *代表所有IP

ip addr show(ip a s)
# 查所有IP
```
HADOOP生態系統只支援IPV4，不支援IPV6

-
#### 開啟hiveserver2(因為前面已加入PATH路徑，故可以在背景程式啟動)
```
lsof -nPi
ps -ef | grep -i 'hiveserver2'
```

#### 前端beeline --> Derby  DB(1527/tcp)：測試可接到資料庫
```
beeline
!connect jdbc:derby://localhost:1527/metastore_db APP mine
# 預設database，db帳/密：APP/mine

# !connect jdbc:derby://localhost:1527//usr/local/derby/data/metastore_db APP mine
# 固定database
# see http://db.apache.org/derby/papers/DerbyTut/ns_intro.html

SELECT * FROM sys.systables;
!commit
!close
!quit
```

#### 前端beeline --> Hive Server2(1000/tcp)：真正分析時使用 
```
beeline
!connect jdbc:hive2:// hadoop hadoop
or 
!connect jdbc:hive2://localhost:10000 hadoop hadoop (need to start hiveserver2 and set doAS to fasle)
# 系統帳/密：hadoop/hadoop
          <property>
             <name>hive.server2.enable.doAs</name>
             <value>false</value>
          </property>	   
show databases;
!close
!quit

```

#### 停用HIVE

1.先停HiveServer2
```
# 查PID或查IP
jobs -l 
ps -ef | grep -i 'hiveserver2'

kill (PID)
```
2.再停DerbyDB
```
cd /usr/local/derby/bin/
stopNetworkServer
```

---

# Day 3

#### 開機：
```
zkServer.sh start (01 03 05)
start-dfs.sh (03)
start-yarn.sh (01 05)
mr-jobhistory-daemon.sh start historyserver (12)
```
#### 

一般：
1.一個hadoop cluster只要一台Hive Server 

Hadoop Application 一般都安裝在edge client（有hadoop套件但是不執行任何Hadoop的服務） 
ex) NamdeNode DataNode ZooKeeper...etc 

2. Hive Server 要有hadoop套件
3. Hive關聯DB(預設derby)裡面只存放meta data(資料庫的資訊、schema)       
4. Hive真正的DATA放在hdfs (預設路徑/user/hive/warehouse)

```
cd $DERBY_HOME/data
nohup $DERBY_HOME/bin/startNetworkServer -h 0.0.0.0 &

su - hadoop
/usr/local/hive/bin/hiveserver2 &

beeline 

!connect jdbc:hive2:// hadoop hadoop
or
!connect jdbc:hive2://localhost:10000 hadoop hadoop

```

#### 松鼠環境安裝

1.複製兩個檔案到vagrant

```
cp /usr/local/hive/lib/hive-jdbc-*-standalone.jar /vagrant
cp /usr/local/hadoop/share/hadoop/common/hadoop-common-2.7.3.jar /vagrant
```
2.以系統管理員身份打開CMD,在windows安裝松鼠

---

## R的安裝

HADOOP MODE:
1.Local Mode
2.Pseudo Distributed Cluster (standalone) --> R
3.Fully Distributed Cluster

``` 先啟動hadoop clusters服務
su - hadoop
hdfs namenode -format
start-dfs.sh
```
Starting namenodes on [localhost]
localhost: starting namenode, logging to /usr/local/hadoop/logs/hadoop-hadoop-namenode-node.out
localhost: starting datanode, logging to /usr/local/hadoop/logs/hadoop-hadoop-datanode-node.out
Starting **secondary namenodes [0.0.0.0]** 
The authenticity of host '0.0.0.0 (0.0.0.0)' can't be established.
ECDSA key fingerprint is 79:31:eb:fb:07:e7:82:bf:0d:2d:f3:f6:a5: cd:08:fe.
Are you sure you want to continue connecting (yes/no)? yes
0.0.0.0: Warning: Permanently added '0.0.0.0' (ECDSA) to the list of known hosts.
0.0.0.0: starting secondarynamenode, logging to /usr/local/hadoop/logs/hadoop-hadoop-secondarynamenode-node.out


```
start-yarn.sh
jps
```

#### 在所有的nodes上都要安裝R Switch to root & Set Env. variables
``` 
sudo -i
java -version
   

nano /etc/profile.d/hadoopenv.sh
export HADOOP_HOME="/usr/local/hadoop"
export HADOOP_CMD="/usr/local/hadoop/bin/hadoop"
export HADOOP_STREAMING="/usr/local/hadoop/share/hadoop/tools/lib/hadoop-streaming-2.7.3.jar"
   
chmod a+x /etc/profile.d/hadoopenv.sh
source /etc/profile.d/hadoopenv.sh
```
#### add the CRAN repository
```
Select mirror site from https://cran.r-project.org/mirrors.html
https://ftp.yzu.edu.tw/CRAN/	
http://ftp.yzu.edu.tw/CRAN/	
http://cran.csie.ntu.edu.tw/
   
nano /etc/apt/sources.list.d/rpackages.list
deb https://<my.favorite.cran.mirror>/bin/linux/ubuntu trusty/
deb https://ftp.yzu.edu.tw/CRAN/bin/linux/ubuntu trusty/

apt-get update
   
4) Secure APT (Optional) 額外:去取得apt-key 因為元智大學的apt裡面沒有
   gpg --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys E084DAB9
   (apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys E084DAB9)
   gpg -a --export E084DAB9 | sudo apt-key add -

5) install R system  安裝時間久 但是要十分小心 都必裝 少裝或是有出錯會影響下面的步驟
   apt-get install openssl libssl-dev     # for Hmisc R package
   apt-get install libcurl4-openssl-dev   # for devtools R package
   apt-get install r-base

6) Run R console 進到R的介面
   R
   help()
   q()   
 
7) Download the latest RHadoop related tar.gz files from 
   https://github.com/RevolutionAnalytics/RHadoop/wiki/Downloads and
#   https://github.com/RevolutionAnalytics/quickcheck

   plyrmr : perform common data manipulation operations on very large data sets stored on Hadoop
            Install this package on every node in the cluster
   rmr2   : perform statistical analysis in R via Hadoop MapReduce functionality on a Hadoop cluster
            Install this package on every node in the cluster
   rhdfs  : basic connectivity to the Hadoop Distributed File System
            Install this package only on the node that will run the R client
   ravro  : ability to read and write avro files from local and HDFS file system and adds an avro input format for rmr2
            Install this package only on the node that will run the R client
   rhbase : basic connectivity to the HBASE distributed database, using the Thrift server
            Install this package only on the node that will run the R client

   su - hadoop
   cd
   mkdir R
   cd R
   
   如果已經放到自己的目錄下 進到R目錄下之後CP檔案進來 (注意最後的"點號")
   cd R
   cp /vagrant/plyrmr_0.6.0.tar.gz .
   cp /vagrant/rhdfs_1.0.8.tar.gz .
   cp /vagrant/rmr2_3.3.1.tar.gz .
   
#   wget https://github.com/RevolutionAnalytics/quickcheck/releases/download/3.5.0/quickcheck_3.5.0.tar.gz
   wget https://github.com/RevolutionAnalytics/rhdfs/raw/master/build/rhdfs_1.0.8.tar.gz
   wget https://github.com/RevolutionAnalytics/rmr2/releases/download/3.3.1/rmr2_3.3.1.tar.gz
   wget https://github.com/RevolutionAnalytics/plyrmr/releases/download/0.6.0/plyrmr_0.6.0.tar.gz
   wget https://github.com/RevolutionAnalytics/ravro/raw/1.0.4/build/ravro_1.0.4.tar.gz (optional)
   wget https://github.com/RevolutionAnalytics/rhbase/raw/master/build/rhbase_1.2.1.tar.gz (optional)
			
8) Install R packages dependencies of step 7
   check the website : https://github.com/RevolutionAnalytics/RHadoop/wiki/Installing-RHadoop-on-RHEL
   
   Switch to root, execute these commands from R console:(有沒裝成功的package 再裝一次)
   >install.packages(c("codetools", "Rcpp", "RJSONIO", "bitops", "digest", "functional", "stringr", "plyr", "reshape2", "rJava")) 
   
   # if there are messages:		@@@@如果有出現下面的錯誤訊息 就離開R執行88行後的步驟
   Warning in install.packages :
   installation of package ‘rJava’ had non-zero exit status
   
   # solution:
   java -version
   R CMD javareconf
   因為預設的Java是open版的jdk，安裝的路徑跟Oracle的Java jdk安裝的路徑不一樣
   
   #enter R console and re-install packages
   
   >install.packages(c("dplyr","R.methodsS3"))
   >install.packages(c("Hmisc"))
   >install.packages(c("caTools"))
   >install.packages(c("memoise","rjson"))
   >install.packages("bit64")  # for ravro
   
   # Install quickcheck R package from github for rmr2, see https://github.com/RevolutionAnalytics/quickcheck
   >install.packages("devtools")
   >library(devtools)
   >install_github("RevolutionAnalytics/quickcheck@3.5.0", subdir = "pkg")  
      
#   >Sys.setenv(HADOOP_HOME="/usr/local/hadoop")
#   >Sys.setenv(HADOOP_CMD="/usr/local/hadoop/bin/hadoop")
#   >Sys.setenv(HADOOP_STREAMING="/usr/local/hadoop/share/hadoop/tools/lib/hadoop-streaming-2.7.3.jar")
   
   >Sys.getenv("HADOOP_HOME")
   >Sys.getenv("HADOOP_CMD")
   >Sys.getenv("HADOOP_STREAMING")

第7步驟有做完，才可以執行下面的步驟
   >update.packages(lib.loc="/home/hadoop/R")
#   >install.packages("/home/hadoop/R/quickcheck_3.5.0.tar.gz", repos=NULL, type="source") 
   >install.packages("/home/hadoop/R/rhdfs_1.0.8.tar.gz", repos=NULL, type="source") 
   >install.packages("/home/hadoop/R/rmr2_3.3.1.tar.gz", repos=NULL, type="source") 
   >install.packages("/home/hadoop/R/plyrmr_0.6.0.tar.gz", repos=NULL, type="source") 
   >install.packages("/home/hadoop/R/ravro_1.0.4.tar.gz", repos=NULL, type="source") 
#   >install.packages("/home/hadoop/R/rhbase_1.2.1.tar.gz", repos=NULL, type="source")

   >q()

9) Test R in hadoop is working or not:
   su - hadoop
   R
   >library(rmr2)
   >library(rhdfs)
   >hdfs.init()
   
   >ints = to.dfs(1:100)  # hdfs dfs -ls /tmp
   >calc = mapreduce(input = ints,map = function(k, v) cbind(v, 2*v))
   >from.dfs(calc)
   
   >q()



```


#### 安裝 Ganglia server
```
1) 在vagrant下進入管理者身份
sudo -i
apt-get update
apt-get install gmetad ganglia-webfrontend		#安裝SERVER
   apt-get install ganglia-monitor (if you need to monitor ganglia server)	#在SERVER上安裝監控軟體
   
When asked to restart Apache. Select "yes"

2)Copy Ganglia web front end configuration file to Apache sites folder
/usr/bin/install -c -m 644 /etc/ganglia-webfrontend/apache.conf /etc/apache2/sites-available/ganglia.conf

cd /etc/apache2/sites-enabled
ln -s ../sites-available/ganglia.conf .
service apache2 reload

3) Edit configuration file of the gmetad  (找到第44行)
nano /etc/ganglia/gmetad.conf
            Cluster ID 幾秒收集資料  collectorIP       
data_source "semantic" 300 192.168.33.202

PS:
a)gmetad: server
b)gmond: agent

4) vim /etc/ganglia/gmond.conf
comment out all mcast and 239.2.11.71 address
# mcast_join = 239.2.11.71    第35、42行加入#關閉(有mcast的部分
# bind = 239.2.11.71          第44行加入#關閉

```
#### 安裝Ganglia Agent
```
1) Install gmond (所有cluster都安裝)
   apt-get install ganglia-monitor
   
2) On Cluster data collector(master node--collector那台要改)
   nano /etc/ganglia/gmond.conf
   
   cluster {
      name = "semantic"
      owner = "unspecified"
      latlong = "unspecified"
      url = "unspecified"
   }

   udp_send_channel {
    # mcast_join = 239.2.11.71
      host = 192.168.33.202  #can not use localhost 要用collecter的IP
      port = 8649
      ttl = 1
   }
   
   udp_recv_channel {
      port = 8649
      bind = 192.168.56.90
   } 

3) On Other nodes(all slave nodes)
   vim /etc/ganglia/gmond.conf
   
   cluster {
      name = "semantic"
      owner = "unspecified"
      latlong = "unspecified"
      url = "unspecified"
   }

   udp_send_channel {
    # mcast_join = 239.2.11.71	#關閉mcast
      host = 192.168.56.90		#加入collector IP
      port = 8649
      ttl = 1
   }
   
   udp_recv_channel {
    # mcast_join = 239.2.11.71	#關閉mcast
      port = 8649
    # bind = 239.2.11.71		#關閉bind
   }
   
4) On all hadoop nodes, enable monitoring metrics
nano /usr/local/hadoop/etc/hadoop/hadoop-metrics2.properties 

*.period=600    #第4行 10改600
*.sink.file.class=org.apache.hadoop.metrics2.sink.FileSink
# for Ganglia 3.1 support #到39 41行去掉#打開功能
   *.sink.ganglia.class=org.apache.hadoop.metrics2.sink.ganglia.GangliaSink31 
*.sink.ganglia.period=600    #第41行10改600				

# default for supportsparse is false     #到44、46、47行去掉#打開功能
*.sink.ganglia.supportsparse=true		
   *.sink.ganglia.slope=jvm.metrics.gcCount=zero,jvm.metrics.memHeapUsedM=both
   *.sink.ganglia.dmax=jvm.metrics.threadsBlocked=70,jvm.metrics.memHeapUsedM=40

#到58 660 62 64 66 68行去掉#打開功能 並更改成collector的IP
namenode.sink.ganglia.servers=192.168.33.202:8649
datanode.sink.ganglia.servers=192.168.33.202:8649
resourcemanager.sink.ganglia.servers=192.168.33.202:8649
nodemanager.sink.ganglia.servers=192.168.33.202:8649
mrappmaster.sink.ganglia.servers=192.168.33.202:8649
jobhistoryserver.sink.ganglia.servers=192.168.33.202:8649


5) Restart the hadoop cluster, restart the gmetad on monitoring server,
   and restart gmond in hadoop nodes 

mr-jobhistory-daemon.sh stop historyserver
stop-yarn.sh
stop-dfs.sh
   
start-dfs.sh
start-yarn.sh
mr-jobhistory-daemon.sh start historyserver

sudo -i
service ganglia-monitor restart (on all nodes in cluster)
   
sudo -i
service gmetad restart (on monitoring server)
    
6) Open ganglia web interface 
   http://192.168.56.80/ganglia 
```
#### 
#### 
```
關閉Ganglia monitor
先轉成管理員權限
service ganglia-monitor stop
service ganglia-monitor status

```