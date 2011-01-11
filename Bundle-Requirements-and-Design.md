# What is a Bundle?
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


 