What Is DevOps, Anyway?
    Approach to culture, automation, and platform design aiming to provide better business value and responsiveness
    Goal: Increase speed and flexibility in delivering new features and services

Red Hat® DevOps Vision
    Red Hat culture is built on openness and transparency, has worked for more than 25 years
    Red Hat has history to guide customers through making DevOps reality
    Red Hat building tools, platforms, and infrastructure to support customers throughout DevOps journey

Technologies that enable DevOps are evolving:
    Prescriptive Platform-as-a-Service evolves into unopinionated container platforms
    Complicated cluster deployments evolve into cluster-managed self-deployments
    Platform updates that interrupt business evolve into over-the-air updates
    Custom configuration automation evolves into immutable infrastructure
    Infrastructure-as-a-Service (IaaS) evolves into cluster-driven infrastructure
    Microservices evolve into serverless computing
    Containers from Internet evolve into validated, supported runtime and middleware containers
    Bespoke infrastructure deployment consulting evolves into DevOps consulting and Red Hat Open Innovation Labs

Silos Create Inefficiency
    Developers Care About:
        Building applications
        Automating tests
        Continuous integration
        Performance tuning
        Debugging
    Operations Cares About:
        Deploying applications
        Managing applications, infrastructure
        Reliability
        Security
        Compliance

Processes below repeat for every application project

Physical                                                
    Have idea
    Get budget
    Submit hardware acquisition request
    Wait
    Get hardware
    Rack and stack hardware
    Install operating system
    Install OS patches/fix-packs
    Create user accounts
    Deploy framework/application server
    Deploy testing tools
    Test testing tools
    Code
    Configure production servers (buy if needed)
    Push to production
    Launch
    Order more servers to meet demand
    Wait
    Deploy new servers
    Etc.

Virtual
    Have idea
    Get budget
    Submit VM request
    Wait
    Deploy framework/application server
    Deploy testing tools
    Test testing tools
    Code
    Configure production VMs
    Push to production
    Launch
    Order more production VMs to meet demand
    Wait
    Deploy application to new VMs
    Etc.

Application Development with Container Platforms
    Assembly Line
    One of the key benefits that a container platform implementation such as OpenShift offers is the ability to streamline your workflow so that it looks more like this:
        Based on Agile methodologies, the organization, department, or team identifies a requirement.
        
        The group makes a commitment to implement a solution by allocating a budget and a relatively short schedule—typically, a few weeks to a few months.

        Stakeholders review and agree on tooling and process.

        Developers code the solution in the given timeframe using development environments that closely mirror the QA and production environments.

        Along the way, developers leverage the continuous integration tooling built into the framework to minimize major testing cycles prior to production.

        The solution is run through final QA and pushed to production.

        Depending on demand, the framework can elastically scale the solution.

What DevOps Is About - Collaboration and Cooperation
    Eliminating silos:
        Improves communication and consensus
        Increases agility and efficiency
        Accelerates time-to-market

Deployments
    OpenShift® deployments provide fine-grained management over applications
        Based on user-defined template called "deployment configuration"
    Many different templating forms to fit team’s style
        Examples: Templates, Helm, Kustomize
    In response to deployment configuration, deployment system creates replication controller to run application
    OpenShift is Kubernetes
        Supports Kubernetes "deployments," which are functional equivalents of OpenShift deployment configurations

    Features Provided by Deployment System
    Deployment configuration: Template for running applications
        Contains version number incremented each time new replication controller created from configuration
        Contains cause of last deployed replication controller
    Triggers that drive automated deployments in response to events
    Strategies to transition from previous version to new version
    Rollbacks to previous version in case of deployment failure
        Either manual or automatic
    Manual replication scaling and autoscaling

    Rollbacks
        Deployments allow rollbacks
            Rollbacks revert application back to previous revision
            Performed via REST API, CLI, or web console
        Deployment configurations support automatic rollback to last successful revision of configuration
            Triggered if latest template fails to deploy

    Deployment Strategy
        Defined by deployment configuration
        Determines deployment process
            During deployments, each application has different requirements
                Availability
                Other considerations
        OpenShift provides strategies to support variety of deployment scenarios
        Readiness checks determine if new pod ready for use
            If readiness check fails, deployment configuration retries until it times out

    Rolling Deployment Strategy
        Performs rolling update
        Supports life-cycle hooks for injecting code into deployment process
        Waits for pods to pass readiness check before scaling down old components
            Does not allow pods that do not pass readiness check within timeout
        Used by default if no strategy specified in deployment configuration

    Rolling Deployment Strategy Process
        Steps in rolling strategy process:
            Execute pre life-cycle hook
            Scale up new deployment by one or more pods (based on maxSurge value)
                Wait for readiness checks to complete
            Scale down old deployment by one or more pods (based on maxUnavailable value)
            Repeat scaling until new deployment reaches desired replica count and old deployment scales to zero
            Execute any post life-cycle hooks
        When scaling down, strategy waits for pods to become ready
            Decides whether further scaling would affect availability
        If scaled-up pods never become ready, deployment times out
            Results in deployment failure

    Recreate Strategy
        Has basic rollout behavior
        Supports life-cycle hooks for injecting code into deployment process
        Steps in Recreate strategy deployment:
            Execute pre life-cycle hook
            Scale down previous deployment to zero
            Scale up new deployment
            Execute post life-cycle hook

