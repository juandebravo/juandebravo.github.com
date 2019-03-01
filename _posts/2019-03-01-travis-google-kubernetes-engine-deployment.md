---
layout: post
title: "Continuous Delivery to Google K8s Engine using Travis"
categories: [google, cloud, kubernetes, travis]
---

When you start a new project one of the key points to tackle and nail down
is how to automate the recurrent tasks you will be facing frequently, being
those tasks mainly around continuous integration and deployment.
Small investments in time around automation at the beginning (or at any time)
will pay off very quickly.

In a side project I've been working on over the last weeks, we chose
[**Travis CI**](https://travis-ci.org/) for Continuous Integration
and [**Google Kubernetes Engine**](https://cloud.google.com/kubernetes-engine/)
for building our production environment (very optimistic to call it *production*
at our stage).

[Travis official documentation](https://docs.travis-ci.com/user/tutorial/) describes
with detail how to set up your project for running tests, being them either unit or acceptance,
upon opening a Pull Request in GitHub or committing a change to a specific branch.

On the other hand, Google Cloud documentation includes a detailed example about how to
implement [continuous delivery to App Engine using
Travis CI](https://cloud.google.com/solutions/continuous-delivery-with-travis-ci).

However, I could not find any official documentation, blog post or even StackOverflow
thread describing how to configure Travis for deploying a change to Google
Kubernetes Engine when the integration phase ends with success. So this is what this post is about.

# The goal

The picture below describes what I was aiming for (remember that I'm putting aside
the Continuous Integration workflow):

![Continuous Delivery to Google Kubernetes Engine](/gfx/posts/travis-google-kubernetes/travis-gke.png)

I've created this [Github repo](https://github.com/juandebravo/travis-google-kubernetes-engine) as
an example. It may be useful for you in case you are considering to implement this workflow; if that's
the case, remember to update the configuration in the
[Makefile](https://github.com/juandebravo/travis-google-kubernetes-engine/blob/master/Makefile)
with your Google Cloud data. If you're already familiar with Travis and Google Cloud, you may skip
the rest of the post and go directly to that repository.

# Setting up Travis

Travis setup is configured in the [.travis.yml](https://travis-ci.com/getting_started) file, that should
be part of the repository that keeps your source code. In this chapter we'll go through the main
aspects to consider for our scenario.

Even though we want to deploy the new version whenever there's a new change in *master branch*, we should
do that *only if* the unit/integration testing phase ends successfully. Following [Travis job cycle
documentation](https://docs.travis-ci.com/user/job-lifecycle/), the configuration to achieve that is
the following:

- *branches -> only*: set this configuration to _master_:

```
branches:
  only:
  - master
```

- *after_success*: define the script that will tackle the deployment to GKE. We use [Travis automatic
env variables](https://docs.travis-ci.com/user/environment-variables/) to ensure we don't deploy a new
version in case of the event being associated to a new Pull Request.

```
# Deploy web version to Google Kubernetes Engine
after_success:
- if [ "$TRAVIS_PULL_REQUEST" == "false" ]; then ./deploy-web.sh; fi
```

Another relevant point is to enable *Docker as a service*, as we need to build a Docker image and push it
to Docker registry.

```
services:
- docker
```

[Here](https://github.com/juandebravo/travis-google-kubernetes-engine/blob/master/.travis.yml)
you can find the `.travis.yml` from the example.

# Setting up Google Cloud

This guide assumes you already have a Google Cloud project running a [Kubernetes
Engine](https://cloud.google.com/kubernetes-engine/) cluster.

## Storage

In order to be able to deploy a new workload to Kubernetes, it's required to push the relevant
Docker images to a [Docker registry](https://docs.docker.com/registry/), that will be used by
Kubernetes do pull them.

Even though we could use [Docker hub](https://hub.docker.com/) for that, we will use [Google Container
Registry](https://cloud.google.com/container-registry/docs/). This means we'll need to
grant to our script running in Travis permissions to get access to **Google Storage**, as this is
the infrastructure where Container Registry stands.

## Create Service account and key associated to it

In order to deploy a new version from Travis to Google Cloud, we need to create a service account that
enables the *Cloud SDK* (installed in our Travis environment, as we'll see) to authenticate with
our Google Cloud project.

- In the Google Cloud Project Console, open the _IAM & admin_ page.
- Go to _Service accounts_.
- Click _Create Service Account_.
- Enter a Service account name, such as _continuous-delivery-from-travis_.
- Include a description for the account.
- Click Create.
- In the _Service account permissions_ select box, select the following roles:
  - _Kubernetes Engine -> Kubernetes Engine Developer_.
  - _Storage -> Storage Admin_.
- Click Create.
- Click Create Key, choose _JSON_ as Key type and press Create. A JSON file containing the generated key is downloaded to your computer.
- Click Done.

# Define your strategy for making available the Service Account Key in Travis

Key management is a top priority task from security point of view for any project, and good news is
that any cloud provider gives you a solid solution for secrets management. I'm not tackling this
point in this article, but the as this article is related to Travis and Google Cloud,
those two links are the key entry points for the subject:

- [Travis encryption files](https://cloud.google.com/solutions/secrets-management/).
- [Google Cloud Secrets management](https://cloud.google.com/solutions/secrets-management/).

# Deployment script

Now that we've Travis and Google Cloud configured, the last point to address is the script that will
deploy the latest version of our software to our Kubernetes cluster. The script should:

- Use [gcloud command-line tool](https://cloud.google.com/sdk/gcloud/) to connect to Google Cloud.
- Ensure `kubectl` is available.
- Authenticate against Google Cloud (using the key we generated and encrypted in previous steps).
- Configure the project, Kubernetes cluster and Docker registry.
- Build Docker images.
- Push Docker images to Docker registry.
- Deploy to Kubernetes cluster.

[This bash script](https://github.com/juandebravo/travis-google-kubernetes-engine/blob/master/deploy.sh)
covers the steps above for the example repo.

# Conclusion

Using Travis for Continuous Delivery to a Google Cloud Kubernetes Engine is quite straight forward... as
long as you're familiar with both tools :rocket:.
