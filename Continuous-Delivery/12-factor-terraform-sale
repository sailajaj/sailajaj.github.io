The Twelve Factor App is a Software as a Service (SaaS) design methodology created by Heroku. The idea is that in order to be really suited to SaaS and avoid problems with software erosion — where over time an application that’s not updated gets to be out of sync with the latest operating systems, security patches, and so on — an app should follow these 12 principles:

    Codebase One codebase tracked in revision control, many deploys
    Dependencies Explicitly declare and isolate dependencies
    Config Store config in the environment
    Backing services Treat backing services as attached resources
    Build, release, run  Strictly separate build and run stages
    Processes  Execute the app as one or more stateless processes
    Port binding  Export services via port binding
    Concurrency  Scale out via the process model
    Disposability  Maximize robustness with fast startup and graceful shutdown
    Dev/prod parity  Keep development, staging, and production as similar as possible
    Logs  Treat logs as event streams
    Admin processes Run admin/management tasks as one-off processes

1. Codebase:
With container orchestration the revision control is tracked in the docker file specifying the version to be run.  Image built from Versioned code repos are stored by their version in Docker Hubs or Nexus.  A Pod declares the particular image they intent to run.  In this way we can have multiple deploys of the same versioned code base; or have different versions of the code base deployed at the same time.  However, each code base has a one-to-one correspondence to a version.

spec:
       containers:
       - name: AcctApp
         image: acctApp:v3

2. Dependencies: 
An application should be self-contained.  All dependencies should be explicitly stated.  There should not be any implicit assumptions on the availability of a software in the OS env (ex: dependency on curl should be explicitly stated).  This also includes ensuring that the service is not compromised by conflicting libraries installed on the host.
Python allows us to use pip install for dependencies and VirtualEnv to contain these dependencies within the env.
For applications that are modularized and depend on other components, such as an HTTP service and a log fetcher, Kubernetes provides a way to combine all of these pieces into a single Pod, for an environment that encapsulates those pieces appropriately.

3. Config
Configs : Credentials,  resource handlers, canonical hostnames,..
12 factor requires strict separation of config from code. Config varies substantially across deploys, code does not.

Kubernetes enables you to specify environment variables in manifests via the Downward API, but as these manifests themselves do get checked int source control, that’s not a complete solution.  The secret keys are exposed and checked into a repo.  Instead the content can be retrieved with Kubernetes ConfigMap or Secret, which can be kept separate from the application. Diogo Mónica points to a tool called Keywhiz you can use with Kubernetes, creating secure secret storage.

Separate config directories by the env : sandbox, dev, prod.

4.  Configuring Backing Service
The code for a twelve-factor app makes no distinction between local and third party services. To the app, both are attached resources, accessed via a URL or other locator/credentials stored in the config. A deploy of the twelve-factor app should be able to swap out a local MySQL database with one managed by a third party (such as Amazon RDS) without any changes to the app’s code. Likewise, a local SMTP server could be swapped with a third-party SMTP service (such as Postmark) without code changes. In both cases, only the resource handle in the config needs to change.

If you needed to change to, say, PostgreSQL or a remotely hosted MySQL server, you could create a new container image, update the Pod definition, and restart the Pod (or more likely the Deployment or StatefulSet managing it).  Similarly, if you’re storing credentials or address information in environment variables backed by a ConfigMap, you can change that information and replace the Pod.

Note that both of these examples assume that though you’re not making any to the source code (or even the container image for the main application) you will need to replace the Pod; 

5.  Build, Release and Run
Build stage: Only build code
Release : Add config to Build
Run : Deploy 

Releases should be identifiable.  You should be able to say, ‘This deployment is running Release 1.14 of this application” or something similar, the same way we say we’re running “the OpenStack Ocata release” or “Kubernetes 1.6”.  They should also be immutable; any changes should lead to a new release.  

 All of this is so that when the app is running, that “run” process can be completely automated. Twelve factor apps need to be capable of running in an automated fashion because they need to be capable of restarting should there be a problem.

6. Processes

Twelve-factor processes are stateless and share-nothing. Any data that needs to persist must be stored in a stateful backing service, typically a database.

The memory space or filesystem of the process can be used as a brief, single-transaction cache. For example, downloading a large file, operating on it, and storing the results of the operation in the database. The twelve-factor app never assumes that anything cached in memory or on disk will be available on a future request or job – with many processes of each type running, chances are high that a future request will be served by a different process. Even when running only one process, a restart (triggered by code deploy, config change, or the execution environment relocating the process to a different physical location) will usually wipe out all local (e.g., memory and filesystem) state.

 7. Port Binding

The twelve-factor app is completely self-contained and does not rely on runtime injection of a webserver into the execution environment to create a web-facing service. The web app exports HTTP as a service by binding to a port, and listening to requests coming in on that port.

