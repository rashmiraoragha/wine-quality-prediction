# wine-quality-prediction
wine-quality-prediction is a scala application which uses hadoop spark to train and predict wine quality in a distributed environment
Docker image: https://hub.docker.com/r/rashmiraoraghavendra94/cloudcomputing-project

To run prediction (using docker)
docker run -v "$(pwd)"/input:/input rashmiraoraghavendra94/cloudcomputing-project:myfirstimage
Note:
1. pathToTestDataSetFile from current working directory
2. Test dataset filename should be “TestDataset.csv”
 
To run prediction (without docker)

Goto spark-2.4.0/bin directory and run below command

 ./spark-submit --class demo.common.WineQualityModelTrain  s3a://cs-643-850-assignment-2/JAR/wine-quality-prediction_2.11-1.0.jar --master=local[*] --run-type=prediction 
--model-store-path=s3a://cs-643-850-assignment-2/output/model --testing-file-path=s3a:///”$(pwd)”/input/TestDataset.csv
Note:
Sample directory and path from current working directory

 
Docker build
docker build --no-cache -t rashmiraoragha/wine-quality-prediction:myfirstimage 
Commands
All installation commands should be run on all master and worker nodes
JAVA installation
sudo yum install java-1.8.0-openjdk
sudo yum install java-devel
sudo yum install java-1.8.0-openjdk-devel
sudo yum remove java-1.7.*
 
 
Hadoop installation
export HADOOP_VERSION=2.8.4
export HADOOP_HOME=${HOME}/hadoop-$HADOOP_VERSION
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
export PATH=${HADOOP_HOME}/bin:$PATH
curl -sL --retry 3   "http://archive.apache.org/dist/hadoop/common/hadoop-$HADOOP_VERSION/hadoop-$HADOOP_VERSION.tar.gz"   | gunzip   | tar -x -C /tmp/  && mv /tmp/hadoop-$HADOOP_VERSION $HADOOP_HOME  && rm -rf $HADOOP_HOME/share/doc


Spark intallation
export SPARK_VERSION=2.4.0
export SPARK_PACKAGE=spark-${SPARK_VERSION}-bin-without-hadoop
export SPARK_HOME=$HOME/spark-${SPARK_VERSION}
export SPARK_DIST_CLASSPATH=$(hadoop classpath)
export PATH=${SPARK_HOME}/bin:$PATH
Curl -sL --retry 3   "https://archive.apache.org/dist/spark/spark-${SPARK_VERSION}/${SPARK_PACKAGE}.tgz"   | gunzip   | tar x -C /tmp/   && mv /tmp/$SPARK_PACKAGE $SPARK_HOME

Start master
ssh -I ccpa2.pem ec2-user@54.82.211.128
cd spark-2.4.0-bin-without-hadoop/bin
./start-master.sh

Start worker nodes
worker 1:
ssh -I ccpa2.pem ec2-user@54.221.183.229
cd spark-2.4.0-bin-without-hadoop/bin
./start-slave.sh spark://ip-172-31-43-226.ec2.internal:7077

worker 2:
ssh -I ccpa2.pem ec2-user@54.242.155.83
cd spark-2.4.0-bin-without-hadoop/bin
./start-slave.sh spark://ip-172-31-43-226.ec2.internal:7077

worker 3:
ssh -I ccpa2.pem ec2-user@34.228.224.67
cd spark-2.4.0-bin-without-hadoop/bin
./start-slave.sh spark://ip-172-31-43-226.ec2.internal:7077
 

To train
./spark-submit --class demo.common.WineQualityModelTrain  --conf spark.executor.extraClassPath=s3a://cs-643-850-assignment-2/JAR/aws-java-sdk-1.11.95.jar:s3a://cs-643-850-assignment-2/JAR/aws-java-sdk-core-1.11.95.jar:s3a://cs-643-850-assignment-2/JAR/aws-java-sdk-s3-1.11.95.jar:s3a://cs-643-850-assignment-2/JAR/hadoop-aws-2.8.4.jar --driver-class-path s3a://cs-643-850-assignment-2/JAR/*.jar  --jars s3a://cs-643-850-assignment-2/JAR/*.jar s3a://cs-643-850-assignment-2/JAR/wine-quality-prediction_2.11-1.0.jar --master=spark://ip-172-31-43-226.ec2.internal:7077 --run-type=training --training-file-path=s3a://cs-643-850-assignment-2/input/training/TrainingDataset.csv --validation-file-path=s3a://cs-643-850-assignment-2/input/validation/ValidationDataset.csv --model-store-path=s3a://cs-643-850-assignment-2/output/model --testing-file-path=s3a://cs-643-850-assignment-2/input/validation/ValidationDataset.csv
Spark UI
http://ec2-54-82-211-128.compute-1.amazonaws.com:8080/
 
 
Spark executors
http://ec2-54-82-211-128.compute-1.amazonaws.com:4040/executors/ 
 
If you are running from EC2 instance, IAM role with S3 access or S3 keys are required to attach to talk to public or private s3 buckets. As a global configuration i used IAM role in my project.

1. IAM role creation with full Ec2/S3 access

2. Attaching the IAM role to create instance to access the s3 (private or public buckets) smoothly without any credentials

Attach the created IAM role and apply
 
3. Add your docker run below including build & spark-submit run

wget https://repo1.maven.org/maven2/com/amazonaws/aws-java-sdk-s3/1.11.95/aws-java-sdk-s3-1.11.95.jar
wget https://repo1.maven.org/maven2/com/amazonaws/aws-java-sdk-core/1.11.95/aws-java-sdk-core-1.11.95.jar
wget https://repo1.maven.org/maven2/org/apache/spark/spark-hive_2.11/2.4.0/spark-hive_2.11-2.4.0.jar
wget https://repo1.maven.org/maven2/com/databricks/spark-csv_2.11/1.5.0/spark-csv_2.11-1.5.0.jar
wget https://repo1.maven.org/maven2/org/apache/httpcomponents/httpclient/4.5.2/httpclient-4.5.2.jar
wget https://repo1.maven.org/maven2/org/apache/hadoop/hadoop-client/2.8.4/hadoop-client-2.8.4.jar
 wget https://repo1.maven.org/maven2/org/apache/hadoop/hadoop-common/2.8.4/hadoop-common-2.8.4.jar
wget https://repo1.maven.org/maven2/org/apache/hadoop/hadoop-aws/2.8.4/hadoop-aws-2.8.4.jar
wget https://repo1.maven.org/maven2/com/amazonaws/aws-java-sdk/1.11.95/aws-java-sdk-1.11.95.jar

Above commands should copy the jars under spark2.4.0/jars/ folder. Use the below command for docker prediction

./spark-submit --class demo.common.WineQualityModelTrain  s3a://cs-643-850-assignment-2/JAR/wine-quality-prediction_2.11-1.0.jar --master=local[*] --run-type=prediction 
--model-store-path=s3a://cs-643-850-assignment-2/output/model --testing-file-path=s3a://cs-643-850-assignment-2/input/validation/ValidationDataset.csv 




