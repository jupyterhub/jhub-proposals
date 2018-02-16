# BinderHub SciPy 2018 Proposal

## Info + Links
**Submission Date: Feb 15th**
* [SciPy submission guidelines](https://scipy2018.scipy.org/ehome/299527/648140/)

**NOTE: There are a lot of notes/outlines/conversations at the end of this document. Anything in "short description" or "long description" should be treated as the latest version of info. Those notes are for inspiration / organizing thoughts**

## Title
- Binder 2.0: the next generation of reproducible scientific environments with repo2docker and BinderHub 

## Short Description
*About 100 words*

[Binder](http://mybinder.org/) allows researchers to quickly create and share computational environments needed to interact with research code and data. 

In this talk, we discuss the technical decisions and changes made in Binder 2.0. 

Many changes focus on deployment — modern-day cloud management tools (Kubernetes and Helm) and JupyterHub make the service more reliable, scalable, and easier to deploy.

We focus on improvements for researchers using Binder to share their work. Supporting new workflows, languages, and REST APIs in Binder 2.0 required developing two new components: repo2docker and BinderHub. We will describe how these components work and how researchers can best use them. We will conclude with a vision for the future of Binder.

## Long Description
*About 500 words*

Computational reproducibility has long been a goal of computational researchers, but has remained a challenge. Even when it is achieved locally, sharing this work often comes at the cost of reproducibility. Fortunately, modern technologies are making this ideal far more achievable by even the casual researcher.

Binder 1.0, [introduced at SciPy 2016](https://www.youtube.com/watch?v=OK6M4w7LYIc), was a prototype for building, deploying, and sharing computational environments. It provided a public web service at mybinder.org that generated Docker images from GitHub repositories, and served Jupyter Notebooks connected to those Docker images.

In 2017, we [introduced Binder 2.0](https://blog.jupyter.org/binder-2-0-a-tech-guide-2017-fd40515a3a84), a dramatic change that required completely re-writing the underpinning code. This meant both adopting pre-existing modern-day tools and developing new components. 

Binder 2.0 uses JupyterHub to manage user sessions and authentication. It uses Kubernetes to manage cloud resources, and Helm for deployment and upgrading. As a result, Binder is more scalable, reliable and easier to deploy.

Improving the user experience required developing repo2docker and BinderHub. These two libraries will be the main focus of this talk.

[**`repo2docker`**](https://github.com/jupyter/repo2docker) is a tool to build Docker images from source code repositories. While it is optimized for being managed by a Binder process, it can be used to generate Docker images for JupyterHub deployments.

While Binder 1.0 could generate environments from a small number of files, `repo2docker` is more flexible. With Binder 2.0 authors can define environments in a greater variety of ways. This flexibility is enabled by "buildpacks" which tie specific files to different kinds of functionality:

 - `requirements.txt` and `environment.yml` define Python installations with corresponding dependencies.
 - `install.R` installs R and RStudio
 - `REQUIRE` installs a Julia kernel
 - `apt.txt` uses `apt-get` to install dependencies
 - `postBuild` runs an arbitrary shell script after all dependencies are installed

Buildpacks are composable, and can be used in combination to specify computational environments (multiple languages, interfaces, etc). After building an image, `repo2docker` can push the image to a Docker registry.

[BinderHub](https://github.com/jupyterhub/binderhub) provides REST APIs (and a user interface) to control and connect `repo2docker` and `JupyterHub`. Users can share links that specify which git repository is the basis for the associated image. BinderHub caches images, and uses repo2docker to build requested images absent from the cache. Then, BinderHub uses JupyterHub to create a live environment for the user, serves the repository's image, and directs the user to a UI (such as a Jupyter Notebook, JupyterLab, or RStudio).

Finally, we will discuss plans for the next phase of Binder's development, which includes a federated network of BinderHub servers, as well as development towards incorporating Binder into scientific workflows such as academic publishing and reproducibility.

Prior speaking experience:

M Pacer:
Pacer, M, Avila, D., & Hamrick, J. B. (2017). The Jupyter notebook as document: from structure to application. [Talk presented at JupyterCon](https://www.youtube.com/watch?v=V3cENs1UYQU)

Chris Holdgraf:
Holdgraf, C. (2017). Visualizing the brain with open-source tools. [Talk presented at Plotcon](https://www.youtube.com/watch?v=V847VcZaC_Q)

Min Ragan Kelley:
Ragan Kelley, M., & Kluyver, T. (2016). JupyterHub: Deploying Jupyter Notebooks for students and researchers. [Talk presented at PyData](https://www.youtube.com/watch?v=gSVvxOchT8Y)