In a local development environment, the developer visits a service URL like http://localhost:5000/ to access the service exported by their app. In deployment, a routing layer handles routing requests from a public-facing hostname to the port-bound web processes.  This is typically implemented by using dependency declaration to add a webserver library to the app

8. Concurrency:

The process model truly shines when it comes time to scale out. The share-nothing, horizontally partitionable nature of twelve-factor app processes means that adding more concurrency is a simple and reliable operation. The array of process types and number of processes of each type is known as the process formation.  

When you’re writing a twelve-factor app, make sure that  you provide a way for it to be scaled out, rather than scaled up. That means that in order to add more capacity, you should be able to add more instances rather than more memory or CPU to the machine on which the app is running. Note that this specifically means being able to start additional processes on additional machines, which is, fortunately, a key capability of Kubernetes. 

9. Disposabilty

Processes should strive to minimize startup time. Ideally, a process takes a few seconds from the time the launch command is executed until the process is up and ready to receive requests or jobs.

With Kubernetes we can perform green-blue deployments for easy disposabilty.

10, Dev/prod parity 
Keep development, staging, and production as similar as possible 

11. Logs

Logs are the stream of aggregated, time-ordered events collected from the output streams of all running processes and backing services.  During local development, the developer will view this stream in the foreground of their terminal to observe the app’s behavior.n staging or production deploys, each process’ stream will be captured by the execution environment, collated together with all other streams from the app, and routed to one or more final destinations for viewing and long-term archival. 

=> Key to developers requiring prod access.  If logs are streamed to a central point then there is no compelling reason for prod access.

12.  Admin
Run admin/management tasks as one-off processes

 

---------

AA-Streaming Build process 

Current build pipeline is in the diagram below. 
1. Resources (dependencies for container) are loaded into GCS bucket
2. Jenkins retrieves these versioned resource files (see gcsResourcesBucketPath below)
3. Jenkins Executes the grade command scripts to generate the image

 Reasoning behind: Build Dataflow Template

Dataflow templates use runtime parameters to accept values that are only available during pipeline execution. To customize the execution of a templated pipeline, you can pass these parameters to functions that run within the pipeline (such as a DoFn).  To create a template from your Apache Beam pipeline, you must modify your pipeline code to support runtime parameters:

    Use ValueProvider for all pipeline options that you want to set or use at runtime.
    Call I/O methods that accept runtime parameters wherever you want to parameterize your pipeline.
    Use DoFn objects that accept runtime parameters.

Then, create and stage your template.



 

 

 


    #!groovy
    @Library("dsv") _


    gitFlow {

        stage("Gradle build") {
            sh("./gradlew clean build")
            junit(allowEmptyResults: true, testResults: "**/build/test-results/test/*.xml")
        }

        stage("Prepare Variables") {
            gitFlow.whenMaster {
                GCS_ACCOUNT = "iodine-jenkins-sa-prod"
                GOOGLE_PROJECT_ID = "prd-dumbo-00-808d68"
                IODINE_ENVIRONMENT = "PROD"
                prefix = "prd-"
            }

            gitFlow.whenDevelop {
                GCS_ACCOUNT = "iodine-jenkins-sa-dev"
                GOOGLE_PROJECT_ID = "dev-dumbo-00-0cef88"
                IODINE_ENVIRONMENT = "DEV"
                prefix = "dev-"
            }

            gitFlow.whenFeature {
                GCS_ACCOUNT = "iodine-jenkins-sa-sbx"
                GOOGLE_PROJECT_ID = "sbx-dumbo-00-a0ad9e"
                IODINE_ENVIRONMENT = "SBX"
                prefix = "sbx-"
            }
        }

        gitFlow.deployStage("Build Dataflow Template") {

            withCredentials([file(credentialsId: GCS_ACCOUNT, variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                gcsResourcesBucketPath = "gs://${prefix}streaming-resource"
                gcsDataflowBucketPath = "gs://${prefix}streaming-dataflow"

                sh "gcloud auth activate-service-account --key-file=${GOOGLE_APPLICATION_CREDENTIALS}"
                sh "gcloud config set account iodine-jenkins-sa@${GOOGLE_PROJECT_ID}.iam.gserviceaccount.com"
                sh "gcloud config set core/project ${GOOGLE_PROJECT_ID}"

                sh """./gradlew prepareTemplate \
                    -PgcpProject=${GOOGLE_PROJECT_ID} \
                    -PstagingLocation=${gcsResourcesBucketPath} \
                    -PtempLocation=${gcsDataflowBucketPath} \
                    -PtemplateLocation=${gcsResourcesBucketPath} \
                    -PbuildVersion=${version}
                """

                archiveArtifacts '*_template'
                sh "rm *_template"

            }
        }
    }


