## Workflow Use Cases

### Map Reduce Action

```xml
<workflow-app xmlns='uri:oozie:workflow:0.1' name='map-reduce-wf'>
    <start to='hadoop1' />
    <action name='hadoop1'>
        <map-reduce>
            <job-tracker>${jobTracker}</job-tracker>
            <name-node>${nameNode}</name-node>
            <configuration>
                <property>
                    <name>mapred.mapper.class</name>
                    <value>org.apache.oozie.example.SampleMapper</value>
                </property>
                <property>
                    <name>mapred.reducer.class</name>
                    <value>org.apache.oozie.example.SampleReducer</value>
                </property>
                <property>
                    <name>mapred.map.tasks</name>
                    <value>1</value>
                </property>
                <property>
                    <name>mapred.input.dir</name>
                    <value>input-data</value>
                </property>
                <property>
                    <name>mapred.output.dir</name>
                    <value>output-map-reduce</value>
                </property>
                <property>
                  <name>mapred.job.queue.name</name>
                  <value>unfunded</value>
                </property>
            </configuration>
        </map-reduce>
        <ok to="end" />
        <error to="fail" />
    </action>
    <kill name="fail">
        <message>Map/Reduce failed, error message[${wf:errorMessage(wf:lastErrorNode())}]</message>
    </kill>
    <end name='end' />
</workflow-app>
```

### PIG Action

```xml
<workflow-app xmlns='uri:oozie:workflow:0.1' name='pig-wf'>
    <start to='pig1' />
    <action name='pig1'>
        <pig>
            <job-tracker>${jobTracker}</job-tracker>
            <name-node>${nameNode}</name-node>
            <configuration>
                <property>
                    <name>mapred.compress.map.output</name>
                    <value>true</value>
                </property>
                <property>
                  <name>mapred.job.queue.name</name>
                  <value>unfunded</value>
                </property>
            </configuration>
            <script>org/apache/oozie/examples/pig/id.pig</script>
            <param>INPUT=input-data</param>
            <param>OUTPUT=output-data-pig/pig-output</param>
        </pig>
        <ok to="end" />
        <error to="fail" />
    </action>
    <kill name="fail">
        <message>Pig failed, error message[${wf:errorMessage(wf:lastErrorNode())}]</message>
    </kill>
    <end name='end' />
</workflow-app>
```



### PIG Action with UDFs

 * Workflow

```xml
<workflow-app xmlns='uri:oozie:workflow:0.1' name='pig-wf'>
  <action name="pig_1">
    <pig>
      <job-tracker>${jobTracker}</job-tracker>
      <name-node>${nameNode}</name-node>
      <prepare>
        <delete path="${nameNode}${outputDir}/pig_1" />
      </prepare>
       <configuration>
          <property>
             <name>mapred.map.output.compress</name>
             <value>false</value>
          </property>
          <property>
             <name>mapred.job.queue.name</name>
             <value>${queueName}</value>
          </property>
          <!-- optional -->
          <property>
            <name>mapred.child.java.opts</name>
            <value>-server -Xmx1024M -Djava.net.preferIPv4Stack=true -Dtest=QA</value>
          </property>
        </configuration>
        <script>org/apache/oozie/example/pig/script.pig</script>
        <param>INPUT=${inputDir}</param>
        <param>OUTPUT=${outputDir}/pig_1</param>
        <archive>archivedir/tutorial-udf.jar#udfjar</archive>
    </pig>
    <ok to="end" />
    <error to="fail" />
  </action>
</workflow-app>
```


 * PIG Script

```text
REGISTER udfjar/tutorial-udf.jar;
A = load '$INPUT/student_data' using PigStorage('\t') as (name: chararray, age: int, gpa: float);
B = foreach A generate org.apache.pig.tutorial.UPPER(name);
store B into '$OUTPUT' USING PigStorage(); 
```


### Streaming Action

