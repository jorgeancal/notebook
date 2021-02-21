+++
title= "Como configurar SOPS en Helmfile"
date= "2021-02-30"
comments = true
categories = ["Helm", "kubernetes", "How to", "helmfile"]
description = "Te explicare como implementar sops en Helmfile de una manera rapida y sencilla."
author = "Jorge Andreu Calatayud"
tags= ["helm", "helmcharts", "sops"","kubernetes", "helmfile", "cluster", "Kluster", "helm-secrets", "secrets"]
+++

Como veíamos por encima en nuestro anterior post sobre helmfile podemos tener un fichero en nuestro repo que este encriptado mediante sops y tener ahi las variables para usar en nuestra chart mediante Helmfile.

## ¿Qué es Sops?
Si nos vamos a si [proyecto](https://github.com/mozilla/sops) veremos que Mozilla define SOPS como un editor de archivos cifrados que admite los formatos YAML, JSON, ENV, INI y BINARY. Además de soportar el cifrado de estos mediante los cifrados AWS KMS, GCP KMS, Azure Key Vault y PGP. 

## ¿Cómo implementarlo?

Como muchos sabréis, en esta vida todo se basa en plugins...y como Helm nos deja instalar diferentes plugins solo hemos de encontrar el plugin perfecto y en este caso hay uno llamado `helm-sercrets`. Si buscamos este plugin en [Helm Community](https://helm.sh/docs/community/related/#helm-plugins) vemos que ellos recomiendan  [jkroepke/helm-secrets](https://github.com/jkroepke/helm-secrets) que este es un fork de [zendesk/helm-secrets](https://github.com/zendesk/helm-secrets) pero este ultimo ha sido abandonado...

He de decir que cuando empece a mirar este plugin fue a mediados del 2020 que para aquel entonces no estaba obsoleto. Por suerte no ha cambiado mucho su utilización y sigue siendo rápida y sencilla.

Lo primero de todo hemos de instalar las dependecias del plugin y esta es instalar SOPS. Si mal no recuerdo si tienes Ubuntu o Debian se instala solo cuando instalar el plugin pero aqui un servidor usa ArchLinux y ha de instalarlo manualmente. Yo os recomiendo que lo instaleis manualmente que no cuesta nada solo habeis de ir a los [releases](https://github.com/mozilla/sops/releases) del repo e instalar.

Una vez que lo habéis instalado ejecutáis el siguiente comando para instalar el plugin siendo `${HELM_SECRERS_VERSION}` la última version stable del plugin.
```bash
helm plugin install https://github.com/jkroepke/helm-secrets --version ${HELM_SECRERS_VERSION}
```

Una vez que tenéis esto solo habéis de ir a vuestra carpeta de helmfile y crear el siguiente fichero llamado `.sops.yaml`, por puesto teneis que elegir una de las tres opciones he de decir que está permitido tener multiples KSM y PGP keys.

```yaml
creation_rules:
  # Encrypt with AWS KMS
  - kms: 'arn:aws:kms:eu-west-1:222222222222:key/111b1c11-1c11-1fd1-aa11-a1c1a1sa1dsl1'

  # Encrypt using GCP KMS
  - gcp_kms: projects/mygcproject/locations/global/keyRings/mykeyring/cryptoKeys/thekey

  # As failover encrypt with PGP
  - pgp: '000111122223333444AAAADDDDFFFFGGGG000999'

  # For more help look at https://github.com/mozilla/sops
```

Ahora ya solo os queda agregar el campo `secrets:` en vuestro fichero del release y listo. Os quedaría una cosa asi:

```yaml
- <<: *defaultTmpl
  name:  "grafana"
  chart: "grafana/grafana"
  namespace: "monitoring"
  version: "3.2.5"
  installed: {{ .Values | getOrNil "grafana.installed" | default false }}
  needs: 
    - observability/fluentd
    - observability/prometheus
    - operators/jaeger-operator
  secrets:
    - releases/grafana/secrets.yaml
```

## ¿Como usar SOPS como editor de texto?
Hay muchas formas diferentes de editar los ficheros pero yo te recomendaría que te crearas una variable de entorno y luego vayas a la carpeta donde tienes el dichero y lo abras con el siguiente comando `sops secrets.yaml` y sops se encarga de desencriptarlo y encriptarlo por ti.

Yo por ejemplo uso SOPS con KMS ARN por lo que tengo una variable de entorno llamada `SOPS_KMS_ARN` ejemplo:

```
export SOPS_KMS_ARN=arn:aws:kms:eu-west-1:222222222222:key/111b1c11-1c11-1fd1-aa11-a1c1a1sa1dsl1
``` 


Esto ha sido todo! Nos leemos!