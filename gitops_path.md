# Path to GitOps

## What is GitOps

### GitOps Principles

#### Declarative

> A system managed by GitOps must have its desired state expressed declaratively

1. _system_ is defined as one or more runtime environments consisting of resources, agents and policies controlling repositories, deployments and runtimes.

2. _desired state_ is the way that you represent the way you want the system to work.

3. the _desired state_ must be **declarative**. This state is stored as a set of declarations.

#### Versioned and Immutable

> Desired stat eis stored in a way that enforces immutability and versioning that retains a complete version history.

Git store and tracks each change without altering previous versions. Anything that complies with this principle can be used in GitOps (S3 storage, for example).

#### Pulled Automaticaly

> Software agents automatically pull the desired state declarations from the source

Although triggering changes and updates via webhooks or other events is a valid way to automate builds, **it is not GitOps**. GitOps software agents (GitOps controllers) **checks the desired state by pulling** decclarations from the **state store** at **regular intervals**, which means _polling_ and _pulling_.

> In GitOps there is no webhook that needs to be hit. Instead, there is a **reconciliation loop**

#### Continuously Reconciled

> Software agents continuously observe actual system state and attempt to apply the desired state

- this principle **mirrors the functions of the Kubernetes controllers** but Gitops applies it to a **whole application infrastructure stack instead of just one object**.

As the desired state is pulled from configuration information that is versioned and stored in an immutable storage system, if **there is a difference** between the running and desired states, they are reconcilied by changing the running state to match the desired state.

> Continuous reconciliation distinguises GitOps from traditional CI/CD where automation is generally driven by pre-set triggers. **GitOps triggers reconciliation whenever there is a divergence.**

### GitOps and CI/CD

CI/CD is a DevOps practice that delivers applications at frequent intervals to internal or external customers by automating stages of applicatoin development. The main conceptos attributed to CI/CD are continuous integration, continuous delivery and continuous deployment.

#### Traditional CI/CD

CI/CD is event-driven. CI/CD is **linear** basing each stage on previous ones. That's why the term commonly used for the CI/CD build/test/deploy process is a **pipeline**. It can provide integracion, delivery and deployment in a continuous stream of releases

- Continuous integration
    1. Build
    2. Test
    3. Security Check
    4. Release
- Continuous Delivery
    5. Deploy Stage
    6. Deploy Prod

**Continuous Integration** *builds* and *tests* new code changes and merges them into a shared repository. Is the solution to the problem of having too many branches of an application.

**Continuous delivery** refers to automating releases of changes to the dev/stagging and pre-production environments. The changes can thenm with the approval of the operations tema or release managers, get deployed to production. This aspect of CD is an answer to the problem of poor visibility and communication between development and business teams.

#### Where does GitOps fit in CI/CD

GitOps **does not replace** CI/CD, but **participates in it**. GitOps focuses on the CD aspect (deployment and delivery). Now, instead of having deployment imperative in CI/CD platform, it can be declarative based on a **source of truth.** **Based on a reconciliation loop,** the GitOps controller makes changes to the cluster by deploying new instances, once those changes have been committed to the state store.

Under GitOps, the CI process just needs either to make a commit directly to your state store, or mark that the change is ready for deployment in a Git repository.

If you are in GitHub, the recommended way to trigger a deployment is by submitting a **pull request**; in GitLab it is by sumiting a merge request.

#### Operations via Pull Request

The idea of GitOps is to apply the same workflow to any change that goes into infrastructure and applications.

## Tools of the Trade

### Infrastructure as Code

Infrastructure as code is a method for managing, configuring and updating the entire software infrastructure at a datacenter using configuration files that are both machine-readable and human-readable (usually YAML format).

IaC lets administrators leave behind manual processes to manage every component from the bare metal servers or virtual machines by writing files.

### Argo CD

Argo CD was written with GitOps in mind to deliver changes t o Kubernetes cluster at massive scale.

Argo CD works with raw YAML stored in a Git repository and using the apply functionality in Kubernetes.

Argo CD can also work with Helm and Kustomize to render the YAML produced by those tools before applying them to the Kubernetes Cluster.

Argo CD is part of a larger ecosystem called Argo Project. Tolls also includes Argo Workflows, Argo Rollouts, and Argo Events.

### Other Tools

