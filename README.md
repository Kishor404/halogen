#!/bin/bash

# Update system
sudo apt update && sudo apt upgrade -y

# Install Java 8
sudo apt install -y openjdk-8-jdk
java -version

# Create Hadoop directory
mkdir -p ~/hadoop
mv ../hadoop-3.4.1.tar.gz ~/hadoop
cd ~/hadoop

# Download and extract Hadoop
tar -xvzf hadoop-3.4.1.tar.gz
sudo mv hadoop-3.4.1 /usr/local/hadoop

# Set environment variables
echo '
# Hadoop environment variables
export HADOOP_HOME=/usr/local/hadoop
export HADOOP_INSTALL=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_YARN_HOME=$HADOOP_HOME
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
' >> ~/.bashrc

# Reload bashrc
source ~/.bashrc

# Change to Hadoop config directory
cd /usr/local/hadoop/etc/hadoop

# Edit core-site.xml
cat > core-site.xml << EOF
<configuration>
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://localhost:9000</value>
  </property>
</configuration>
EOF

# Edit hdfs-site.xml
cat > hdfs-site.xml << EOF
<configuration>
  <property>
    <name>dfs.replication</name>
    <value>1</value>
  </property>
  <property>
    <name>dfs.name.dir</name>
    <value>file:///usr/local/hadoop/hdfs/namenode</value>
  </property>
  <property>
    <name>dfs.data.dir</name>
    <value>file:///usr/local/hadoop/hdfs/datanode</value>
  </property>
</configuration>
EOF

# Create necessary directories
sudo mkdir -p /usr/local/hadoop/hdfs/namenode
sudo mkdir -p /usr/local/hadoop/hdfs/datanode
sudo chown -R $USER:$USER /usr/local/hadoop/hdfs

# Edit mapred-site.xml
cp mapred-site.xml.template mapred-site.xml
cat > mapred-site.xml << EOF
<configuration>
  <property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
  </property>
</configuration>
EOF


# Edit yarn-site.xml
cat > yarn-site.xml << EOF
<configuration>
  <property>
    <name>yarn.resourcemanager.hostname</name>
    <value>localhost</value>
  </property>
</configuration>
EOF

# Set JAVA_HOME in hadoop-env.sh
JAVA_PATH=$(readlink -f $(which java) | sed 's:/bin/java::')
echo "export JAVA_HOME=$JAVA_PATH" >> /usr/local/hadoop/etc/hadoop/hadoop-env.sh

# Reload bashrc again
source ~/.bashrc

# Format HDFS
hdfs namenode -format

# Setup SSH
sudo apt install -y openssh-server
sudo systemctl start ssh
sudo systemctl enable ssh
ssh-keygen -t rsa -P "" -f ~/.ssh/id_rsa
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
ssh localhost

