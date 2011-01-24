# Unified Credentials Module for Oozie


## Background

Oozie is a workflow/scheduling solution for pure Grid processing needs to support the different job types exist in a Grid environment (M/R, PIG, Streaming, HDFS, etc.). This scheduling system is data aware, extensible, scalable and light weight. As Oozie is envisioned as a geteway for the grid for all the batch processing needs, it has to be aware of all other data processing systems which is getting used or will be used in the future for these purposes.

As Secure Hadoop is being used for the data processing then all components which had been built on hadoop will be using the same/different model for security needs and has its own security model to authenticate users. Now all the jobs are going through Oozie for hadoop and for these systems then Oozie should be having one unique interface and same/different implementation for these credentials modules by that Oozie will authenticate users with all those systems and run job seamlessly. 


## Options 

We have couple of options for implementation those are as follows:

   * Introduce separate actions ahead of all workflow applications which need specific authentication.
   * Oozie will get credentials for user based on configuration in each action.

### Separate Actions for Authentication

In this option Oozie will introduce multiple authentication actions and User will be using those actions ahead of their workflows to first get all the necessary credentials and pass those credentials to all the underneath actions in the workflows.
