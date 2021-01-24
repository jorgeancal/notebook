+++
title= "How to set up your kluster with helmfile"
date= "2021-01-24"
comments = true
categories = ["Helm", "kubernetes", "How to"]
description = ""
tags= []
author = "Jorge Andreu Calatayud"
+++


Helmfile is a tool that allows you to get more out from Helm. When you use helmfile you can implement much helmcharts as you want. Helmfile allows you to template the helmcharts with the values that you want to and itwill ship it to your cluster. Helmfile bring modular deployments too, saying this I meaen that you can have a huge list of deployments and you can tell deploy only this group of helmchart, furthermore you can deploy them in sequence.

## How has helmfile been ported?
After 6 months using helmfile the truth is that I didn't know why we did not migrate everything earlier. The implementation of new features or new deployments in the cluster for our infra is easier and faster to start with. Helmfile bring me security that everything than I'm pushing is going to work in the pipelines because I can do test locally in an easy way.

## What do I like about helmfile? 
After using it for a while, this is what I liked the most about helmfile:
