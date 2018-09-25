## Create an App from a Docker image

In this lab you will learn how to create a new project on OpenShift and how to create an application from an existing docker image.

**Note** Please substitute the student ## below with your assigned # **if you are in a shared environment**. If you have a dedicated environment, this is not necessary. 

**Step 1: Add a new project from command line**


```
$ oc new-project mycliproject-student-## --description="My CLI Project student-##" --display-name="CLI Project"
```
Upon project creation, OpenShift will automatically switch to the newly created project/namespace. If you wish to view the list of projects, run the following command:

````
$ oc get projects
````
If you have more than one project, you can switch to a different one by issuing `oc project <project name>`. Although you don't want to do it now.

You can also check the status of the project by running the following command. It says that the project is currently not running anything.

```
$ oc status

In project CLI Project (mycliproject)

You have no services, deployment configs, or build configs.
Run 'oc new-app' to create an application.
```

**Step 2: Create an application from a Docker Image**

Next we will create an application inside the above project using an existing docker image. We will be using a very simple docker image on dockerhub that just says "Welcome to OpenShift V3". Let us just use that for this exercise.

First create a new application using the docker image using the `oc new-app` command as shown below:

```
$ oc new-app redhatworkshops/welcome-php --name=welcome

--> Found Docker image f001b13 (8 months old) from Docker Hub for "redhatworkshops/welcome-php"

    * An image stream will be created as "welcome:latest" that will track this image
    * This image will be deployed in deployment config "welcome"
    * Port 8080/tcp will be load balanced by service "welcome"
      * Other containers can access this service through the hostname "welcome"
    * WARNING: Image "redhatworkshops/welcome-php" runs as the 'root' user which may not be permitted by your cluster administrator

--> Creating resources with label app=welcome ...
    imagestream "welcome" created
    deploymentconfig "welcome" created
    service "welcome" created
--> Success
    Run 'oc status' to view your app.
```
The above command uses the docker image to deploy a docker container in a pod. If you quickly run `oc get pods` you will notice that a deployer pod runs and it starts an application pod as shown below.

```
$ oc get pods

NAME               READY     STATUS    RESTARTS   AGE
welcome-1-deploy   1/1       Running   0          1m
welcome-1-dkyyq    0/1       Pending   0          0s
```
In the above example `welcome-1-deploy` is the deployer pod and the other one is the actual application pod. In a little while the deployer pod will succeed and the application pod will change for `Pending` to `Running` status.

```
$ oc get pods

NAME              READY     STATUS    RESTARTS   AGE
welcome-1-dkyyq   1/1       Running   0          56s
```

**Step 3: Add a Route for your application**

OpenShift also spins up a service for this application. Run the following command to view the list of services in the project.

````
$ oc get services

NAME      CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
welcome   172.30.77.93   <none>        8080/TCP   2m
````

You will notice the `welcome` service was created for this project.

However, there is no route for this application yet. So you cannot access this application from outside.

Now add a route to the service with the following command. `oc expose` command will allow you to expose your service to the world so that you can access it from the browser.


````
$ oc expose service welcome

$ oc get route

NAME      HOST/PORT                                        PATH      SERVICE   LABELS
welcome    welcome-mycliproject-user02.apps.ose3sandbox.com          welcome   
````
**Note** Take note of your specific route URL above. 

**Step 4: Try your application**

Access the application: Now access the application using curl (looking for 200 status code) or from the browser and see the result: 

**Note** Take note of your specific route URL above. 


````
$ curl -Is http://welcome-mycliproject.apps.ocp.lab.arctiq.ca
````

Voila!! you created your first application using an existing docker image on OpenShift.

**Step 4: Clean up**

Run the `oc get all` command to view all the components that were created in your project.

````
$ oc get all

NAME      TYPE      SOURCE
NAME      TYPE      STATUS    POD
NAME      DOCKER REPO                   TAGS      UPDATED
welcome   redhatworkshops/welcome-php   latest    5 hours ago
NAME      TRIGGERS                    LATEST VERSION
welcome   ConfigChange, ImageChange   1
CONTROLLER   CONTAINER(S)   IMAGE(S)                             SELECTOR                                        REPLICAS
welcome-1    welcome        redhatworkshops/welcome-php:latest   deployment=welcome-1,deploymentconfig=welcome   1
NAME      HOST/PORT                     PATH      SERVICE   LABELS
welcome   welcome.apps.osecloud.com             welcome   
NAME      LABELS    SELECTOR                   IP(S)           PORT(S)
welcome   <none>    deploymentconfig=welcome   172.30.155.37   80/TCP
NAME              READY     REASON    RESTARTS   AGE
welcome-1-8d7nk   1/1       Running   0          4h
````

Now you can delete all these components by running one command.

````
$ oc delete all -l app=welcome

imagestream "welcome" deleted
deploymentconfig "welcome" deleted
route "welcome" deleted
service "welcome" deleted
pod "welcome-1-ynedb" deleted
````
You will notice that it has deleted the imagestream for the application, the deploymentconfig, the service and the route.

You can run `oc get all` again to make sure the project is empty.