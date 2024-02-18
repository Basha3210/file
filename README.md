Kafka + Spark Streaming Example

Watch the video here
This is an example of building a Proof-of-concept for Kafka + Spark streaming from scratch. This is meant to be a resource for video tutorial I made, so it won't go into extreme detail on certain steps. It can still be used as a follow-along tutorial if you like.

Also, this isn't meant to explain the design of Kafka/Hadoop, instead it's an actual hands-on example. I'd recommend learning the basics of these technologies before jumping in.

When considering this POC, I thought Twitter would be a great source of streamed data, plus it would be easy to peform simple transformations on the data. So for this example, we will

create a stream of tweets that will be sent to a Kafka queue
pull the tweets from the Kafka cluster
calculate the character count and word count for each tweet
save this data to a Hive table
To do this, we are going to set up an environment that includes

a single-node Kafka cluster
a single-node Hadoop cluster
Hive and Spark


2. Install Kafka
"Installing" Kafka is done by downloading the code from one of the several mirrors. After finding the latest binaries from the downloads page, choose one of the mirror sites and wget it into your home directory.

~$ wget http://apache.claz.org/kafka/2.2.0/kafka_2.12-2.2.0.tgz
After that you will need to unpack it. At this point, I also like to rename the Kafka to something a little more concise.

~$ tar -xvf kafka_2.12-2.2.0.tgz
~$ mv kafka_2.12-2.2.0.tgz kafka
Before continuing with Kafka, we'll need to install Java.

~$ sudo apt install openjdk-8-jdk -y
Test the Java installation by checking the version.

~$ java -version
Now you can cd into the kafka/ directory and start a Zookeeper instance, create a Kafka broker, and publish/subscribe to topics. You can get a feel for this by walking thru the Kafka Quickstart, but it will also be covered later in this example.

While we are here, we should install a Python package that will allow us to connect to our Kafka cluster. But first, you'll need to make sure that you have Python 3 and pip installed on your system. For Lubuntu, Python 3 was already installed and accessible with python3, but I had to install pip3 with

~$ pip3 install kakfa-python 
Confirm this installation with

~$ pip3 list | grep kafka
3. Install Hadoop
Similar to Kafka, we first need to download the binaries from a mirror. All releases can be found here. I used Hadoop 2.8.5. Select a mirror, download it, then unpack it to your home directory.

NOTE: I wrote out this step as using wget, but feel free to download the tar thru a browser (sometimes it is faster i think)
NOTE #2: Again, I renamed my directory for convenience.
~$ wget https://archive.apache.org/dist/hadoop/common/hadoop-2.8.5/hadoop-2.8.5.tar.gz
~$ tar -xvf hadoop-2.8.5.tar.gz
~$ mv hadoop-2.8.5.tar.gz hadoop
~$ cd hadoop
~/hadoop$ pwd
/home/<USER>/hadoop
Edit .bashrc found in your home directory, and add the following.

export HADOOP_HOME=/home/<USER>/hadoop
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
export HADOOP_HDFS_HOME=$HADOOP_HOME
export HADOOP_INSTALL=$HADOOP_HOME
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin
export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib/native"

export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
Be sure to insert your username on the first line, for example, for me it is export HADOOP_HOME=/home/davis/hadoop. Also, double check that your $JAVA_HOME is correct. This should be the default install location, but it may go somewhere else.

Also also, remember to execute your .bashrc after editing it so that the changes take place (this will be important later on)

~$ source .bashrc
Next, we will need to edit/add some configuration files. From the Hadoop home folder (the one named hadoop that is in your home directory), cd into etc/hadoop.

Add the following to hadoop-env.sh

export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export HADOOP_CONF_DIR=${HADOOP_CONF_DIR:-"/home/<USER>/hadoop/etc/hadoop"}
Replace the file core-site.xml with the following:

<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
    <property>
        <name>fs.default.name</name>
        <value>hdfs://localhost:9000</value>
    </property>
</configuration>
Replcae the file hdfs-site.xml with the following:

<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
    <property>
        <name>dfs.permission</name>
        <value>false</value>
    </property>
</configuration>
As mentioned earlier, we are just setting up a single-node Hadoop cluster. This isn'y very realistic, but it works for this example. For this to work, we need to allow our machine to SSH into itself.

First, install SSH with

~$ sudo apt install openssh-server openssh-client -y
Then, set up password-less authentication

~$ ssh-keygen -t rsa
~$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
Then, try SSH-ing into the machine (type exit to quit the SSH session and return)

~$ ssh localhost
With Hadoop configured and SSH setup, we can start the Hadoop cluster and test the installation.

~$ hdfs namenode -format
~$ start-dfs.sh
NOTE: These commands should be available anywhere since we added them to the PATH during configuration. If you're having troubles, hdfs and start-dfs are located in hadoop/bin and hadoop/sbin, respectively.

Finally, test that Hadoop was correctly installed by checking the Hadoop Distributed File System (HDFS):

~$ hadoop fs -ls /
If this command doesn't return an error, then we can continue!