CI/CD Workflow
    Continuous integration is a software development practice that does the following:
        It verifies build integrity by checking if the source code can be pulled from the repository and built for deployment purposes. The build process may include compiling, packaging, and configuring the software.

        It validates the test results by running all of the tests created by developers in a production-like environment to verify that the source code did not break as a side effect of the commit.
        
        It checks the integration among multiple systems by using integration tests to validate all systems integration used by the software.

        And it identifies any problems and alerts all affected teams to fix the problems promptly.

    The benefits of continuous integration include the following:

        Rapid feedback: When a build is executed, the team members are immediately notified about the build’s status. This reduces the time it takes to discover and fix any new defects.

        Reduced risk: By integrating many times a day, you can reduce the risks in your project. Bugs are detected and fixed sooner. The health of the software can be measured by using unit testing and code inspection reports.

        Team ownership: With CI, there is no longer an "us" versus "them." All team members receive regular reports on the build status. This enables greater project visibility where everyone can notice trends and make effective decisions. This also increases confidence to add new features to the project, because everyone is on board with the current health of the project.

        Building of deployable software: The build process should generate software that can be deployed. Everyone should strive toward the goal of creating software that can be deployed at any time. This does not mean that you must deploy the software, but that it is a good release candidate if you want to deploy. Today, many development teams struggle with this scenario.

        Automated process: Automating the build saves time, costs, and effort. In addition, the process runs the same every time. This frees developers from having to perform repetitive processes, which allows them to focus on more high-value work.

    CI Best Practices
        Maintain code repository
        Automate build
        Make build self-testing
        Make sure everyone commits every day
        Keep build fast
        Test in production clone
        Make getting deliverables easy
        Make sure everyone can view build results
    
    Builds
        Process of transforming input parameters into resulting object
            Most often used to transform input parameters or source code into runnable image
        BuildConfig object: Definition of entire build process
        OpenShift leverages Kubernetes
            Creates containers from build images
            Pushes containers to integrated registry

    Builds and S2I
        Build Strategies
            Build system in OpenShift provides extensible support for build strategies
                Selectable strategies specified in build API
            Four primary build strategies:
                Container build
                Source-to-Image (S2I) build
                Custom build
                Pipeline build
            Object resulting from build depends on builder used to create it
                Container and S2I builds: Runnable images
                Custom builds: Whatever builder image author specified
                Pipeline builds: Jenkins pipelines executable by Jenkins Pipeline plug-in
        Source-to-Image (S2I) Build
            Source-to-Image (S2I): Tool for building reproducible container images
                Produces ready-to-run images
                    Injects application source into container image, assembles new container image
                New image incorporates base image (builder) and built source
                Ready to use with podman run command
        S2I supports incremental builds
            Reuses previously downloaded dependencies, previously built artifacts, etc.

        Some of the benefits that S2I provides include:
            Image flexibility: You can write S2I scripts to inject application code into almost any existing container image, taking advantage of the existing ecosystem.

            Speed: With S2I, the assemble process can perform a large number of complex operations without creating a new layer at each step, resulting in a fast process. In addition, you can write S2I scripts to reuse artifacts stored in a previous version of the application image, rather than having to download or build them each time the build is run.

            Patchability: S2I allows you to rebuild the application consistently if an underlying image needs a patch due to a security issue.

            Operational efficiency: By restricting build operations instead of allowing arbitrary actions, as a Dockerfile would allow, you can avoid accidental or intentional abuses of the build system.
        
            Operational security: S2I restricts the operations performed as a root user and can run the scripts as a non-root user. This eliminates the scenario in which building an arbitrary Dockerfile exposes the host system to root privilege escalation, which can be exploited by a malicious user because the entire container build process is run by a user with container privileges.

            Ecosystem: S2I encourages a shared ecosystem of images where you can leverage best practices for your applications.

            Reproducibility: Produced images can include all inputs including specific versions of build tools and dependencies. This ensures that the image can be reproduced precisely.

        Custom Build
            Custom build strategy: You define specific builder image responsible for entire build process
                Using own builder image allows you to customize build process
            Custom builder image is plain OCI-compliant container image embedded with build process logic
                Examples: Building RPMs or base container images

        Image Stream
            Comprises OCI-compliant container images identified by tags
            Presents single virtual view of related images
                Similar to container image repository
            May contain images from any of following:
                Image repository in integrated OpenShift Container Platform registry
                Other image streams
                Container image repositories from external registries

            New images added to associated image stream
            Builds and deployments watch image stream
                Receive notifications when new images added
                React by performing build or deployment
            Example: If deployment uses certain image and new version of image created, deployment performed automatically

    Pipelines
        Jenkins jobs enabled by Pipeline plug-in
            Built with simple text scripts that use Pipeline DSL based on Groovy
        Enable one script to address all steps in complex workflow
        Leverage power of multiple steps to execute simple and complex tasks per user-established parameters
        Build code, orchestrate work required to drive applications from commit to delivery

        Enable you to define entire application life cycle
        Support continuous integration/continuous deployment (CI/CD)
        Pipeline plug-in built with requirements for flexible, extensible, script-based CI/CD workflow

        Pipeline Characteristics
            Durable
                Survive planned and unplanned restarts of Jenkins master
            Pausable
                Optionally stop and wait for human input or approval before completing jobs
            Versatile
                Support complex real-world CD requirements
                    Ability to fork or join
                    Ability to loop
                    Ability to work in parallel with other pipelines
            Extensible
                Plug-in supports custom extensions to its DSL, multiple integration options with other plug-ins

            Pipeline Build
                Strategy allows developers to define Jenkins pipeline for execution by Jenkins Pipeline plug-in
                Build started, monitored, and managed by OpenShift same as other build types
            
            Pipeline workflows defined in Jenkinsfile
                Embedded directly in build configuration
                Or supplied in Git repository and referenced by build configuration

                