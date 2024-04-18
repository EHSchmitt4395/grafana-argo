# Grafana-Argo-Ex

## Overview
This repository facilitates a fully autonomous deployment of the Grafana agent, complete with metrics scrapers, into a virtual Kubernetes (k8s) environment. This setup is ideal for localized environments like Minikube and k3s. It leverages ArgoCD for orchestrating autonomous Helm deployments and includes tools for encrypting sensitive values.

Please see argo-per-cluster for the full example.  This is a more classical and transparent means of orchestrating your kubernetes as state with argocd and a more legacy app of of apps type model. But a great first step before jumping into more abstract orchestration as code models.

------------

For production today i'd recommend an applicationset matrix generator. These are a bit more complex to wrap ones head around but are much more powerful with much less code, and explicit directive with kustomizations and patches. 

These example foundation generators can be foud in the master-argo-many-env-zones directory. Like the name eludes, these applicationset objects enable a 'master' argocd to deploy applicatons across a fleet of clusters. Vs the other directories more one to one model (one argo per env). Mind you the same can be achieved in the first direc, as this one but this is a much more efficient method. 

Utilizing these matrix generators in combination with multi source applications, one can point to a plethora of sources, and a plethora of varying destinations with slight differences and enforce the git repos state as code onto the cluster.  This in todays devops climate is much more efficient than a standard CD push model as argocd will monitor each clusters state and enforce that current state matches its environment specific code. 

-------------

Be sure to check the sourced repo that provides instruction on matrix generators with GCP fleet. a great foundation and lab for deploying and managing across multiple clusters in google cloud. 

In the perspective of grafana, the multi cluster model is great for instantly instantiating environments as code for out of the box functionality in deployments.  A management cluster can be procured with all meta-monitoring tools, argo-rollouts utility for blue-green or canary based helm upgrades backed by k6 tests for shift right based validations pre production in dev-staging-prod environments. 