# Building and deploying a RESTful web service on OpenShift 4.x

## What you will learn

Red Hat OpenShift is a leading hybrid cloud, enterprise Kubernetes application platform. In this lab, you will learn how to build and deploy a simple REST service with JAX-RS and JSON-B on OpenShift 4.X (the 'X' indicates that the lab can run on any OpenShift4 version). The REST service will respond to **GET** requests made to the /LibertyProject/System/properties endpoint.

The service responds to a **GET** request with a JSON representation of the system properties, where each property is a field  in a JSON object like this:
```JSON
{
  
  "user.timezone": "Etc/UTC",
  "java.vm.specification.version": "11",
  "os.name": "Linux",
  "server.output.dir": "/opt/ol/wlp/output/defaultServer/",
  
}
```
### Introduction

When you create a new REST application, the design of the API is important. The JAX-RS APIs can be used to create JSON-RPC, or XML-RPC APIs, but it wouldn't be a RESTful service. A good RESTful service is designed around the resources that are exposed, and on how to create, read, update, and delete the resources. The service responds to **GET** requests to the **/LibertyProject/System/properties** path. The **GET** request should return a **200 OK** response that contains all of the JVM's system properties.

The platform where your application is deployed to is equally important as the design of your application/API. OpenShift provides a secure, scalable and universal way to build and deploy your application. Regardless of the infrastructure, OpenShift can run your application on private cloud, public cloud or physical machines. Although OpenShift offers multiple ways to build your application, you'll be building from your local files using a binary build process that matches close to a typical developer workflow. To learn more about OpenShift 4.X build processes, refer to [this link](https://docs.openshift.com/container-platform/4.3/builds/understanding-image-builds.html). 

# Getting Started

You should see a terminal running. In case a terminal window does not open, navigate:

**Terminal -> New Terminal**

Check you are in the **home/project** folder:

```
pwd
```
{: codeblock}

To ensure any resource created from previous lab is cleaned up, please type the command **cleanup.sh** (from any directory, the file is added to the path):
```
cleanup.sh
```
{: codeblock}


