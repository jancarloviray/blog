---
layout: post
title: PAAS comparison (2017): Dokku, Flynn, Deis, Kubernetes
permalink: blog/paas-comparison-2017-dokku-flynn-deis-kubernetes/
comments: True
excerpt_separator: <!--more-->
---

2017 is when container orchestration and technology will mature and clear winners for different use cases will dominate. Here are my personal notes on common paas technology, comparing Dokku vs Flynn vs Deis vs Kubernetes. **tldr**: Easy but no HA: Dokku. Easy with HA: Flynn. The big dawg: Kubernetes.

<!--more-->

## Dokku

The mini PaaS. It's more like the introduction to PaaS world. This can be run anywhere, even on a small single server. For simple deployments. Does not scale to multiple clusters. Many are happy with it for their simple projects. Note that this is **single host** only, so "high availability" is non-existent. Want to have a PaaS in minutes? Choose Dokku.

**WHY CHOOSE DOKKU**: for hobby and side projects that do not require high availability. Easy to deploy applications, with little to no devops needed.

## Flynn

The upgraded Dokku and is used by around a dozen companies. It can scale and has ability to be highly available, so there's no single point of failure. It is noted as more production-ready than Dokku. Runs high availability databases within the platform in addition to stateless apps. Their goal is to have an easy to use unified solution, but at the cost of having a lot of components. Big plus: it has its own web dashboard! You can run in single server or scale out. It's known for its flexibility and ease. This does not use CoreOS so you can run it on Ubuntu.

**WHY CHOOSE FLYNN**: liked Dokku but need high availability and a web UI to manage your clusters? Choose this.

## Deis

Similar to Flynn but has more companies using it in production it seems? It is built around the Kubernetes and Docker ecosystems. Doing more research on this...

## Kubernetes

The big giant and the future of container orchestration. This is obviously the choice for more serious deployments. It is packed with 10 years of Google experience, plus it powered Pokemon Go deployment. This is undoubtly what you should choose if you are serious about scaling with container technology. Big caveit: it is very hard to setup that you will most likely want to host your application in a one-click solution that Google Cloud built.

**WHY CHOOSE KUBERNETES**: you are serious pretty serious about deployment and you have serious scaling needs. You need something mature and battle-tested

## Conclusion

```
Dokku           Hobby Projects / Prototypes
Flynn           Small Production Deployments
Deis            ???
Kubernetes      Huge Deployments
```
