# Create a Production Kubernetes Cluster with OpenShift and Enforce RBAC for Cluster Security

This project provides a comprehensive walkthrough on deploying a production-ready Kubernetes application environment using Red Hat OpenShift. 

It specifically focuses on implementing robust security measures through Role-Based Access Control (RBAC). 

While standard Kubernetes provides the orchestration engine, OpenShift offers a simplified platform with advanced tools for managing and securing containerized workloads. 

This guide uses the Red Hat OpenShift Developer Sandbox for a 30-day shared cluster experience.



## ARCHITECTURAL CONCEPTS

The project demonstrates key Kubernetes RBAC components:

ServiceAccount: Provides an identity for processes running in your Pods to communicate with the API server.

Role: Defines a set of permissions (verbs) applicable within a specific namespace.

RoleBinding: Grants the permissions defined in a Role to a subject (User, Group, or ServiceAccount).


### PDF GUIDE: [PRODUCTION KUBERNETES CLUSTER WITH OPENSHIFT.pdf](https://github.com/user-attachments/files/32320974/CREATE.YOUR.FIRST.PRODUCTION.KUBERNETES.CLUSTER.WITH.OPENSHIFT.pdf)

### WATCH VIDEO WALKTHROUGH HERE: https://youtu.be/25PNfppwrBM



## PREREQUISITES

Internet access and a web browser.

A Red Hat account (registration instructions included).

Local terminal environment (Linux/macOS/WSL) with curl and tar installed.


## REPOSITORY ARTIFACTS

This repository contains the following Kubernetes manifest files required for the RBAC walkthrough:

`serviceaccount.yml`: Defines the ServiceAccount resource.

`role.yml`: Defines the Role resource with specific resource permissions (Pods, Deployments).

`binding_role.yml`: Binds the ServiceAccount to the Role.

Additional examples (`cluster_role.yml`, `cluster_binding_role.yml`, `new_role.yml`, `new_binding_role.yml`) are discussed in the guide but not used in the core sandbox walkthrough.



## STEP-BY-STEP INSTRUCTIONS

### Step 1: Initialize OpenShift Developer Sandbox (30 Days)

1) Navigate to cite: https://developers.redhat.com/developer-sandbox

2) Click Start your sandbox for free.

3) On the login page, click Register for a Red Hat account and provide your details (Name, Email, etc.).

4) Verify your email address, log in, and locate OpenShift from the product catalog. Click Try it.



### Step 2: Explore Projects and Namespaces

1) From the OpenShift dashboard, click Home -> Projects. 

2) Note the default namespaces available to you (usually your-username-dev).


### Step 3: Verify Permissions
1) Locate Workloads in the menu to see standard cluster services.

2) Observation: If you attempt to access services outside your designated namespaces, you will observe "permission denied" errors. 

3) Your primary workspace is constrained to your assigned projects.


### Step 4: Access Designated Namespace

1) Return to Home -> Projects.

2) Click on your username-dev namespace.

3) Observation: You have permissions to create and manage workloads within this specific boundary.


### Step 5: Retrieve Login Command

1) At the top right of the screen, click your Username -> Copy Login Command.

2) A new page appears. Click Display Token.

3) Copy the provided login token command.


### Step 6: Configure Terminal and Login via CLI

Before logging in, you must install the OpenShift CLI tool (oc). Open your local terminal.

1) Download and install oc:

   #### Download the client tools
   <PRE>curl -LO https://mirror.openshift.com/pub/openshift-v4/clients/ocp/stable/openshift-client-linux.tar.gz</PRE>

   #### Confirm download
   <PRE>ls</PRE>

   #### Extract the archive
   <PRE>tar -xvzf openshift-client-linux.tar.gz</PRE>

   #### Move the oc binary to your path
   <PRE>sudo mv oc /usr/local/bin/</PRE>

   #### Optional: If you already have kubectl, rename the OpenShift version to avoid conflicts
   <PRE>sudo mv kubectl /usr/local/bin/kubectl-oc-bundled 2>/dev/null</PRE>