- Flux: Performs many of the same functions as Argo CD. But the main difference is that flux uses Helm Golang package.
- Open Cluster Management (OCM): Project used by Red Hat for Kubernetes. Goes beyond just applying YAML and detecting drift, It also maanages the lifecycle of your Kubernetes cluster. The idea is **multicluster management from a single pane of glass**. You can also manage the lifecycle of those clusters, from creation to destruction to manage clusters with IaC as well.
- PipeCD: Lets promote deployments from one environment to another.
- Keptn was designed to provide visibility into your environment. It aims to provide SLOs throughout the multistage deployment Keptn is meant to automatically set up environments (dev/stg/prd) with gating based on metrics for your CI/CD workflows. Also intgrate testing tools, do performace testing, chaos engineering, and more.
- Palumi Kubernetes Operator: Lets developers write declarations in their chosen language: Golang, TypeStript, Python, etc. Instead of translating their infrastructure ideas into YAML, developers can write declarations in the language they use for the rest of the project.

## Templating

If you are using mainly raw Kubernetes manifest files, your best bet is to use Kustomize as mucho as possible.

Although you can do a lot with Kustomize, it's not a templating enigne (it is a patching framework).

Helm is valuable when you want to parameterize your configuratinos instead of patching them. Helm is a templating engine and acts as a package manager.

Operators can help with snowflake situations when you need **more logic** put in place beyond just applying a manifest. Operators can help by putting declarative language around your imperative actions. You can store them in Git repositories.

Specific tools exist for writing operators (Operator SDK and KubeBuild), so you can effectively write the logic for how your application gets deployed.

## Git Workflows

> Git Workflows are at the center of the GitOps deployment, because workflows are the means of implementing the canges in the environment.

When you implement GitOps:

- Git is the source of truth
- Git is your interface into your environment.

But there are differences between how you manage your code in Git and how you manage your GitOps configuration in Git

- Separate configuration from code
- How to use branches
- How to use turnk-based development workflows

### Separate Repositories

> Separate your application code repository from your YAML configurations.
> This might seem counterintuitive initially!

Reasons for separate repositories:

1. You don't want a configuration change to triger a rebuild of your application if your application code did not change.
2. The approval process of getting a change into an environment should not hold back continuous integration of your code. **In general, application code and configuration information have independent lifecycles.**
3. You don't want bottleneck between teams

### Separate Directories, Not Branches

> Separate environments into different directories, **but not create branches for them**.
> This might seem counterintuitive also!

Every environment has configuration details specific to that environment. You can manually make the changes one by one or "cherry-pick", but then your "simple merge" IS NO LONGER THAT SIMPLE.

In the world of Kubernetes, Kustomize and Helm, using branches **using branches for environments is an antipattern**.

- Kustomize and Helm make using directories and overlays for your environments easier.
  - Kustomize allows you to have a core set of manifests (called "base") and store the deltas in directories (called overlays) You can use these overlays as directories with specific environment configurations in these directories.

### Trunk-Based Development

> The recommended workflow for GitOps with Kubernetes manifests is known as **trunk-based development**.

This method defines one branch as the "trunk" and carries out development on each environment in a different short-lived branch. When development is cmoplete for that environment, the developer creates a pull request for the branch to the trunk. Developers can also create a fork to work on a environment, and then create a branch to merge the fork into the trunk.

Once the approvals are done, the pull request (or the branch from the fork) gets merged into the trunk. The branch for that feature **is deleted** keeping uour branches to a minimum. **Trunk-based development trades branches for directories**

You can think of the trunk as a "main" or primary branch. production and prod are the popular names for the trunk branch.

This model works for GitOps because it keeps things simple and more in tune with how Kustomize and Helm work. When you record deltas between environments, you can clearly see what changes will be merged into the trunk.

> There is a single branch where the things can go wrong.

### Policies and Security

> Pay special attention
 to the features that version control offers for policy management and security to protect your trunk and provide stability to the environment.

Setting branch protection rules in GitHub (or equivalent from other Git providers) provides several benefits:

Branch protection also protects the branch from being accidentally or intentionally deletes.

> You need to trust what is in Git because it is in charge of managing your environment.

Also make sure that **all affected parties in your organization see a proposed merge**. For example, a network change should be approved not only by the system administration team buy also by the networking team and the security team.

> A rule can take the form of a "minimum number of approvals", but that does not limit the number of approvals to the minimum.

References:

