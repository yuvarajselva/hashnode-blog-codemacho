---
title: "How to handle Multi-Cluster setup for AKS in your local machine"
subtitle: "Handling multi cluster setup can be time-consuming to setup and this blog would make your job easier"
date: "2023-05-04"
hero_image: "./hero_1.jpg"
hero_image_alt: "A man juggling with balls"
hero_image_credit_text: "Matt Bero"
hero_image_credit_link: "https://unsplash.com/@mbeero?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText"

tags: Kubernetes, Bash, Scripting, jq
cover: https://images.unsplash.com/photo-1583169462080-824fc2ebed23?ixlib=rb-4.0.3&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=2340&q=80
domain: yuvarajselva.hashnode.dev 
publishAs: Yuvi
---

# Introduction

> Are you tired of juggling of between multiple clusters in your local
> machine?{" "} 
>
> Then, it is time to automate your local setup and it's configuration. If you're
> working as a 
> "cluster-admin"
>  for anyone of the enterprises, then this blog might be able to help you.

## Prerequisites

1. Bash Scripting
2. [JQ](https://gist.github.com/olih/f7437fb6962fb3ee9fe95bda8d2c8fa4)

## Pros

1. Multiple clusters can configured locally in an automated way
2. If your local configuration is messed up (Which I do often :P), it is easy to fix it.

## Manual Way?

To setup a cluster locally, the cluster context needs to be added in your machine.

```sh
az aks get-credentials -n "${name}" -g "${resource_group}" # Setting the Kube Context
```

> Running the above cmd for every cluster can be cumbersome. Let's automate it!!

## CODE

Let's jump into the solution right away.

1. First, try to create a json file that acts as the input for our script.

```json
{
  "<Subscription-ID>": {
    "region": "<region>",
    "name": "<cluster  name>",
    "resource_group": "<Resource Group Name>"
  },
  "<Subscription-ID>": {
    "region": "<region>",
    "name": "<cluster  name>",
    "resource_group": "<Resource Group Name>"
  },
  "<Subscription-ID>": {
    "region": "<region>",
    "name": "<cluster  name>",
    "resource_group": "<Resource Group Name>"
  },
  "<Subscription-ID>": {
    "region": "<region>",
    "name": "<cluster  name>",
    "resource_group": "<Resource Group Name>"
  },
  "<Subscription-ID>": {
    "region": "<region>",
    "name": "<cluster  name>",
    "resource_group": "<Resource Group Name>"
  },
  "<Subscription-ID>": {
    "region": "<region>",
    "name": "<cluster  name>",
    "resource_group": "<Resource Group Name>"
  },
  "<Subscription-ID>": {
    "region": "<region>",
    "name": "<cluster  name>",
    "resource_group": "<Resource Group Name>"
  }
}
```

2. Read the data from the input JSON file created at the previous step using the JQ cmd and fetch the list of subscription IDs.

```sh
# Getting the list of subscriptoin IDs
subscription_ids=$(echo "${data}" | jq -r 'keys | .[]')
```

3. Let's loop the list of subscription IDs and update the kube context for every cluster

```sh
echo "${subscription_ids}" | while read -r subscription_id; do
    echo "Setting the AZ Context for the Current Sub ID: ${subscription_id}"
    az account set -s "${subscription_id}" # Setting the Az Context

    name=$(echo ${data} | jq -re '.'\"$subscription_id\"'.'name'') # Fetching the Name of the Cluster
    resource_group=$(echo ${data} | jq -re '.'\"$subscription_id\"'.'resource_group'') # Fetching the name of the resource group
    echo "Getting the aks credentials for the cluster : ${name} in the rg: ${resource_group}"
    az aks get-credentials -n "${name}" -g "${resource_group}" # Setting the Kube Context
done
```

The whole code looks like something below.

```sh
#!/bin/bash

# Getting the Cluster Info from the local Json
data=$(cat <"./data/cluster_info.json" | jq -r)

# Getting the list of subscriptoin IDs
subscription_ids=$(echo "${data}" | jq -r 'keys | .[]')
echo "${subscription_ids}" | while read -r subscription_id; do
    echo "Setting the AZ Context for the Current Sub ID: ${subscription_id}"
    az account set -s "${subscription_id}" # Setting the Az Context

    name=$(echo ${data} | jq -re '.'\"$subscription_id\"'.'name'') # Fetching the Name of the Cluster
    resource_group=$(echo ${data} | jq -re '.'\"$subscription_id\"'.'resource_group'') # Fetching the name of the resource group
    echo "Getting the aks credentials for the cluster : ${name} in the rg: ${resource_group}"
    az aks get-credentials -n "${name}" -g "${resource_group}" # Setting the Kube Context
done

```

> NOTE: The above solution is applicable only for AKS Clusters.
