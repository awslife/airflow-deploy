# airflow-deploy

Airflow is a platform for developing, scheduling and monitoring workflows programmatically. By using Airflow, it is possible to develop a DAG (Directed Acyclic Graph) type of workflow, and it is also possible to develop an expandable pipeline by connecting the DAG. Because of this convenience, many projects use Airflow, and related materials are also very easy to find, making it very easy to migrate legacy code. In addition, the public cloud provides Managed Airflow, so if you pay only the cost, the operational burden is very low.

You can check how to install Airflow Cluster through the official website. Considering the operating environment, installing Airflow using Production Docker Image, Helm Chart, and Managed Airflow Service is the best way. In a general single project, it is possible to respond to all requirements even with the system structure provided on the Airflow homepage, but in an environment where multiple project members share and use large-scale computing resources such as GPU clusters, the above configuration has many limitations. . For example, if Data Engineer and Data Scientist need to use the same Airflow Cluster, they need to think about ACL for DAG. In this article, we are going to install Airflow Cluster in an environment where VMs and K8S are mixed instead of using Managed Airflow, and furthermore, we will try to set up an Airflow Cluster that supports multi-tenancy.

## Airflow architecture

If you use Airflow 1.10 or higher, you will most likely use Kubernetes Executor for the resource expansion and convenience of Airflow Worker. The system structure at this time is as the image below guided by the Airflow homepage, and the DAG file of the Airflow Web Server and Scheduler is shared with the Worker Pod.
![Basic Airflow architecture](https://airflow.apache.org/docs/apache-airflow/2.0.1/concepts.html#concepts)

In order to run DAG in Airflow, the DAG file must be shared with the Web Server, Scheduler, and Worker. In order to share the DAG file with the Worker Pod running as a Kubernetes Pod, how to store DAG in Pod Image, How to use Persistent Volume. How to use Git Sync for example. The method of using Pod Image and PVC is very easy in terms of system construction, but it is not commonly used because the user is very difficult to modify the DAG file, and the method of using Git Sync has the advantage of being very easy to build and use, but modify the DAG file. The downside is that you have to set Git permissions on all of the users you need. Separating the Git branch and using it can be a good alternative, but since the splitting method is a way to share Git, it is also judged not to be a good method in terms of multi-tenancy.
![Airflow architecture with S3 for sharing DAG](https://awslife.github.io/assets/image/cicd/2022-05-19-airflow_cluster_architecture.png)

### Airflow architecture considering multi-tenancy with S3

In this article, DAG sharing between Web Server, Scheduler, and Kubernetes Executor was solved using S3, which is relatively easy to use, and the duplication of Web Server and Scheduler was also configured using Load Balance. The overall System Architecture is shown in the image below.

## Airflow cluster installation

Airflow’s Web Server and Scheduler will be installed in the VM environment, and it is assumed that Load Balancer, Metadata DB, and S3 required for Airflow Cluster are already established. Installation was decided on 3 units considering the HA configuration, and all settings should be executed in the same way for all 3 units.

### Airflow web server and scheduler H/W resource

The specifications of the Airflow web server and scheduler server are as follows, and since separate jobs are not executed in the web server and scheduler, high H/W resources are not required.

|Hostname|OS|IP|Role|vCPU|Memory|Disk|
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|airflow1|Rocky 8|10.0.0.120|Web Server, Scheduler|3|8 GiB|50 GiB|
|airflow2|Rocky 8|10.0.0.121|Web Server, Scheduler|3|8 GiB|50 GiB|
|airflow3|Rocky 8|10.0.0.122|Web Server, Scheduler|3|8 GiB|50 GiB|

### Airflow playbook deploy

- https://blogs.halodoc.io/airflow-cluster-architecture-at-halodoc-self-managed-challenges-and-learnings/
- https://airflow.apache.org/docs/apache-airflow/stable/installation.html#prerequisites
- https://medium.com/softwaresanders/making-apache-airflow-highly-available-1cfcec8996f2