```xml
<workflow-app xmlns='uri:oozie:workflow:0.1' name='streaming-wf'>
    <start to='streaming1' />
    <action name='streaming1'>
        <map-reduce>
            <job-tracker>${jobTracker}</job-tracker>
            <name-node>${nameNode}</name-node>
            <streaming>
                <mapper>/bin/cat</mapper>
                <reducer>/usr/bin/wc</reducer>
            </streaming>
            <configuration>
                <property>
                    <name>mapred.input.dir</name>
                    <value>${inputDir}</value>
                </property>
                <property>
                    <name>mapred.output.dir</name>
                    <value>${outputDir}/streaming-output</value>
                </property>
                <property>
                  <name>mapred.job.queue.name</name>
                  <value>${queueName}</value>
                </property>
            </configuration>
        </map-reduce>
        <ok to="end" />
        <error to="fail" />
    </action>
    <kill name="fail">
        <message>Streaming Map/Reduce failed, error message[${wf:errorMessage(wf:lastErrorNode())}]</message>
    </kill>
    <end name='end' />
</workflow-app>
```



### Sub Workflow Action

```xml
<workflow-app xmlns='uri:oozie:workflow:0.1' name='subwf'>
    <start to='subwf1' />
    <action name='subwf1'>
        <sub-workflow>
            <oozie>${oozie}</oozie>
            <app-path>${nameNode}/tmp/${wf:user()}/workflows/map-reduce</app-path>
            <configuration>
                <property>
                    <name>jobTracker</name>
                    <value>${jobTracker}</value>
                </property>
                <property>
                    <name>nameNode</name>
                    <value>${nameNode}</value>
                </property>
                <property>
                    <name>mapred.mapper.class</name>
                    <value>org.apache.oozie.example.SampleMapper</value>
                </property>
                <property>
                    <name>mapred.reducer.class</name>
                    <value>org.apache.oozie.example.SampleReducer</value>
                </property>
                <property>
                    <name>mapred.map.tasks</name>
                    <value>1</value>
                </property>
                <property>
                    <name>mapred.input.dir</name>
                    <value>${inputDir}</value>
                </property>
                <property>
                    <name>mapred.output.dir</name>
                    <value>${outputDir}/mapRed</value>
                </property>
                <property>
                  <name>mapred.job.queue.name</name>
                  <value>${queueName}</value>
                </property>
            </configuration>
        </sub-workflow>
        <ok to="end" />
        <error to="fail" />
    </action>
    <kill name="fail">
        <message>Sub workflow failed, error message[${wf:errorMessage(wf:lastErrorNode())}]</message>
    </kill>
    <end name='end' />
</workflow-app>
```

### Java-Main Action

```xml
<workflow-app xmlns='uri:oozie:workflow:0.1' name='java-main-wf'>
    <start to='java1' />
    <action name='java1'>
        <java>
            <job-tracker>${jobTracker}</job-tracker>
            <name-node>${nameNode}</name-node>
            <configuration>
                <property>
                    <name>mapred.job.queue.name</name>
                    <value>default</value>
                </property>
            </configuration>
            <main-class>org.apache.oozie.example.DemoJavaMain</main-class>
            <arg>argument1</arg>
            <arg>argument2</arg>
        </java>
        <ok to="end" />
        <error to="fail" />
    </action>
    <kill name="fail">
        <message>Java failed, error message[${wf:errorMessage(wf:lastErrorNode())}]</message>
    </kill>
    <end name='end' />
</workflow-app>
```


### Java-Main Action with Script support
Java-Main action could be use to runa perl or any shell script. In this example, a perl script test.pl that uses perl module DatetimeHlp.pm.

```xml
<workflow-app xmlns='uri:oozie:workflow:0.1' name='java-script-wf'>
    <start to='java2' />
    <action name='java2'>
        <java>
            <job-tracker>${jobTracker}</job-tracker>
            <name-node>${nameNode}</name-node>
            <configuration>
                <property>
                    <name>mapred.job.queue.name</name>
                    <value>${queueName}</value>
                </property>
            </configuration>
            <main-class>qa.test.tests.testShell</main-class>
            <arg>./test.pl</arg>
            <arg>WORLD</arg>
            <file>/tmp/${wf:user()}/test.pl#test.pl</file>
            <file>/tmp/${wf:user()}/DatetimeHlp.pm#DatetimeHlp.pm</file>
            <capture-output/>
        </java>
        <ok to="decision1" />
        <error to="fail" />
    </action>
    <decision name="decision1">
           <switch>
           <case to="end">${(wf:actionData('java2')['key1'] == "value1") and (wf:actionData('java2')['key2'] == "value2")}</case>
           <default to="fail" />
           </switch>
    </decision>
    <kill name="fail">
        <message>Java failed, error message[${wf:errorMessage(wf:lastErrorNode())}]</message>
    </kill>
    <end name='end' />
</workflow-app>
```


