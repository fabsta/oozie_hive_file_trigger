# Short tutorial on how to run oozie jobs

Requirements:
oozie

### become oozie user
all actions will be executed as user oozie
```sh
$su - oozie
```
### define oozie server url
```sh
$export OOZIE_URL=http://192.168.100.141:11000/oozie
```

### go to project folder
```sh
$cd /home/oozie/examples/apps/oozie-hive.
```
you will find the following files:
- job.properties: namenode, jobtracker (please adjust these) as well as URI of workflow || coordinator app
- coordinator.xml: infos about scheduling and file trigger
- script.q: example hive script
- workflow.xml: the coordinator will execute what is in the workflow.xml (script.q) once the file exists


### copy files to hdfs
all project files - except job.properties - are expected to be on hdfs. So we copy the files to hdfs
```sh
$hdfs dfs -put -f /home/oozie/examples/apps/oozie-hive/ /user/oozie/examples/apps/
```

### start job
```sh
$oozie job -config job.properties -run
```
returns the job id. e.g. 0000000-160331163827649-oozie-oozi-C

What happens under the hood:
- oozie reads job.properties and the get the url of the wf (workflow) or coord (coordinator) application (on hdfs).
- (the only file not in hdfs is the job.properties file. keep that in mind when editing files. Update the hdfs via
(hdfs dfs -put -f examples/apps/oozie-hive/ /user/oozie/examples/apps/))
- oozie starts the wf/coord job.

Ok, the job is now running.

### check job log
There are two ways to check what your oozie jobs are doing: via command line and via web interface
#### command line
```sh
$oozie job  -log 0000000-160331163827649-oozie-oozi-C
```
#### web interface
```sh
oozie-UI http://172.17.97.141:11000/oozie/)
```
Checking the log wills tell you that the job is currently waiting for the following file: 
```sh
hdfs://biginsights01:8020/user/oozie/examples/input-data/rawLogs/2016/03/31//_SUCCESS
```
So, to start the job, we create the file with:
```sh
$hdfs dfs -put /home/oozie/_SUCCESS /user/oozie/examples/input-data/rawLogs/2016/03/31/
```

now if you check the file log again it should tell that it has found the file and that the job is running.

#### hive action + job verification
the example hive-script (script.q) is supposed to create a 'test' table in hive.
To verify that the workflow has worked, just do a 
```sh
hive -e 'show tables'
```