2) Login to your OpenShift cluster:

   Use the token command copied from Step 5:

   #### Example command (Replace token and server with your displayed values)
   <PRE>oc login --token=sha256~Cv5dKX6Oj2nE_wFV-5rWgeh1IplRnd3nDfi8x25NuEA --server=https://api.rm1.0a51.p1.openshiftapps.com:6443</PRE>


3) List and switch projects:

   #### This will throw an error because you do not have permission to access resources in other namespaces or workspaces within the cluster
   <PRE>kubectl get pods</PRE>

   #### List accessible projects
   <PRE>oc projects</PRE>

   #### Switch to your dev project
   <PRE>oc project your-username-dev</PRE>
   #### Example: oc project chinedu-dev


### Step 7: Apply RBAC in Designated Namespace

You will now create resources and restrict access based on RBAC policies. 
Ensure you are still inside your username-dev project or namespace.


1) Create ServiceAccount:

   #### create a service account file
   <PRE>vim serviceaccount.yml</PRE>

   #### create service account called service-resource
   <PRE>kubectl apply -f serviceaccount.yml</PRE>

   #### Ask Kubernetes if the service account resource you created can access pods from
   #### the namespace you are currently on. The response should be NO
   <PRE>kubectl auth can-i --as system:serviceaccount:<name-of-your-namespace>:<name-of-your-service-account> get pods -n <name-of-your-namespace></PRE>

   <PRE>kubectl auth can-i --as system:serviceaccount:techdealer1000-dev:service-resource get pods -n techdealer1000-dev</PRE>


2) Create Role:

   #### before you create a role, check the permissions your namespace has
   #### scroll down to where you have OpenShift and see the permissions you have
   #### the permissions your namespace has will determine the type of access you can grant
   #### if you create an OpenShift cluster, you will have all the permissions because
   #### you are not sharing it with other users, and you can grant wildcards
   <PRE>kubectl auth can-i --list --namespace=:<your-name-space-name></PRE>
   <PRE>kubectl auth can-i --list --namespace=:techdealer1000-dev</PRE>

   #### create a role file
   <PRE>vim role.yml</PRE>

   #### create role called role-resource
   <PRE>kubectl apply -f role.yml</PRE>

   #### Ask Kubernetes if the service account resource you created can now access pods from
   #### the namespace you are currently on. The response should still be NO because even if
   #### you created a role, you did not bind that role to the service account to assume the role
   <PRE>kubectl auth can-i --as system:serviceaccount:<name-of-your-namespace>:<name-of-your-service-account> get pods -n <name-of-your-namespace></PRE>
   <PRE>kubectl auth can-i --as system:serviceaccount:techdealer1000-dev:service-resource get pods -n techdealer1000-dev</PRE>


3) Create Role Binding:

   #### create a role binding file
   <PRE>vim binding_role.yml</PRE>

   #### create the role binding resource called role-binding-resource
   <PRE>kubectl apply -f binding_role.yml</PRE>

   #### Ask Kubernetes if the service account resource you created can now access pods from
   #### the namespace you are currently on. The response should be YES this time because you
   #### have bound the role to the service account, and it can now assume the role
   <PRE>kubectl auth can-i --as system:serviceaccount:<name-of-your-namespace>:<name-of-your-service-account> get pods -n <name-of-your-namespace></PRE>
   <PRE>kubectl auth can-i --as system:serviceaccount:techdealer1000-dev:service-resource get pods -n techdealer1000-dev</PRE>

   Checking Additional Permissions:

   #### Check that the service account can create pods in your accessed namespace.
   #### This should respond with yes
   <PRE>kubectl auth can-i --as system:serviceaccount:<name-of-your-namespace>:<name-of-your-service-account> get pods -n <name-of-your-namespace></PRE>
   <PRE>kubectl auth can-i --as system:serviceaccount:techdealer1000-dev:service-resource create pods -n techdealer1000-dev</PRE>

   #### Check that the service account can create deployments in your accessed namespace.
   #### This should respond with yes
   <PRE>kubectl auth can-i --as system:serviceaccount:<name-of-your-namespace>:<name-of-your-service-account> get pods -n <name-of-your-namespace></PRE>
   <PRE>kubectl auth can-i --as system:serviceaccount:techdealer1000-dev:service-resource create deployments -n techdealer1000-dev</PRE>