The corresponding java class is shown below.

```java
package qa.test.tests;
import qa.test.common.*;
import java.io.File;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.OutputStream;
import java.util.Calendar;
import java.util.Properties;
import java.util.Vector;
public class testShell {	
	public static void main (String[] args)
	{
		String cmdfile = args[0];
		String text = args[1];
		try{
			String runCmd1;
			runCmd1	      = cmdfile +" "+text;
                        System.out.println("Command: "+runCmd1);
			CmdRunner cr1 = new CmdRunner(runCmd1);
			Vector    v1  = cr1.run();
			String    l1  = ((String) v1.elementAt(0));
                        System.out.println("Output: "+l1);
            String s2 = "HELLO WORLD Time:";
            File file = new File(System.getProperty("oozie.action.output.properties"));
            Properties props = new Properties();
            if (l1.contains(s2)) {
               props.setProperty("key1", "value1");
               props.setProperty("key2", "value2");
            } else {
               props.setProperty("key1", "novalue");
               props.setProperty("key2", "novalue");
            }
            OutputStream os = new FileOutputStream(file);
            props.store(os, "");
            os.close();
            System.out.println(file.getAbsolutePath());
		}
		 catch (Exception e) {
			e.printStackTrace();
		} finally {
                        System.out.println("Done.");
                }
	}
}
```

Typical CmdRunner class could be:

```java
        import java.lang.*;
 	import java.io.*;
 	import java.util.*;
 	 // ================================================================
 	// CmdRunner is useful to run a command and wait until it's
	// finished and returns.
 	// ================================================================
 	public class CmdRunner {
 	
 	String[] cmd = new String[3];
 	Object token;
	
 	String BINPATH = "/usr/local/bin:/sbin:/usr/sbin:$PATH";
 	String HADOOPENV = "export HADOOP_HOME=.../hadoop/current;export HADOOP_CONF_DIR=...../conf/current;export JAVA_HOME=../share/yjava_jdk/java;";
	
 	public CmdRunner(String cmdline) {
 	
 	cmd[0] = "/bin/bash";
	cmd[1] = "-c";
 	//cmd[2] = HADOOPENV + "export PATH=" + BINPATH + ";" + cmdline;
	cmd[2] = cmdline;
	
	
	token = new Object();
	}
	
 	public Vector run() {
 	
 	Vector v = new Vector();
 	try {
 	String line;
 	Process p = Runtime.getRuntime().exec(cmd);
 	BufferedReader input = new BufferedReader(new InputStreamReader(p.getInputStream()));
 	BufferedReader error = new BufferedReader(new InputStreamReader(p.getErrorStream()));
 	ReadErrorStream re = new ReadErrorStream(error, token, v);
	re.run();
	
 	while ((line = input.readLine()) != null) {
 	synchronized (token) {
 	v.add(line);
 	}
 	}
 	input.close();
 	}
 	catch (Exception e) {
 	    e.printStackTrace();
	}
	
	return v;
	}
 	} 
```

### Multiple Actions

