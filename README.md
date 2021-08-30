# Getting Started with OCI DevOps

This is an example project using Node.js with the Express [getting started generator](https://expressjs.com/en/starter/generator.html). With the [OCI DevOps service](https://www.oracle.com/devops/devops-service/) and this project, you'll be able to build, test and deploy this application to Oracle Container Engine for Kubernetes (OKE).

In this example, you'll build a container image of this Express getting started app, and deploy your built container to the OCI Container Registry, then deploy the getting started app to Oracle Container Engine for Kubernetes (OKE) all using the OCI DevOps service!

Let's go!

## Download the repo
The first step to get started is to download the repository to your local workspace

```shell
git clone git@github.com:jonschreiber/oci-devops-node-express.git
cd oci-devops-node-express
```

## Install and run the Express example
Open a terminal and test out the simple Express example - a web app that returns a "Welcome to Express" page

1. Install Node 12 and NPM: https://docs.npmjs.com/downloading-and-installing-node-js-and-npm 
1. Build the app: `npm install`
1. Run tests: `npm test`
1. Run the app: `npm start`
1. Verify the app locally, open your browser to [http://localhost:3000/](http://localhost:3000/) or whatever port you set, if you've changed the local port

## Build a container image for the app
You can locally build a container image using docker (or your favorite container image builder), to verify that you can run the app within a container

```
docker build --pull --rm -t node-express-getting-started -f DOCKERFILE .
```

Verify that your image was built, with `docker images` 

Next run your local container and confirm you can access the app running in the container
```
docker run --rm -d -p 3000:3000 --name node-express-getting-started node-express-getting-started:latest
```

And open your browser to [http://localhost:3000/](http://localhost:3000/)

# Build and test the app in OCI DevOps
Now that you've seen you can locally build and test this app, let's build our CI/CD pipeline in OCI DevOps

## Create your Git repo

1. [Create a DevOps project](https://docs.oracle.com/en-us/iaas/Content/devops/using/devops_projects.htm), or use an existing project
1. Create a Code Repository in your DevOps project
1. Add the new Code Repository as a remote to your local git repo
```
git remote add devops ssh://devops.scmservice.us-ashburn-1.oci.oraclecloud.com/namespaces/MY-TENANCY/projects/MY-PROJECT/repositories/MY-REPO
```
1. View the Getting Started Guide to connect to your Code Repository via https or ssh

## Setup your Build Pipeline
Create a new Build Pipeline to build, test and deliver artifacts from a recent commit

## Managed Build stage
In your Build Pipeline, first add a Managed Build stage
1. The Build Spec File Path is the relative location in your repo of the build_spec.yaml . Leave the default, for this example
1. For the Primary Code Repository choose your Code Repository you created above
    1. The Name of your Primary Code Repository is used in the build_spec.yaml. In this example, you will need to use the name `node_express` for the build_spec.yaml instructions to acess this source code
    1. Select the `main` branch

## Create a Container Registry repository
Create a [Container Registry repository](https://docs.oracle.com/en-us/iaas/Content/Registry/Tasks/registrycreatingarepository.htm) for the `node-express-getting-started` container image built in the Managed Build stage. 
1. You can name the repo: `node-express-getting-started`. So if you create the repository in the Ashburn region, the path is iad.ocir.io/TENANCY-NAMESPACE/node-express-getting-started
1. Set the repostiory access to public so that you can pull the container image without authorization, from OKE. Under "Actions", choose `Change to public`.


## Create a DevOps Artifact for your container image repository
The version of the container image that will be delivered to the OCI repository is defined by a [parameter](https://docs.oracle.com/en-us/iaas/Content/devops/using/configuring_parameters.htm) in the Artifact URI that matches a Build Spec exported variable or Build Pipeline parameter name.

Create a DevOps Artifact to point to the Container Registry repository location you just created above. Enter the information for the Artifact location:
1. Name: node-express-getting-started container
1. Type: Container image repository
1. Path: `iad.ocir.io/TENANCY-NAMESPACE/node-express-getting-started`
1. Replace parameters: Yes

Next, you'll set the container image tag to use the the Managed Build stage `exportedVariables:` name for the version of the container image to deliver in a run of a build pipeline. In the build_spec.yaml for this project, the variable name is: `BUILDRUN_HASH`
```
  exportedVariables:
    - BUILDRUN_HASH
```

Edit the DevOps Artifact path to add the tag value as a parameter name.
1. Path: `iad.ocir.io/TENANCY-NAMESPACE/node-express-getting-started:${BUILDRUN_HASH}`

## Add a Deliver Artifacts stage
Let's add a **Deliver Artifacts** stage to your Build Pipeline to deliver the `node-express-getting-started` container to an OCI repository. 

The Deliver Artifacts stage **maps** the ouput Artifacts from the Managed Build stage with the version to deliver to a DevOps Artifact resource, and then to the OCI repository.

Add a **Deliver Artifacts** stage to your Build Pipeline after the **Managed Build** stage. To configure this stage:
1. In your Deliver Artifacts stage, choose `Select Artifact` 
1. From the list of artifacts select the `node-express-getting-started container` artifact that you created above
1. In the next section, you'll assign the  container image outputArtifact from the `build_spec.yaml` to the DevOps project artifact. For the "Build config/result Artifact name" enter: `output01`


# Run your Build in OCI DevOps

## From your Build Pipeline, choose `Manual Run` 
Use the Manual Run button to start a Build Run

Manual Run will use the latest commit to your Primary Code Repository, if you want to specify a specific commit, you can optionally make that choice for the Primary Code Repository in the dropdown and selection below the Parameters section.


## Connect your Code Repository to your Build Pipeline
To automatically start your Build Pipeline from a commit to your Code Repository, navigate to your Project and create a Trigger. 

A Trigger is the resource to 
filter the events from your Code Repository and on a matching event will start the run of a Build Pipeline.

## Push a commit to your DevOps Code Repository
Test out your Trigger by editing a file in this repo and pushing a change to your DevOps code repository.

# Connect your Build Pipeline with a Deployment Pipeline

For CI + CD: continous integration with a Build Pipeline and continuous deployment with a Deployment Pipeline, you can add a **Trigger Deployment** stage as the last step of your Build Pipeline

## Trigger Deployment stage
After the latest version of the container image is delivered to the Container Registry via the **Deliver Artifacts** stage, we can start a deployment to an OKE cluster

1. Add stage to your Build Pipeline
1. Choose a **Trigger Deployment** stage type
1. Choose `Select Deployment Pipeline` to choose the Deployment Pipeline that will apply the Kubernetes Manifest: gettingstarted-manifest.yaml in this repo to your OKE cluster

From the Deployment Pipeline you selected, you can confirm the parameters of that pipeline in the Deployment Pipeline details.



# Make this your own
Fork this repo from Github and make changes if you want to play around with the sample app, the OCI DevOps build configuration, and the k8s manifest.