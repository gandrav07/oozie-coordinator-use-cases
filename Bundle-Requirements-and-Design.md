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
`[[/path/to/image.png]]
**         Transition	
	
From	····            To	                           Trigger	                                             Action

=============================================================**

Start	                 Prep	                user SUBMIT

Prep	                 Running	               User START               	            SUBMIT children (START Bundle)

Prep	                 Running	               Kick off time reaches	            SUBMIT children (START Bundle)	

Prep	                 PrepPaused	       Pause time reaches	

		START	
Prep	PrepSuspended	SUSPEND	
PrepPaused	Prep	RESET_PAUSE_TIME	
PrepSuspended	Prep	RESUME	
Running	Suspended	SUSPEND	SUSPEND children
Running	Paused	Pause time	PAUSE children
Paused	Running	RESET_PAUSE_TIME	
Paused	Suspended	SUSPEND	SUSPEND children
Suspended	Running	RESUME	RESUME children
Running	Failed	If SUBMIT critical child fails	KILL children
Running	RunningWithErrors	If SUBMIT non-critical child fails	
RunningWithErrors	Failed	If SUBMIT critical child fails	KILL children
RunningWithErrors	DoneWithErrors	All children terminated	
RunningWithErrors	SuspendedError	SUSPEND	SUSPEND children
RunningWithErrors	PausedError	Pause time	PAUSE children
PausedError	RunningWithErrors	RESET_PAUSE_TIME	
PausedError	SuspendedError	SUSPEND	SUSPEND children
SuspendedError	RunningWithErrors	RESUME	RESUME children
Running	Succeeded	All children terminated	
*	Killed	KILL	KILL children`
## Operations

### Bundle Submission
When a bundle submission request will reach to oozie

### Bundle Start

### Bundle Kill

### Bundle Suspend

### Bundle Resume

### Bundle Pause/Unpause

### Bundle Rerun


 