![](https://ibm.box.com/shared/static/aym3lq2tcs5mms0pgg1nsj0hxy6b7fv2.gif)

The fastest way to work through this guide is to clone [this Git repository](https://github.com/dewan-ahmed/guide-rest-intro.git) and use the maven projects that are provided inside.

```
git clone https://github.com/dewan-ahmed/guide-rest-intro.git
```
{: codeblock}

The **finish** directory in the root of this guide contains the finished application. Due to limitation of time, we suggest using **finish** directory for this exercise. The **start** directory provides a skeleton of the finished project and you can follow the steps [in this guide](https://openliberty.io/guides/rest-intro.html) to add the missing pieces of the application yourself (at a later time and on your local environment).

### Understanding a JAX-RS application

JAX-RS has two key concepts for creating REST APIs. The most obvious one is the resource itself, which is modelled as a class. The second is a JAX-RS application, which groups all exposed resources under a common path. You can think of the JAX-RS application as a wrapper for all of your resources.

Click on **File**-->**Open**-->**guide-rest-intro**-->**finish**-->**src**/**main**/**java**/**io**/**openliberty**/**guides**/**rest**/**SystemApplication.java**

The **SystemApplication** class extends the **Application** class, which in turn associates all JAX-RS resource classes in the WAR file with this JAX-RS application, making them available under the common path specified in the **SystemApplication** class. The **@ApplicationPath** annotation has a value that indicates the path within the WAR that the JAX-RS application accepts requests from.

The expectation is that when you repeat this lab yourself using the **start** directory, you'll be creating the SystemApplication class yourself. 

### Understanding the JAX-RS resource

In JAX-RS, a single class should represent a single resource, or a group of resources of the same type. In this application, a resource might be a system property, or a set of system properties. It is easy to have a single class handle multiple different resources, but keeping a clean separation between types of resources helps with maintainability in the long run.

Click on **File**-->**Open**-->**guide-rest-intro**-->**finish**-->**src**/**main**/**java**/**io**/**openliberty**/**guides**/**rest**/**PropertiesResource.java**

This resource class has quite a bit of code in it, so let's break it down into manageable chunks.

The **@Path** annotation on the class indicates that this resource responds to the **properties** path in the JAX-RS application. The **@ApplicationPath** annotation in the **SystemApplication** class together with the **@Path** annotation in this class indicates that the resource is available at the **System/properties** path.

JAX-RS maps the HTTP methods on the URL to the methods on the class. The method to call is determined by the annotations that are specified on the methods. In the application you are building, an HTTP **GET** request to the **System/properties** path results in the system properties being returned.

The **@GET** annotation on the method indicates that this method is to be called for the HTTP **GET** method. The **@Produces** annotation indicates the format of the content that will be returned. The value of the **@Produces** annotation will be specified in the HTTP **Content-Type** response header. For this application, a JSON structure is to be returned. The desired **Content-Type** for a JSON response is **application/json** with **MediaType.APPLICATION_JSON** instead of the **String** content type. Using a constant such as **MediaType.APPLICATION_JSON** is better because if there's a spelling error, a compile failure occurs.

JAX-RS supports a number of ways to marshal JSON. The JAX-RS 2.1 specification mandates JSON-Binding
(JSON-B) and JAX-B. 

The method body returns the result of **System.getProperties()** that is of type **java.util.Properties**. Since the method 
is annotated with **@Produces(MediaType.APPLICATION_JSON)**, JAX-RS uses JSON-B to automatically convert the returned object
to JSON data in the HTTP response.

The expectation is that when you repeat this lab yourself using the **start** directory, you'll be creating the PropertiesResource class yourself. 


### Understanding the server configuration

To get the service running, the Liberty server needs to be correctly configured. 

Click on **File**-->**Open**-->**guide-rest-intro**-->**finish**-->**src**/**main**/**liberty**/**config**/**server.xml**

The configuration does the following actions:

1. Configures the server to enable JAX-RS. This is specified in the **featureManager** element.
2. Configures the server to resolve the HTTP port numbers from variables, which are then specified in the Maven **pom.xml** file. This is specified in the **<httpEndpoint/>** element. Variables use the **${variableName}** syntax. 
3. Configures the server to run the produced web application on a context root specified in the **pom.xml** file. This is specified in the **<webApplication/>** element.

Take a look at the pom.xml file. Click on File-->Open-->**guide-rest-intro**-->**finish**-->**pom.xml**

The variables that are being used in the **server.xml** file are provided by the properties set in the Maven **pom.xml** file. The properties must be formatted as **liberty.var.variableName**.

# Building and running the application locally

To try out the application locally, run the following Maven goal to build the application and deploy it to Open Liberty:

```
cd guide-rest-intro/finish/
pwd
```
{: codeblock}

This should show **/home/project/guide-rest-intro/finish** and now execute:

```
mvn liberty:run
```
{: codeblock}

Wait till you see **The defaultServer server is ready to run a smarter planet** message in the logs. Click on the **Launch Application** tab at the top and enter "9080" for the port. This will take you to the OpenLiberty landing page (some images might not load properly due to the page being loaded via proxy). To view the system properties, append "/LibertyProject/System/properties" after the URL and you should be seeing a long list of parameters like below:

{
  ...
  "user.timezone": "Etc/UTC",
  "java.vm.specification.version": "11",
  "os.name": "Linux",
  "server.output.dir": "/opt/ol/wlp/output/defaultServer/",
  ...
}

For better readability, install a plug-in for viewing JSON on your browser. Remember to stop the server when you're done.
```
ctrl+c
```
{: codeblock}

# Deploying the application on OpenShift

(Assuming you have already cloned the repositoty and are under **guide-rest-intro/finish** folder)

Step 1: Generate the *.war file
```
mvn package
```
{: codeblock}

Step 2: Create a new binary build
```
oc new-build --name=rest-quicklab --binary --strategy=docker
```
{: codeblock}

Step 3: Start the binary build using current directory as binary input for the build
```
oc start-build rest-quicklab --from-dir=.
```
{: codeblock}

Step 4: Observe the build

Execute the following command to view the available builds.
```
oc get builds
```
{: codeblock}

The output will be something like this:

| NAME  | TYPE  | FROM | STATUS  | STARTED  | DURATION  |
|---|---|---|---|---|---|
| rest-quicklab-1  | Docker  | Binary@f9c9544  | Running  | 29 seconds ago  |   |

                                                   
The following command will show you the logs for this build. Make sure to specify the build name you see from previous command.

```
oc logs -f build/rest-quicklab-1
```
{: codeblock}

This logs-stream should end with **Push successful** message (the build process might take between two to three minutes to complete) and this is the indication that the image was built and has been pushed to OpenShift internal image registry.

Step 5: Create a new OpenShift app from the build
```
oc new-app rest-quicklab
```
{: codeblock}

This command creates OpenShift DeploymentConfigs and services for your application.

Step 6: Exposing the app by creating a route to your application
```
oc expose svc/rest-quicklab
```
{: codeblock}

This command ensures that your app is accessible from the internet by a public URL.

Step 7: 

```
oc get routes
```
{: codeblock}

The above command outputs the publicly accessible route to your OpenLiberty application.

Your app URL will be something like this: rest-quicklab-sn-labs-<your-userID>.sn-labs-user-sandbox-pr-a45631dc5778dc6371c67d206ba9ae5c-0000.tor01.containers.appdomain.cloud
  
Navigate to that URL (refresh the page if it didn't load on the first try) and you should see the OpenLiberty page that gets generated from the base image. Append "/LibertyProject" after the URL and you should see a page with "Welcome to your Liberty Application" message. Finally, "/LibertyProject/System/properties" subURL should show you a list of system properties from the machine the OpenLiberty server is running on. For best viewing result, you can install a JSON viewer tool to your browser.

Step 8: Troubleshooting (optional)

If your application on OpenShift is not running as expected, you can run the following commands to view the logs of pods.

```
oc get pods
```
{: codeblock}

The above command should output five pods - two of the pods are related to the environment, one application pod (e.g. rest-quicklab-1-25tgb), one build pod (e.g. rest-quicklab-1-build) and one deploy pod (rest-quicklab-1-deploy).
**Make a note of this output**. You'll need this output later if you wish to take part in the challenge.

The following command will display the logs from your application pod and the output should give you information on what might be wrong.
```
oc logs -f rest-quicklab-1-25tgb
```
{: codeblock}

Note: Name of your application pod might be different.

Step 9: Cleanup

Let's clean up the resources we just created. You can execute the following commands:
```
oc delete all -l app=rest-quicklab

oc delete builds rest-quicklab-1 

oc delete buildconfigs rest-quicklab
```
{: codeblock}

Note: Your app, buildconfigs and build names might be different. The following commands will help you find the exact names for your builds, buildconfigs and imagestreams.

```
oc get builds
oc get buildconfigs
oc get imagestreams
```
{: codeblock}

# Summary and next steps

In this lab, you learned:

1. How to build a Java/OpenLiberty REST service with JAX-RS and JSON-B
2. How to build and run the app locally
3. How to build and deploy the app on Red Hat OpenShift 4.X

**Well Done**

Nice work! You have learned how to develop a REST service in Open Liberty by using JAX-RS and JSON-B and then deployed the application on OpenShift 4.x.

Please run **cleanup.sh** in a terminal to make sure you don't run out of resources in future labs.
```
cleanup.sh
```
{: codeblock}


If you are interested in continuing on this journey, you should get a [free Kubernetes cluster](https://www.ibm.com/cloud/container-service/) and your own free [IBM Container Registry](https://www.ibm.com/cloud/container-registry).