```xml
<workflow-app xmlns='uri:oozie:workflow:0.1' name='demo-wf'>
  <start to="map_reduce_1" />
  <action name="map_reduce_1">
    <map-reduce>
      <job-tracker>${jobTracker}</job-tracker>
      <name-node>${nameNode}</name-node>
      <configuration>
        <property>
          <name>mapred.mapper.class</name>
          <value>org.apache.oozie.example.DemoMapper</value>
        </property>
        <property>
            <name>mapred.mapoutput.key.class</name>
            <value>org.apache.hadoop.io.Text</value>
        </property>
        <property>
            <name>mapred.mapoutput.value.class</name>
            <value>org.apache.hadoop.io.IntWritable</value>
        </property>
        <property>
          <name>mapred.reducer.class</name>
          <value>org.apache.oozie.example.DemoReducer</value>
        </property>
        <property>
          <name>mapred.map.tasks</name>
          <value>1</value>
        </property>
        <property>
          <name>mapred.input.dir</name>
          <value>${inputDir}</value>
        </property>
        <property>
          <name>mapred.output.dir</name>
          <value>${outputDir}/mapred_1</value>
        </property>
        <property>
          <name>mapred.job.queue.name</name>
          <value>${queueName}</value>
        </property>
      </configuration>
    </map-reduce>
    <ok to="fork_1" />
    <error to="fail_1" />
  </action>
  <fork name='fork_1'>
        <path start='hdfs_1' />
        <path start='hadoop_streaming_1' />
  </fork>
  <action name="hdfs_1">
    <fs>
      <mkdir path="${nameNode}/tmp/${wf:user()}/hdfsdir1" />
    </fs>
    <ok to="join_1" />
    <error to="fail_1" />
  </action>
  <action name="hadoop_streaming_1">
  <map-reduce>
      <job-tracker>${jobTracker}</job-tracker>
      <name-node>${nameNode}</name-node>
      <prepare>
        <delete path="${nameNode}/tmp/${wf:user()}/hdfsdir1" />
      </prepare>
      <streaming>
        <mapper>/bin/cat</mapper>
        <reducer>/usr/bin/wc</reducer>
      </streaming>
      <configuration>
        <property>
          <name>mapred.input.dir</name>
          <value>${outputDir}/mapred_1</value>
        </property>
        <property>
          <name>mapred.output.dir</name>
          <value>${outputDir}/streaming</value>
        </property>
      </configuration>
    </map-reduce>
    <ok to="join_1" />
    <error to="fail_1" />
  </action>
  <join name='join_1' to='pig_1' />
   <action name="pig_1">
    <pig>
        <job-tracker>${jobTracker}</job-tracker>
        <name-node>${nameNode}</name-node>
        <configuration>
            <property>
                <name>mapred.map.output.compress</name>
                <value>false</value>
            </property>
            <property>
              <name>mapred.job.queue.name</name>
              <value>${queueName}</value>
            </property>
        </configuration>
        <script>org/apache/oozie/examples/pig/id.pig</script>
        <param>INPUT=${outputDir}/mapred_1</param>
        <param>OUTPUT=${outputDir}/pig_1</param>
    </pig>
    <ok to="end_1" />
    <error to="fail_1" />
  </action>
  <kill name="fail_1">
   <message>Demo workflow failed, error message[${wf:errorMessage(wf:lastErrorNode())}]</message>
 </kill>
 <end name="end_1" />
</workflow-app>
```
### How to run Map-reduce job written using new Hadoop API? 
Since new MR API (a.k.a. Hadoop 20 API) is neither stable nor supported, Hadoop team highly recommends not to use new MR API. Instead, Hadoop team recommends using the old API at least until Hadoop 0.22.x is released. Some more documents could be found at: Which MapReduce API to use. The reasons behind this recommendation are as follows

   1. You are guaranteed needing to rewrite once the api changes. You would not be saving the cost of rewrite.
   2. The api is not final and not mature. You would be taking the risk/cost of testing the code and then have it changed on you in the future.
   3. There is a possibility of backward incompatibility as Hadoop 20 API is not approved. You would take the risk of figuring our backward incompatibility issues.
   4. There would not be any support efforts if users bump into a problem. You would take the risk of maintaining unsupported code. 

Having said that, there is a way of running MR jobs written using 20 API in Oozie. Basically, you have to

    * change mapred.mapper.class to mapreduce.map.class
    * change mapred.reducer.class to mapreduce.reduce.class
    * add mapred.output.key.class
    * add mapred.output.value.class
    * and, include the following property into MR action configuration 
