# Bundle Requirements and Design
# Introduction
Bundle is a higher-level oozie abstraction that will batch a set of coordinator applications. The user will be able to start/stop/suspend/resume/rerun in the bundle level resulting a better control.

# High-level Bundle requirements
1. This feature will allow user to specify a list of coordinator applications in XML file format.

2. The name of the bundle xml file is not hard-coded. User can specify any name as bundle file.

3. User will submit a bundle by specifying the bundle application path in config file . An example command is: oozie job -run -config <bundle.properties>

4. Bundle application path is defined in config file as property "oozie.application.bundle.path" with a value of full path to bundle xml in the hdfs.

5. User can also submit a bundle job through WS API.

7. User will be able to define variables /parameters for each coordinator application.

8. All variables should be resolved during job submission. For any unresolved variable, oozie will throw an Exception.

9. User will be able to submit a bundle with an user-defined external id to avoid duplicate submissions in case of Timeout in first submission.

10. Oozie will not support any explicit dependencies among the coordinator XML in bundle definition.

11. Oozie will not support any partial bundle submission.

12. When user will submit a bundle , it will get a bundle id to track. Oozie will put the bundle job into PREP state.

13. User will be able to start a bundle using bundle id. It will put the bundle job into RUNNING state.

14. User will be able to combine submit and start into run that will start the bundle immediately.

15. User will be able to optionally specify the kick-off time to determine when to start a bundle. The bundle will not run until kick-off time reached.

16. User will be able to query Oozie for its status through CLI and WS API.

17. User will be able to query Oozie for all coordinator jobs that it started through CLI and WS API.

18. User will be able to kill a bundle id that will kill all spawned coordinator jobs.

19. User will be able to suspend a bundle id that will suspend all spawned coordinator jobs.

20. User will be able to pause a bundle id with a future time that will pause all spawned coordinator jobs.

21. User will be able to resume a bundle id that will resume all spawned coordinator jobs.

22. Bundle rerun requirements TBD. 

# High-level Design

## XML interface
User will specify a bundle through XML. The following XSD file defines the format of the XML file.

<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema" xmlns:bundle="uri:oozie:bundle:0.1" elementFormDefault="qualified" targetNamespace="uri:oozie:bundle:0.1">

    <xs:element name="bundle-app" type="bundle:BUNDLE-APP"/>
    <xs:simpleType name="IDENTIFIER">
        <xs:restriction base="xs:string">
            <xs:pattern value="([a-zA-Z]([\-_a-zA-Z0-9])*){1,39})"/>
        </xs:restriction>
    </xs:simpleType>
    <xs:complexType name="BUNDLE-APP">
        <xs:sequence>
            <xs:element name="controls" type="bundle:CONTROLS" minOccurs="0" maxOccurs="1"/>
            <xs:element name="coordinator" type="bundle:COORDINATOR" minOccurs="1" maxOccurs="unbounded"/>
        </xs:sequence>
        <xs:attribute name="name" type="bundle:IDENTIFIER" use="required"/>
    </xs:complexType>
    <xs:complexType name="CONTROLS">
        <xs:sequence minOccurs="0" maxOccurs="1">
            <xs:element name="kick-off-time" type="xs:string" minOccurs="0" maxOccurs="1"/>
        </xs:sequence>
    </xs:complexType>
    <xs:complexType name="COORDINATOR">
        <xs:sequence  minOccurs="1" maxOccurs="1">
            <xs:element name="app-path" type="xs:string" minOccurs="1" maxOccurs="1"/>
            <xs:element name="configuration" type="bundle:CONFIGURATION" minOccurs="0" maxOccurs="1"/>
        </xs:sequence>
        <xs:attribute name="name" type="bundle:IDENTIFIER" use="required"/>
        <xs:attribute name="critical" type="xs:string" use="optional"/>
    </xs:complexType>
    <xs:complexType name="CONFIGURATION">
        <xs:sequence>
            <xs:element name="property" minOccurs="1" maxOccurs="unbounded">
                <xs:complexType>
                    <xs:sequence>
                        <xs:element name="name" minOccurs="1" maxOccurs="1" type="xs:string"/>
                        <xs:element name="value" minOccurs="1" maxOccurs="1" type="xs:string"/>
                        <xs:element name="description" minOccurs="0" maxOccurs="1" type="xs:string"/>
                    </xs:sequence>
                </xs:complexType>
            </xs:element>
        </xs:sequence>
    </xs:complexType>
</xs:schema>

## DB Tables
There will be two new tables:

### Bundle Table
This table will contain the information related to a specific bundle.  The table structure would minimally contains the following:

* Bundle ID

* Bundle Status

* Bundle kick-off/start time

* Pending

* Bundle Creation time

* Last modified time

##Bundle Actions
This table will contain the information associated with coordinator jobs associate with each bundle.

* Bundle Action ID

* Coordinator name 

* Coordinator ID

* Coordinator Status

* Pending

* Last modified time



