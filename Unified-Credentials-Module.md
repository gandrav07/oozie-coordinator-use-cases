# Unified Credentials Module for Oozie

## Background

Oozie is a workflow/scheduling solution for pure Grid processing needs to support the different job types exist in a Grid environment (M/R, PIG, Streaming, HDFS, etc.). This scheduling system is data aware, extensible, scalable and light weight. As Oozie is envisioned as a geteway for the grid for all the batch processing needs, it has to be aware of all other data processing systems which is getting used or will be used in the future for these purposes.

As Secure Hadoop is being used for the data processing then all components which had been built on hadoop will be using the same/different model for security needs and has its own security model to authenticate users. Now all the jobs are going through Oozie for hadoop and for these systems then Oozie should be having one unique interface and same/different implementation for these credentials modules by that Oozie will authenticate users with all those systems and run job seamlessly. 


## Options 

We have couple of options for implementation those are as follows:

   * Introduce separate actions ahead of all workflow applications which need specific authentication.
   * Oozie will get credentials for user based on configuration in each action.

Following section will discuss about their pros and cons and why we chosen the second option:

### Separate Actions for Credentials

In this option Oozie would have introduced multiple authentication actions and User will be using those actions ahead of their workflows to first get all the necessary credentials and pass those credentials to all the underneath actions in the workflows.
For an Example if user wants to use M/R actions and Pig Actions using ABC system then they first need to add ABC Action ahead of MR and Pig Actions and then oozie server will run ABC action on the gateway(oozie server) and provide all the necessary credentials to following actions. 

#### shortcomings

This is a nice approach however there are couple of shortcomings with this approach those are as follows.

   * In this approach, there would only be one delegation token for all the actions in the workflow however if workflows have long running actions then that token have potential problem for expiration and then all the subsequent actions will be failed due to authentication reason. The one solution to this approach is to add more time out which is a static number and will be configured at the workflow level (if interface is exposed from underneath system if not then cant be done this way). which will add more load to the underneath authenticator servers in case of short running actions.
   * There is another overhead of running one extra action per workflow.

### Getting Credentials in each action

The solution to above mentioned problem is to make each action responsible for its own needs, in this case credential token for different systems. Currently as well it is implemented in such a way for name node and job tracker. Every actions gets the token for itself for hdfs.

In this approach user will provide configuration for each workflow for all the needed/available credentials modules as well as user will also provided for each action, what are the credentials needed. Every action before running will call the appropriate credential modules to get the tokens and pass them in job conf for the tasks.

#### shortcomings

Shortcoming to this approach is every action has to authenticate itself but as of now there is no other way we can avoid that because of Token expiration problem perhaps one workflow may now authenticate many times with the same service, and that puts load on the auth service. There could be a de-authentication step after the action finishes in the future, if this turns out to be a problem.