```xml
$ cat workflow.xml 
<workflow-app xmlns="uri:oozie:workflow:0.1" name="newAPI-wc-wf">
<!-- An example workflow.xml for the new MapReduce API. -->
  <start to="wc"/>
  <action name="wc">
    <map-reduce xmlns="uri:oozie:workflow:0.1">
      <job-tracker>${jobTracker}</job-tracker>
      <name-node>${nameNode}</name-node>

      <prepare>
        <delete path="${PREFIX}/yoozie_test/output-mr20/mapRed20" />
      </prepare>

      <configuration>

        <!-- These are important. -->
        <property>
          <name>mapred.mapper.new-api</name>
          <value>true</value>
        </property>
        <property>
          <name>mapred.reducer.new-api</name>
          <value>true</value>
        </property>

	<!-- Use WordCount as example MapReduce job. -->
        <property>
           <name>mapreduce.map.class</name>
           <value>org.apache.hadoop.examples.WordCount$TokenizerMapper</value>
        </property>
        <property>
           <name>mapreduce.reduce.class</name>
           <value>org.apache.hadoop.examples.WordCount$IntSumReducer</value>
        </property>
        <property>
           <name>mapred.output.key.class</name>
           <value>org.apache.hadoop.io.Text</value>
        </property>
        <property>
           <name>mapred.output.value.class</name>
           <value>org.apache.hadoop.io.IntWritable</value>
        </property>

        <!-- You need to provide values for the variables used here 
        in your job.properties file. -->
        <property>
          <name>mapred.input.dir</name>
          <value>${PREFIX}/yoozie_test/input-data</value>
        </property>
        <property>
          <name>mapred.output.dir</name>
          <value>${PREFIX}/yoozie_test/output-mr20/mapRed20</value>
        </property>
        <property>
          <name>mapred.job.queue.name</name>
          <value>${queueName}</value>
        </property>
      </configuration>
    </map-reduce>
    <ok to="end"/>
    <error to="fail"/>
  </action>
  <kill name="fail">
    <message>WordCount failed in step [${wf:lastErrorNode()}], error message[${wf:errorMessage(wf:lastErrorNode())}]</message>
  </kill>
  <end name="end"/>
</workflow-app>
```

```xml

# Adapt all values, especially the capitalized parts, then save as job.properties and run as:
#  oozie job -run -config LOCAL_PATH_TO/job.properties

$  cat job.properties
#Hadoop mapred.job.tracker, adapt!
jobTracker=HOSTNAME:8021

#Hadoop fs.default.name, adapt!
nameNode=hdfs://HOSTNAME:8020/

#prefix of the HDFS path for input and output, adapt!
PREFIX=hdfs://HOSTNAME:8020/user/PATH

#HDFS path where you need to copy workflow.xml and lib/hadoop-examples.jar to
oozie.wf.application.path=hdfs://HOSTNAME:8020/user/PATH/oozie-WordCount-NewAPI-app/

#one of the values from Hadoop mapred.queue.names
queueName=default

# These may also be needed and adapted.
mapreduce.jobtracker.kerberos.principal=mapred/_HOST@LOCALHOST
dfs.namenode.kerberos.principal=hdfs/_HOST@LOCALHOST

```

### Workflow Job to Create SLA events
A workflow job could be configured to record the events required to evaluate SLA compliance.