### Step 8: Grant Access To Other Namespaces in Your Openshift Cluster & Enforce RBAC

1) Check Access across Namespaces:

   #### list all the projects or namespaces you can access in your free OpenShift 30-day cluster
   <PRE>oc projects</PRE>

   #### Check if the service account resource can create pods and deployments in the sandbox-shared-models namespace. You should get a NO response.
   <PRE>kubectl auth can-i --as system:serviceaccount:<name-of-your-namespace>:<name-of-your-service-account> get pods -n <name-of-another-namespace-you-can-access></PRE>
   <PRE>kubectl auth can-i --as system:serviceaccount:techdealer1000-dev:service-resource create pods -n sandbox-shared-models</PRE>
   <PRE>kubectl auth can-i --as system:serviceaccount:techdealer1000-dev:service-resource create deployments -n sandbox-shared-models</PRE>

   #### Check for the openshift-virtualization-os-images namespace. You should get NO response
   <PRE>kubectl auth can-i --as system:serviceaccount:techdealer1000-dev:service-resource get pods -n openshift-virtualization-os-images</PRE>
   <PRE>kubectl auth can-i --as system:serviceaccount:techdealer1000-dev:service-resource create deployments -n openshift-virtualization-os-images</PRE>


2) Edit Role and Role Binding Files (for Multi-Namespace Access):

   #### list all the OpenShift projects you can access
   #### copy their names in a separate sheet, you will need them going forward
   <PRE>oc projects</PRE>

   #### edit the role.yml file
   <PRE>vim role.yml</PRE>

   #### apply your changes
   <PRE>kubectl apply -f role.yml</PRE>

   #### Use kubectl to get namespaces from your free OpenShift cluster. Also observe that you cannot list namespaces or projects using kubectl
   <PRE>kubectl get namespaces</PRE>

   #### edit the role binding file
   <PRE>vim binding_role.yml</PRE>

   #### apply your changes
   <PRE>kubectl apply -f binding_role.yml</PRE>


3) Individual Namespace Access Testing:

   #### list all the OpenShift projects you can access
   <PRE>oc projects</PRE>

   #### switch to this namespace or project
   #### list all the permissions you have in this namespace
   <PRE>oc project sandbox-shared-models</PRE>
   <PRE>kubectl auth can-i --list -n sandbox-shared-models</PRE>

   #### switch to this namespace or project
   #### list all the permissions you have in this namespace
   <PRE>oc project openshift-virtualization-os-images</PRE>
   <PRE>kubectl auth can-i --list -n openshift-virtualization-os-images</PRE>


### Step 9: Delete The Roles & Resources You Have Created

This is optional.

#### switch to this namespace or project
#### this should be your active namespace

<PRE>oc project techdealer1000-dev</PRE>

#### list the roles you have in this namespace
<PRE>kubectl get roles -n <your-namespace-name></PRE>
<PRE>kubectl get roles -n techdealer1000-dev</PRE>

#### delete the role resource
<PRE>kubectl delete role role-resource -n <your-namespace-name></PRE>
<PRE>kubectl delete role role-resource -n techdealer1000-dev</PRE>

#### get the role bindings you created in this namespace
<PRE>kubectl get rolebindings -n <your-namespace-name></PRE>
<PRE>kubectl get rolebindings -n techdealer1000-dev</PRE>

#### delete the role bindings you created
<PRE>kubectl delete rolebinding role-binding-resource -n <your-namespace-name></PRE>
<PRE>kubectl delete rolebinding role-binding-resource -n techdealer1000-dev</PRE>

#### get the service account you created in this namespace
<PRE>kubectl get serviceaccount -n <your-namepace-name></PRE>
<PRE>kubectl get serviceaccount -n techdealer1000-dev</PRE>

#### delete the service account resource you created
<PRE>kubectl delete serviceaccount service-resource -n <your-namespace-name></PRE>
<PRE>kubectl delete serviceaccount service-resource -n techdealer1000-dev</PRE>

#### check they are all gone
<PRE>kubectl get roles, rolebindings, serviceaccounts -n techdealer1000-dev</PRE>