## State Transition
```xml


     Transition


From	                To	                  Trigger	                 Action on Children/Coordinator
==========================================================================================================================

Start	                 Prep	               User SUBMIT                          No Action

Prep	                 Running	               User START               	            SUBMIT children (START Bundle)
Prep	                 Running	               Kick off time reaches	            SUBMIT children (START Bundle)	
Prep	                 PrepPaused	       Pause time reaches                   No Action				
Prep	                 PrepSuspended	       User SUSPEND	                    No Action

PrepPaused	         Prep	              User RESET_PAUSE_TIME	            No Action
PrepSuspended	         Prep	              User RESUME	                    No Action

Running	                Suspended	     User SUSPEND	                    SUSPEND children/Coords
Running	                Paused	             Pause time	reaches                     No Action
Running	                Failed	             If SUBMIT of critical child fails	    KILL children
Running	                RunningWithError     If SUBMIT non-critical child fails	    No Action
Running	                Succeeded	    All children terminated                 No Action

RunningWithErrors	Failed	             If SUBMIT critical child fails	    KILL children
RunningWithErrors	DoneWithErrors	     All children terminated	  	    No Action
RunningWithErrors	SuspendedError	     User SUSPEND	                    SUSPEND children
RunningWithErrors	PausedError	     Pause time	reaches                     No Action

Paused	                Running	             User RESET_PAUSE_TIME                  RESET_PAUSE_TIME to children	
Paused	                Suspended	     User SUSPEND	                    SUSPEND children
PausedError	        RunningWithErrors    User RESET_PAUSE_TIME                  No Action	
PausedError	        SuspendedError	     User SUSPEND	                    SUSPEND children

Suspended	        Running	             User RESUME	                            RESUME children
SuspendedError	        RunningWithErrors    User RESUME   	                    RESUME children
	
Any_State	        Killed	             User KILL	                            KILL children

```

## Operations

### Bundle Submission
**How it is initiated**
   
* User submits a bundle

**Steps followed**

   * BundleSubmitXComamnd will authenticate the user request.
   * BundleSubmitXComamnd will parse the bundle XML and verify against corresponding XSD file.
   * BundleSubmitXComamnd will resolve (substitute) all variables referred in the XML.
   * BundleSubmitXComamnd will verify that there is no duplicate coordinator name in bundle XML.
   * BundleSubmitXComamnd will get a unique bundle id.
   * BundleSubmitXComamnd will insert the bundle record into *Bundle* table with status PREP and pending *0*.
   * BundleSubmitXComamnd will send back the bundle id to the user.
  
### Bundle Start
 **How it is initiated**

   * Bundle-start is initiated by any of the following ways (depending on whichever occurs first)
      * If kick-off-time (defined in the bundle xml) reaches. The default value is NOW (i.e. start immediately)
      * If user sends request to *START* the bundle.

**Steps followed**

   * BundleSubmitXComamnd will authenticate the user request.
   * BundleStartXComamnd will check if job status is PREP, otherwise oozie will fail the bundle-start request.
   * BundleStartXComamnd  will insert a record  into BUNDLE_ACTION table for each coordinator job with status PREP and pending *1*.
   * BundleStartXComamnd  will submit all coordinator job *asynchronously*. In other word, CoordSubmitXCommand will be queued .
   * BundleStartXComamnd will update the status from PREP to RUNNING.

### Bundle Kill
**How it is initiated**
   
* User Kills a bundle

**Steps followed**

   * BundleKillXComamnd will authenticate the user request.
   * BundleKillXComamnd will update the _bundle_ record with status KILLED and set pending to 1.
   * BundleKillXComamnd  will kill all coordinator job *asynchronously*. In other word, CoordKillXCommand will be queued.
   * BundleKillXComamnd  will update each related coordinator job with _status Killed and_ pending *1* in *BUNDLE_ACTION* table.
  
  
### Bundle Suspend

### Bundle Resume

### Bundle Pause/Unpause

### Bundle Rerun

### BundleStatusUpdateXCommand
**How it is initiated**
   
* When a coordinator job (associated with a bundle) change it status.

**Steps followed**

* Find the corresponding bundle action record in the *BUNDLE_ACTION* table.
* If the coordinator previous status doesn't match with bundle action current status
   * Update bundle action status with coordinator new status 
   * Decrement the pending count of bundle action.
   * Update the coordinator job Id into bundle action table.
* Otherwise
   * Decrement the pending count of bundle action (without changing its status)
   

### Status Transition Service
**How it is initiated**
  
* This service runs periodically to update the Bundle job status when all its children/actions are updated. All children (coordinator jobs) updates the Bundle Action entry when any status change happens. Status service periodically checks the table and update the corresponding Bundle job record with final status and/or pending flag.

* For example, when oozie receives a bundle KILL command, it updates bundle job status to Kill and its pending flag to true. Then it queues Kill command for all related coordinator jobs. At the same time update the bundle action tables with pending flag = true. When a coordinator job (asynchronously) updates its status to KILL, it update the bundle action table by resetting the pending flag. Status service periodically monitor the bundle action table. If the pending flag of all the bundle action tables are reset, StatusService will update the bundle job table by reseting theat pending flag. 





 