```xml
$ cat workflow.xml 
<workflow-app xmlns='uri:oozie:workflow:0.2'  xmlns:sla="uri:oozie:sla:0.1" name='map-reduce-wf'>
    <start to='hadoop1' />
    <action name='hadoop1'>
        <map-reduce>
            <job-tracker>${jobTracker}</job-tracker>
            <name-node>${nameNode}</name-node>
            <configuration>
                <property>
                    <name>mapred.mapper.class</name>
                    <value>org.apache.oozie.example.SampleMapper</value>
                </property>
                <property>
                    <name>mapred.reducer.class</name>
                    <value>org.apache.oozie.example.SampleReducer</value>
                </property>
                <property>
                    <name>mapred.map.tasks</name>
                    <value>1</value>
                </property>
                <property>
                    <name>mapred.input.dir</name>
                    <value>${inputDir}</value>
                </property>
                <property>
                    <name>mapred.output.dir</name>
                    <value>${outputDir}/mapRed</value>
                </property>
                <property>
                  <name>mapred.job.queue.name</name>
                  <value>${queueName}</value>
                </property>
            </configuration>
        </map-reduce>
        <ok to="end" />
        <error to="fail" />
    </action>
    <kill name="fail">
        <message>Map/Reduce failed, error message[${wf:errorMessage(wf:lastErrorNode())}]</message>
    </kill>
    <end name='end' />
    <sla:info> 
	 <sla:app-name>test-app</sla:app-name> 
	 <sla:nominal-time>2009-03-06T10:00Z</sla:nominal-time> 
	 <sla:should-start>5</sla:should-start> 
	 <sla:should-end>120</sla:should-end> 
	 <sla:notification-msg>Notifying User for nominal time : 2009-03-06T10:00Z </sla:notification-msg> 
	 <sla:alert-contact>abc@yahoo.com</sla:alert-contact> 
	 <sla:dev-contact>abc@yahoo.com</sla:dev-contact> 
	 <sla:qa-contact>abc@yahoo.com</sla:qa-contact> 
	 <sla:se-contact>abc@yahoo.com</sla:se-contact>
         <sla:alert-frequency>LAST_HOUR</sla:alert-frequency>
         <sla:alert-percentage>80</sla:alert-percentage>
    </sla:info>
</workflow-app>
```


      * Each workflow job will create at least three events for normal processing.    
      * The event *CREATED* specifies that the Workflow job is registered for SLA tracking.
      * When the job starts executing, an event record of type *STARTED* is inserted into sla_event table..
      * Finally when a job finishes, event of type either *SUCCEEDED/KILLED/FAILED* is generated.

### Workflow Action to Create SLA events
A workflow action could be configured to record the events required to evaluate SLA compliance.

```xml
$ cat workflow.xml 
<workflow-app xmlns='uri:oozie:workflow:0.2'  xmlns:sla="uri:oozie:sla:0.1" name='map-reduce-wf'>
    <start to='hadoop1' />
    <action name='hadoop1'>
        <map-reduce>
            <job-tracker>${jobTracker}</job-tracker>
            <name-node>${nameNode}</name-node>
            <configuration>
                <property>
                    <name>mapred.mapper.class</name>
                    <value>org.apache.oozie.example.SampleMapper</value>
                </property>
                <property>
                    <name>mapred.reducer.class</name>
                    <value>org.apache.oozie.example.SampleReducer</value>
                </property>
                <property>
                    <name>mapred.map.tasks</name>
                    <value>1</value>
                </property>
                <property>
                    <name>mapred.input.dir</name>
                    <value>${inputDir}</value>
                </property>
                <property>
                    <name>mapred.output.dir</name>
                    <value>${outputDir}/mapRed</value>
                </property>
                <property>
                  <name>mapred.job.queue.name</name>
                  <value>${queueName}</value>
                </property>
            </configuration>
        </map-reduce>
        <ok to="end" />
        <error to="fail" />
    	<sla:info> 
	      <sla:app-name>test-app</sla:app-name> 
	      <sla:nominal-time>2009-03-06T10:00Z</sla:nominal-time> 
	      <sla:should-start>${10 * MINUTES}</sla:should-start> 
	      <sla:should-end>${2 * HOURS}</sla:should-end> 
	      <sla:notification-msg>TEST ACTION : 2009-03-06T10:00Z </sla:notification-msg> 
	      <sla:alert-contact>abc@yahoo.com</sla:alert-contact> 
	      <sla:dev-contact>abc@yahoo.com</sla:dev-contact> 
	      <sla:qa-contact>abc@yahoo.com</sla:qa-contact> 
	      <sla:se-contact>abc@yahoo.com</sla:se-contact>
	      <sla:alert-frequency>LAST_HOUR</sla:alert-frequency>
	      <sla:alert-percentage>80</sla:alert-percentage>
        </sla:info>
    </action>
    <kill name="fail">
        <message>Map/Reduce failed, error message[${wf:errorMessage(wf:lastErrorNode())}]</message>
    </kill>
    <end name='end' />
</workflow-app>
```

      * Each workflow job will create at least three events for normal processing.    
      * The event *CREATED* specifies that the Workflow action is registered for SLA tracking.
      * When the action starts executing, an event record of type *STARTED* is inserted into sla_event table..
      * Finally when an action finishes, event of type either *SUCCEEDED/KILLED/FAILED* is generated.
