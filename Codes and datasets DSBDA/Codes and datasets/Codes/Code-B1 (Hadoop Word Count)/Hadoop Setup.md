# Setup for Hadoop

This file contains instructions to install and setup Hadoop for assignment B1.

---

## Prerequisites

- OpenJDK 17
- ssh
- curl
- gpg (optional)

---

1. Installing OpenJDK 17:

```shell
sudo apt update # Update packages
sudo apt install -y openjdk-17-jdk gpg curl ssh # Install OpenJDK 17
java -version # Check java (JRE) version
javac -version # Check javac (JDK) version
```

2. Setup Hadoop user:

```shell
sudo adduser --home /home/hadoop hadoop # Set default password as Pass@123
sudo usermod -aG sudo hadoop
su - hadoop
# Hit enter to skip entering all the information such as Full name, Room Number, etc.
```

3. SSH setup:

```shell
ssh-keygen -t ed25519 # Hit enter to save in defauly location. You can skip setting a passphrase for simplicity
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 640 ~/.ssh/authorized_keys
ssh localhost # You will be asked for conformation, type "yes"
su - hadoop
```

4. Download Hadoop (version 3.4.1 in this case):

```shell
cd ~
curl -o hadoop-3.4.1.tar.gz https://dlcdn.apache.org/hadoop/common/hadoop-3.4.1/hadoop-3.4.1.tar.gz # Download Hadoop
curl -o hadoop-3.4.1.tar.gz.sha512 https://downloads.apache.org/hadoop/common/hadoop-3.4.1/hadoop-3.4.1.tar.gz.sha512 # Download the hash
curl -o hadoop-3.4.1.tar.gz.asc https://downloads.apache.org/hadoop/common/hadoop-3.4.1/hadoop-3.4.1.tar.gz.asc # Download the signature
curl -o KEYS https://downloads.apache.org/hadoop/common/KEYS # Download the keys

```

5. Verify the hash and extract the tarball:

```shell
gpg --import KEYS # Import the keys
gpg --verify hadoop-3.4.1.tar.gz.asc # Verify hash signature
sha512sum -c hadoop-3.4.1.tar.gz.sha512 # Verify hash for file

tar -xvf hadoop-3.4.1.tar.gz
mv hadoop-3.4.1/ hadoop/
cd hadoop/
mkdir -p ~/hadoopdata/hdfs/{namenode,datanode}
```

6. Modify `~/.bashrc` & `$HADOOP_HOME/etc/hadoop/hadoop-env.sh`:

```shell
cp /etc/bash.bashrc ~/.bashrc # Reset .bashrc file
chown $USER ~/.bashrc # Change ownership of .bashrc file

echo -e "\n#JAVA+HADOOP CONFIG FROM KSKA GIT\nexport JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64\nexport PATH=\$JAVA_HOME/bin:\$PATH" >> ~/.bashrc # Java env var

echo -e "export HADOOP_HOME=/home/hadoop/hadoop/\nexport HADOOP_INSTALL=\$HADOOP_HOME\nexport HADOOP_MAPRED_HOME=\$HADOOP_HOME\nexport HADOOP_COMMON_HOME=\$HADOOP_HOME\nexport HADOOP_HDFS_HOME=\$HADOOP_HOME\nexport YARN_HOME=\$HADOOP_HOME\nexport HADOOP_COMMON_LIB_NATIVE_DIR=\$HADOOP_HOME/lib/native\nexport PATH=\$PATH:\$HADOOP_HOME/sbin:\$HADOOP_HOME/bin\nexport HADOOP_OPTS=\"-Djava.library.path=\$HADOOP_HOME/lib/native\"" >> ~/.bashrc # Hadoop env var

echo -e "PATH=\$PATH:\$HADOOP_HOME/sbin" >> ~/.bashrc

source ~/.bashrc

sed -i 's|^# export JAVA_HOME=.*|export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64/|' "$HADOOP_HOME/etc/hadoop/hadoop-env.sh"

```

7. Modify hadoop config files:

```shell
sed -i '/<configuration>/,/<\/configuration>/d' $HADOOP_HOME/etc/hadoop/core-site.xml
echo "<configuration>
<property>
  <name>hadoop.tmp.dir</name>
  <value>/home/hadoop/tmp</value>
</property>
<property>
  <name>fs.default.name</name>
  <value>hdfs://localhost:9000</value>
</property>
</configuration>" >> $HADOOP_HOME/etc/hadoop/core-site.xml

sed -i '/<configuration>/,/<\/configuration>/d' $HADOOP_HOME/etc/hadoop/hdfs-site.xml
echo "<configuration>
<property>
  <name>dfs.namenode.dir</name>
  <value>file:///home/hadoop/hadoopdata/hdfs/namenode</value>
</property>
<property>
  <name>dfs.data.dir</name>
  <value>file:///home/hadoop/hadoopdata/hdfs/datanode</value>
</property>
<property>
  <name>dfs.replication</name>
  <value>1</value>
</property>
</configuration>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml

sed -i '/<configuration>/,/<\/configuration>/d' $HADOOP_HOME/etc/hadoop/mapred-site.xml
echo "<configuration>
<property>
  <name>yarn.app.mapreduce.am.env</name>
  <value>HADOOP_MAPRED_HOME=$HADOOP_HOME/home/hadoop/hadoop/bin/hadoop</value>
</property>
<property>
  <name>mapreduce.map.env</name>
  <value>HADOOP_MAPRED_HOME=$HADOOP_HOME/home/hadoop/hadoop/bin/hadoop</value>
</property>
<property>
  <name>mapreduce.reduce.env</name>
  <value>HADOOP_MAPRED_HOME=$HADOOP_HOME/home/hadoop/hadoop/bin/hadoop</value>
</property>
</configuration>" >> $HADOOP_HOME/etc/hadoop/mapred-site.xml

sed -i '/<configuration>/,/<\/configuration>/d' $HADOOP_HOME/etc/hadoop/yarn-site.xml
echo "<configuration>
<property>
  <name>yarn.nodemanager.aux-services</name>
  <value>mapreduce_shuffle</value>
</property>
<property>
  <name>yarn.resourcemanager.hostname</name>
  <value>localhost</value>
</property>
</configuration>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml

```

8. Format HDFS namenode:

```shell
hdfs namenode -format # Format hdfs namenode
```

9. Start hadoop cluster:

```shell
start-all.sh
jps
```

> [!NOTE]
> Visit `localhost:9870` on your browser!

---

## Manually start/stop services

### Start all services

```shell
# start
hdfs --daemon start namenode
hdfs --daemon start datanode
yarn --daemon start nodemanager
yarn --daemon start resourcemanager
hdfs --daemon start secondarynamenode
hdfs dfsadmin -report
yarn node -list
jps # Check status
```

### Stop all services

```shell
# stop
hdfs --daemon stop namenode
hdfs --daemon stop datanode
yarn --daemon stop resourcemanager
yarn --daemon stop nodemanager
hdfs --daemon stop secondarynamenode
jps # Check status
```

---

## References

1. https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/SingleCluster.html
2. https://medium.com/@abhikdey06/apache-hadoop-3-3-6-installation-on-ubuntu-22-04-14516bceec85
