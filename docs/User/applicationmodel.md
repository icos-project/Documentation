# Application Descriptor

Disclaimer:the application descriptor definition is evolving and may change as new requirements their functionalities 
are identified.
The ICOS Application definition manifest file is a yaml file which has two parts:

* **Application Header** 
* **Manifests lists**


## Application Header
The Application Header is an ICOS specific information to describe the application architecture, 
determine its distribution, and optionally provides additional information related to the execution 
policies as well as telemetry data specifications. 
The information provided in the header is as follows:

* **Name and Description**: Every application definition starts with a name and a description 
in order to identify and describe the application.

* **Components**: Applications consist of different  components, list the components, 
each identified by their name and manifest. There could be one or more than one manifest in a 
single component and each manifest contains all the information required to deploy it. Each component is deployed 
a **single resource**, which can be a node or a full cluster. 
And all manifests belonging to the same component will be deployed together.

There are other optional fields such as telemetry, policies and matching that need to be defined 
and described later.

As an example, here is the **Application Header** used for the Demo-1. 
In this example, we have two components named *ffmpeg* (video storage) and *mjpeg* (video streaming). 

The first component named *ffmpeg* consists of a single manifest and we can call it 
**comp1-manifest1**.The second component named *mjpeg* further contains two manifests. 
The first manifest named *mjpeg-pod* we can call it **comp2-manifest1** and the second manifest 
is named *mjpeg-service* we can call it **comp2-manifest2**, the two manifests will be deployed on the same target node (or cluster). 

**Application Header**

<figure markdown="1">
![Application Header Example](../../assets/icons/appheader.png){: style="width:390px;"}
</figure>
<p style="text-align: center;font-size:13px;color:blue;">Fig.1: Application Header example</p>

**1. Manifests**

All the manifests required for deploying the application are written in Kubernetes syntax for this 
iteration. We are currently extending it to support also doker compose and other deployment specifications.

So, following the Application Header, the list of manifests are found:

* **comp1-manifest1**

<figure markdown="1">
![comp1-manifest1](../../assets/icons/comp1.png){: style="width:390px;"}
</figure>
<p style="text-align: center;font-size:13px;color:blue;">Fig.2: comp1-manifest1</p>

* **comp2-manifest1**

<figure markdown="1">
![comp2-manifest1](../../assets/icons/comp2_1.png){: style="width:390px;"}
</figure>
<p style="text-align: center;font-size:13px;color:blue;">Fig.3: comp2-manifest1</p>

* **comp2-manifest1**

<figure markdown="1">
![comp2-manifest1](../../assets/icons/comp2_2.png){: style="width:390px;"}
</figure>
<p style="text-align: center;font-size:13px;color:blue;">Fig.4: comp2-manifest2</p>
