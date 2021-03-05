Tekton  / OpenShift Pipelines demo.

There are tons of docs on telling the good things about tekton or OpenShift Pipelines.


===
Resources needed
===
1. tkn cli
2. oc cli
3. connection to an openshift cluster. I have tested this on CRC version 4.6
===


1. To add OpenShift pipelines, install the OpenSHift Pipeline operator from the operatorhub.

It will give some custom api resources that we will use for the pipelines.

Such as:

1. Task
2. Taskrun
3. Pipeline
4. Pipelinerun

===

1. Task: A task is the fundamental unit of carring out the steps. Tasks can have multiple steps that carry out the actual work. They can be seen as similar to a stage in the jenkinsfile. Task can be parameterized for easy sharing with others. It can be added parameters in the definition. The task can be individual as a standalone entity or as a part of the pipeline as well.

2. Taskrun: A taskrun is the instantiation of a task. Just like from a template, we create resources. From a task, we "instantiate" it with our given parameters where we start the task. A Taskrun is automatically created when we start the task from tkn cli. 

3. Pipeline: A pipeline is a collection of multiple tasks that are executed in the order. It is like the JenkinsPipeline in Jenkins that has more than 1 steps and can accept parameters during execution.

4. Pipelinerun: Just as a taskrun which is an execution of a task with given parameters, a pipelinerun is an "instantiation" with custom parameters. A pipelinerun is automatically created on starting the pipeline from the tkn cli. 

==

Artifacts in the pipeline.

--> This pipeline has 2 tasks. There are 2 task files. 
----------------
1. start-build : This task takes an image from a public repository and creates the new application using the image in the project where the task is defined.
In case using a private image, create the imagepullsecret beforehand :)

This task takes 2 input parameters:

1. app-name : this is the name of the application resources which will be given during task creation.
2. image-name : this accepts the fqdn of the image that you want to deploy. 
----------------

2. create-route : This task is executed after the start-build and it exposes the service to create a route that was made in the previous stage. 

It also accept two input param:

1. route-name: The name of the route resource you want to give for the application being delivered.
2. svc-name: the name of the service to use for which the route has to be created.
----------------

HOW TO USE THE PIPELINE:

This pipeline is using 3 different inputs which it will pass to the tasks and then using the inputs the tasks will be executed.

A. pl-app-name: Input for the pipeline that would be passwd to the app-name 


1. Publish / Create the tasks and pipelines to the cluster

[root@localhost ~]#  create -f start-build.yml 
[root@localhost ~]#  create -f create-route.yml 
[root@localhost ~]#  create -f pipeline.yml


2. Use tkn cli to list the pipelines. tkn can use the same context that oc is using that the moment. So the project where you published the tasks and pipelines, tkn will be using the same project. 

[root@localhost ~]#  tkn task ls
NAME           DESCRIPTION   AGE
create-route                 6 seconds ago
start-build                  5 seconds ago
[root@localhost ~]#  tkn task ls
NAME               AGE              LAST RUN                     STARTED         DURATION     STATUS
build-and-deploy   20 seconds ago   -                            -               -            -


3. Use the tkn cli to start the pipeline and verify the steps and tasks.

[root@localhost ~]# tkn pipeline start --showlog build-and-deploy -p pl-app-name=testing -p pl-image-name=registry.access.redhat.com/rhscl/httpd-24-rhel7 -p pl-route-name=shubham


===
Output: you should be able to see the logs that says application was deployed and the route was executed.

Each task is executed in its own pod. To Verify,

[root@localhost ~]# oc get pods 
NAME                                                      READY   STATUS      RESTARTS   AGE
build-and-deploy-run-fcclp-create-route-929px-pod-4mv2v   0/1     Completed   0          29s
build-and-deploy-run-fcclp-deploy-app-n6fz4-pod-rwlzg     0/1     Completed   0          91s
testing-59cb796b-p5nvt                                    1/1     Running     0          27s

