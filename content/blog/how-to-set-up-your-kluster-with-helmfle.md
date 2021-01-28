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

## What do I like about helmfile?
After 6 months using helmfile, this is what I liked the most about helmfile:
-  it's stupidly faster because you have all the releases in the same file and you ship them to the cluster in the order that you want.


- Centralized Values. I love the possibility to have a main config per environment and then be able to apply those different values to the charts.

- `diff`. This command in helmfile let me compare with the actual chart that I have in the cluster. I know that when you exec the diff you have thousands of lines but sometimes is useful because it shows if you did it okay or just blow up it.

- Env Variables. it is a lovely alternative to get everything working without having password in text plain in a file. An alternative to this is `helm-secrets` but I didn't use that much, so I cannot tell how good is it. I have to say that it is an awesome idea that you can use `sops` to encode the variables value. I'm stating to look into it but I didn't implement yet.
  
## How to set up Helmfile?

Firstly, we need to set up a folder schema. You can have it in a folder with another stuff, but I prefer to have it in the root of the repo. it's going to be better if I show you the schema (I'll create a repo with all this soon in github)

```shell
.
├── addReleases.sh
├── base
│   ├── defaults
│   │   └── helmfile.yaml
|   ├── environments
│   │   └── helmfile.yaml
│   ├── repositories
│   │   └── helmfile.yaml
│   ├── templates
│   │   └── template.yaml
│   └── values
│       ├── minikube
│       │   └── values.yaml.gotmpl
│       └── production
│           └── values.yaml.gotmpl
├── helmfile.yaml
├── README.md
└── releases
    ├── prometheus
    │   ├── helmfile.yaml
    │   ├── README.md
    │   └── values.yaml.gotmpl
    └── grafana
        ├── helmfile.yaml
        ├── README.md
        └── values.yaml.gotmpl

```

Now I'm going to show you what I have in the main helmfile file.

```yaml
{{ readFile "base/templates/helmfile.yaml" }}

{{ readFile "base/repositories/helmfile.yaml" }}

releases:

```

As you can see above we have two readFile commands, and a release key with no releases. Let's go to follow line by line the file. The first line is going to the template that I've created with some patters that it's going to be equals for all the releases, and I don't want to write the same line 10 time... After that we have the readfile for the repositories. Yep we have all the repositories from all the file in there... yeah it sounds crazy, but it gives you a bit more of speed. 


Now let's see the template file

```yaml
bases:
  - base/defaults/helmfile.yaml
  - base/environments/helmfile.yaml
  
templates:
  defaultTmpl: &defaultTmpl
    missingFileHandler: Warn
    valuesTemplate:
      - base/values/{{ .Environment.Name }}/values.yaml.gotmpl
      - releases/{{ .Release.Name }}/values.yaml.gotmpl
```
As you can see, we have a bit more than a simple template here. We have the bases in here too. I would probably move them to the main hemlfile or change the name of the file in some point. I like thee bases in the file at the moment becase it kind of feel like the template is part of the base of helmfile.






