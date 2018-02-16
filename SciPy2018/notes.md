Original source: 
https://hackmd.io/MYDg7AzAnALAjBAtAVgKYCMqJhAJgNkRH2QAZFVgpcoRlh9c4Yg=

# Other stuff

## Figures & References for use later

![](https://i.imgur.com/uFNRK6U.png)
Figure 2: Examining image data from Ross et al. on Binder with JupyterLab.  JupyterLab is one of the three user interfaces available to Binder users. In this example, we use JupyterLab to view additional image data in a code console.  Note that this modification to the code can be made without requiring the user to install dependencies.

![](https://i.imgur.com/7zc98nQ.png)
Figure 1: An example of binder from the LIGO Open Science Center (LOSC) which teaches users to work with gravitation wave data.  In the address bar above, we see that this Jupyter notebook is not running locally, but on binder.  To access this binder, we are able to click on a link provided by LOSC and have immediate access to code and data.

References

1\.   [Ross AS, Hughes MC, Doshi-Velez F. Right for the Right Reasons: Training Differentiable Models by Constraining their Explanations. Proceedings of the Twenty-Sixth International Joint Conference on Artificial Intelligence. 2017. p. Pages 2662–2670.](http://paperpile.com/b/FMgQkX/7HDA9)


## General outline / ideas

### Higher level 
* General structure
    - this is Binder (the service), does these things
    - uses components:
        - repo2docker: build Docker images from git repos
        - BinderHub: 
    - Primary usecases
        - reproducible documents
        - education and teaching
        - easy interactivity for open-source documentation
        - easy interactivity for sharing code snippets and analyses/results within collaborators 
        - backend services that can be used for other projects (e.g. back-end building w/o the Binder UI)
        > [name=M Pacer] Is that a currently supported use case? I thought the aim was to discourage non-Binder uses of repo2docker & the like
        > [name=CH] Yes, e.g., `play.nteract.io` and the thebelab
        > [name=MP] Ah, I thought that meant building images with repo2docker without using BinderHub or JupyterHub, that makes sense

### High-level Project Goals
- User goals
    - Empower uses to create their own interactive repositories
    - Don't branch off from pre-existing workflows. Minimal new things one must learn.
    - Support as many use-cases as possible. Not just Python or Jupyter.
- Tech goals
    - Be cloud, language, user-interface agnostic
    - Utilize building blocks that are modular and useful in their own right
    - Use open-source software through the full stack
    - make it possible for others to deploy a Binder server
    - Support both outward and upward scalability

### User-perspective view of Binder
1. User-perspective view of what binder is and why (allow authors to share executable environments; reproducibility & scientific papers)
2. What is the Binder service?
    1. Binderhub: web application
        1. given a git repo url & ref
        2. resolves git ref to a git hash
        2. check if the image for the hash exists
        3. if it doesn't exist, [build image with repo2docker]
        4. binderhub, asks jupyterhub to create a user
        5. binderhub asks jupyterhub to launch a container for this user with the image from steps (3||4)
        6. JupyterHub asks kubernetes to create the container
        7. BinderHub replies to the original request, with url for now running container on Eventstream
        8. eventstream relays back logs to the eventstream
        > [name=M Pacer] I'm not sure what this is actually referring to… how does the eventstream relay logs to itself? (note: this may be do my incomplete note taking)
        > [name=CH] I'm not sure what it means either :-) it was in the original version of these notes so I didn't wanna take it out
    2. If image is not built, launch a new container running repo2docker
        * clones repo
        * looks in repo to identify how to construct the Dockerfile 
            * exactly one Buildpack constructs the Dockerfile
            * buildpacks are configurable
        * builds image (asks long-running docker to build the image from the Dockerfile)
        * pushes image to registry
    3. JupyterHub launches a user pod and connects with the image.


### Key technical pieces

1. `repo2docker`
    * _(where all the magic happens)_
    * given a hash and repository, construct an environment
        * happens to be using Docker and is for running notebooks
    * receive ref to repo -> inspect repo for config files -> build dockerfile from config files -> build image from dockerfile
    * binderhub : repo2docker ~::~ nbviewer : nbconvert
        * (AKA, binderhub is a service that uses repo2docker, while nbviewer is a services that uses nbconvert)
    * Intentional decision not to incorporate data etc. 
        * Need better tools to connect data with Binder sessions.
    * Designed around "build packs" for composability (e.g. Python 2 + R)
    * List current build packs / supported config files.
2. `binderhub`
    * Glue that binds the repo2docker and jupyterhub (and some other things) together
    * Concept of "repo provider", which is a set of instructions to go from ref -> git repository
    * Fetches a git repository, uses repo2docker to build/register a docker image, uses JupyterHub to direct users to an environment with that image
    * Provides a user interface around the above procedure.
    * resolves refs to a hash, so can support tagged repositories / branch names / commit names
    * composed of Python code for the building machinery, javascript for a user-interface, and a helm chart for the JupyterHub deployment needed.
    * All pieces are run in Kubernetes via containers
3. `jupyterhub`
    * Serves user sessions from a Docker image registered in the cloud
    * Runs on Kubernetes which the BinderHub service also uses.
    * Don't go into a ton of detail on JupyterHub since this is just a specific use-case of JupyterHub, not a paper about JupyterHub.


### Key decisions:
- Why a new architecture for Binder?
    - maintenance difficulties for busy group of devs with a complicated service even on k8s
    - new/more mature tools can automate more of the maintenance steps (helm, jupyterhub)
- Why Kubernetes + helm?
    - Kubernetes is the only thing that creates containers, that way it knows what exists to load balance (&c.)
    - helm makes deployment, configuration and upgrades really nice
    - kubernetes + helm give us security controls over the process (both the executable environment, but also the build process)
- Why `jupyterhub`?
    - maintenance question: jupyterhub already existed for managing notebook servers
- Why `repo2docker` is a separate component?
    - testing purposes
    - portability allows you to run images without using Binder

### What's next?
- data (what's done today)
- federation


## Misc


> [name=MB]
> I would add a quick metions the we do CI/CD and that merging a PR deploys mybinder.org and is relatively super-simple for people not familiar with kubernetes/helm.

### good content needed to be cut:
`repo2docker`'s buildpack model also
makes it easier to extend the core functionality of the package in
order to fit new workflows going forward. The project has quickly
added new functionality for interfaces such as RStudio or Jupyter Lab,
and has focused on adopting pre-existing conventions in a field rather
than inventing new motifs for people to learn.


### Content similar to other talks

> The goal is to provide a single link that users can follow to interact with public Jupyter notebooks or RStudio or other environments in a live, temporary environment. Binder has been rebuilt to leverage existing technologies such as JupyterHub, Kubernetes, and Helm to make the best use of community expertise and enable a small team to serve a wide and growing community. Learn about how binder works, how to make your own ideas interactive and reproducible, and how to deploy your own Binder service! 

> [name=M Pacer] Removed this chunk from the proposal as it doesn't focus on the libraries that are new content from scipy's perspective. 
> Also, I don't think we will be able to cover all of that in a SciPy talk length…

### Previous proposal (Binder @ SciPy 2016)

> [name=M Pacer] Here is the previous Binder short description that was given at SciPy in 2016:

> Binder is a service that bundles GitHub repositories with code, Jupyter notebooks, and data into reproducible, executable environments that can be launched instantaneously in the browser with the click of a button. Under the hood, Binder uses simple and flexible dependency specifications to build Docker images on demand, and then launches and schedules them across a public Kubernetes cluster. In this talk, I’ll describe in detail how Binder works, and highlight some exciting use cases. I’ll then describe several future directions for the project, including handling larger datasets, lowering barriers for environment specification, and supporting custom deployments with user-provided computing resources.

## criteria
-   In your abstract, be sure to include answers to some basic questions:
-   Who is the intended audience for your talk?
-   What, specifically, will attendees learn from your talk?
-   Ensure that your talk will be relevant to a broad range of people. If your talk is on a particular Python package or piece of software, it should useful to more than a niche group.
-   Include links to source code, articles, blog posts, or other writing that adds context to the presentation.
-   If you've given a talk, tutorial, or other presentation before, include that information as well as a link to slides or a video if they're available.



##### archived convo for cribbing

> [name=CH] That's about 116 words but not too bad (it's almost entirely just re-worded elife post, I am ++ on re-using old language to save us time)
> [name=M Pacer] My concern is that it doesn't really highlight that we aren't planning on covering the full workings of Binder itself. Because that can't easily be done in 15–20 mins, I don't think we should set ourselves up to not be able to deliver
> [name=CH] IMO we've only promised to introduce binderhub+r2d, and to cover the technical decisions that went into their design. I feel like we can cover that at a high level in 20 min, do you think no?
> [name=MP] what worries me is:
> > We will discuss the technical decisions that went into the next phase of Binder, and describe a vision for where we think it will go next.

> [name=MP] the concern is that suggests a broader scope.
> I share your ++ on reusing language though.

> [name=CH] IMO "technical decisions that went into the next phase of Binder" just means "we wanted to use kubernetes, we wanted more composability, and we wanted to support more than just jupyter notebooks" and we could leave it at that. "vision for where we think it will go next" just means 1 or 2 slides about the idea of federation just to get people excited + mentioning other kinds of workflows we want it to support.

> [name=MRK] "Buildpacks are composable" statement is not true, at least not yet. Adding another dependency file not supported by the chosen buildpack doesn't change what gets installed. Updated to be more precise. Previous Binder also already supported requirements.txt and environment.yml and custom Dockerfiles assuming they inherited from a specific image.


> [name=CH] If that's the case we should remove the language about it from the [repo2docker documentation](http://repo2docker.readthedocs.io/en/latest/design.html#composability) :-) >