- [Git Cherry Pic](https://git-scm.com/docs/git-cherry-pick)
- [Trunk Based Development](https://trunkbaseddevelopment.com/)
- [Protected Branches](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-
your-repository/defining-the-mergeability-of-pull-requests/about-protected-
branches#about-branch-protection-rules)

## Repo and Directory Structures

> Any organization that designs a system will produce a design whose structure is a copy of the organization's communication Structure - Melvin E. Conway

> There is no magic bullet, buy several common patterns ecist to match the various ways the organization interacts internally

Applying Conway's law to GitOps we can expect each team of developers to create its own branches and directories within a repository. Futhermore, the structure will be dictated by organizational boundaries (called "points of demarcation"), such security, operations, regulatory oncerns, etc.

> THIS EXAMPLES ARE A STARTING POINT, NOT A REFLECTION OF HOW YOUR FINAL REPO WILL BE REPRESENTED.

### Best Practices 

The following examples are designed to be generic to all GitOps implementations are not tied to a particular toolset or technology

#### DRY: "Don't Repeat Yourself"

We can adapt this acronym by rewording as "Don't Repeat YAML".

Storing everything in Git can sometimes lead to copying the same YAML over and over again in different places. Use the strategies described before to avoid duplication of YAML.

#### Parameterize Where You Need To

PArameterizing the configurations makes sense in some scenarios. This is where helm shines, specifically when you use the lookup feature.

### Repository Considerations

#### Monorepo

> Monorepo is a valid choice, but it can be quickly outgrown by the evolving requirements of the organization's operational needs. 

All manifests for the entire environment, including end-user applications, cluster config, cluster bootstraping, are stored in the repo. So `dev` and `prod` would live in the same repo.

- **Advantage:** provides a centra location for configuration changes. This enables visibility to the entire organization, making a smoother and clearer approval process.
- **Disvantages**
  - **scalability:** Difficult to manage the complexity of each deployment as it grows.
  - **performnace**: **ESPECIALLY IF YOU USE ARGO CD**. As the monorepo grows and changes become **more and more frquent**, the GitOps controller (Argo CD) takes more time to fetch the changes from the Git repository. This can slow down the reconciliation process and the deviations corrections from the desired state.

##### Monorepo use cases

- Startups 
- environment with a very limited domain

#### Polyrepo

Multiple repositories, possibly to support many clusters or deployment environments. 

Common examples are:

- **Separating concerns** between different departments of an organization.
  - a different repo for security, operations and application teams
- **Multitenancy** one repository per application

> Poliyrepo permits many possible designs

The **disadvantage** is that creates a large number of Git repositories to manage. This can become hard to manage, but **it scales incredibly well and is flexible enough to fit almost any organization**

##### Many-to-many Polyrepo

Each repository points to a **single cluster**.

- N number of clusters
- N number of repos

> Valid when each team take care of deploying its own infrastructure.

##### Polyrepo use cases

- Fits well and is flexible to fit almost any organization.

### Directory Structures

> A Git repo structure **will depend heavily** on how your organization is laid out.
>
> Your repository reflects **how your organization communicates with each other**
>
> Your repository represents your deployment workflow.

#### Repos reflecting organizational boundary

Single boundary between Kubernetes Platform Engineers and Kubernetes Application Developers.

##### Kubenetes Platform Administrator (Operations)

```txt
- bootstrap
  - base
  - overlays
      - default
- cluster-config
  - gitops-controller
  - identyty-provider
  - image-scanner
- components
  - applicationsets
  - applications
  - argocdproj
```

- `bootstrap`: bootstraping configurations.
  - `base`:  YAML installation configuration
  - `overlays`: GitOps controller configurations 
    - `default`
      - kustomization.yaml
        - components/applicatoinsets
        - componetns/argocdproj
- `cluster-config`: YAML for cluster's config manifests.
  - `gitops-controller`: files refers to `bootstrap/overlays/default`. It is deployed as ApplicatoinSet.
- `components`: configures GitOps controllers (Argo CD).
  - `applicationsets` contains YAML for the ApplicatoinSets
  - `argocdproj` contains YAML for the Argo AppProject
  - `applications` is a directory used as "point of entry" to onboard apps.

```sh
kubectl apply -k bootstrap/overlays/default
```

This command loads all the configurations and deploys the cluster-specific configurations onto the Kubernetes cluster.

##### Kubernetes Application Developer

```txt
- deploy
  - base
  - overlays
    - dev
    - prod 
    - stage
```

- The builk of configuration is in the `base` directory
- only deltas are stored in overlays
- Overlays can also be clusters

The YAML 