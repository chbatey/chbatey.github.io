---
layout: page
permalink: /containers/
title: Containers 
---

Containers, specifically Linux containers popularised by Docker, offer a way to package an application and its dependencies
in a language agnostic way. The package, a container image, can be versioned and published to a registry.
Typically, a descriptor, e.g. Kubernetes' manifests, are also required to describe how to run the image as a container such as:

* How many instances to run
* The resources to give the container
* Configuration for [ingress](./systems.md#system-ingress)

Running containers requires orchestration ranging from:

* VM instances with a single container per VM e.g. GCP's [compute engine managed instance group](https://cloud.google.com/compute/docs/containers/deploying-containers)
* A third party PaaS e.g. GCP's [flexible compute engine](https://cloud.google.com/appengine/docs/flexible)
* A managed Kubernetes cluster e.g. [Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine)
* A fully managed container environment based on Kubernetes e.g. [Google Cloud Run](https://cloud.google.com/run)
* A PaaS built with a combination of the above and custom software


