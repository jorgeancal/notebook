+++
title= "Configurar tu kluster con Helmfile"
date= "2020-01-30"
comments = true
categories = ["Helm", "kubernetes", "How to", "helmfile"]
description = "Echamos un vistazo atrás de cómo se ha portado helmfile después de 6 meses usando helmfile en el trabajo y mostramos a cómo configurarlo."
author = "Jorge Andreu Calatayud"
tags= ["helm", "helmcharts", "kubernetes", "helmfile", "sops", "helm secrets" ]
+++

Helmfile es un programa que te permite sacar más partido a Helm. Ya que si usas helmfile puedes implementar cualquier cantidad de helmcharts. Básicamente con helmfile declaras las helmcharts y les das los valores que tú quieres a cada una de las helm chart, helmfile creara el correspondiente deploy mediante helm para mandar a tu cluster todo lo que has definido en helmfile, claro esta puedes decirle a helmfile impleméntame solo este grupo de helmchart y de este grupo quiero implementarlas en esta secuencia. 

## ¿Cómo se ha portado helmfile? 
Despues de estos 6 meses usando helmfile la verdad es que me ha dado mucha más vida. Si queremos implementar cosas nuevas cosas.... todo vas rapido y seguro. 

## ¿Qué me ha gustado de helmfile?
Después estar usando un tiempo esto es lo que más me ha gustado de helmfile:

- Lo que más vas a notar usando helmfile es que el tiempo de implementación de las helmchart habra descendido drasticamente. Por ejemplo en donde trabajo hemos notado una mejora de 15 minutes comparado de lo que usabamos anteriormente que era `Landscaper`, actualmente obsoleto. Dejo aquí un enlace al [repositorio](https://github.com/Eneco/landscaper) por si alguien está interesado.  


- Otra cosa que he notado es que gracias a helmfile tengo la posibilidad de tener toda la configuration de diferentes entornos en un mismo sitio y estoy trabajando en local y el solito ya sabe que valores tine que aplicar cuando está trabajando en local, ya que lo he declarado previamente en un fichero, entraremos en esto en detalle un poco más adelante. 


- `diff`. Posiblemente esto es lo mejor de todo... es estúpidamente sencillo y te salva mas de una vez cuando estas haciendo actualizaciones de charts o estas cambiando valores en la chart y estas seguro si te has cargado algo.


- `sops`. Un salvavidas en cuanto a tener un extra de seguridad. Algunos ya sabréis es pero para los que no... esto te ayudara a tener las contraseñas encriptadas en un archivo, normalmente tendrás que tener una key en tu local con la cual helmfile desencriptara el fichero y lo leera, de esta manera los valores poderas ser usados en donde ayas llamado a ese secret para esa chart.


- Variables de entorno. Básicamente si no quieres implementar `kops` o tener la variable escrita en un fichero puedes tenerla en las variables de torno de tu sistema. Un simple ejemplo necesitamos ponerle un valor a la cuenta del admin en grafana `Export ADMIN_GRAFANA_PASSWORD=salsademalacatoneswthP0llo!` y en el fichero tenderemos algo parecido a esto `password: {{ env ADMIN_GRAFANA_PASSWORD | quote }}` pero si quieres tener este valor que sea obligatorio usaríamos `{{ requiredEnv ADMIN_GRAFANA_PASSWORD | quote }}`.

## ¿Cómo lo configuro?

La configuración es bastante sencilla. Lo primero de todo es tener un repo para todo esto o en una carpeta si vas a tener lo en un mismo repo con mas cosas en el. Yo actualmente lo tengo en un repo exclusivamente para helmfile. Mi esquema del repo es el siguiente:


```txt
.
|- bases
|  |- environments.yaml
|  |- helmDefaults.yaml
|- releases
|  |- external-dns
|  |   |- helmfile.gotmpl
|  |   |- values.gotmpl
|  |   |- production.gotmpl
|  |   |- secrets.yaml
|  |   |- minicuke.gotmpl
|  |- grafana
|  |   |- helmfile.gotmpl
|  |   |- values.gotmpl
|  |   |- production.gotmpl
|  |   |- secrets.yaml
|  |   |- minikube.gotmpl
|  |- prometheus
|  |   |- ...
|  |- thanos
|  |   |- ...
|  |- ...
|- README.md
|- helmfile.yaml
```

Ahora os voy a mostrar lo que tenemos en el principal helmfile.yaml:

```yaml
bases:
  - "bases/helmDefaults.yaml"
  - "bases/environments.yaml"

helmfiles:
  - "releases/external-dns/helmfile.yaml"
  - "releases/grafana/helmfile.yaml"
  - "releases/prometheus/helmfile.yaml"
  - "releases/thanos/helmfile.yaml"
  - ...

missingFileHandler: Warn
```

Como podéis ver anteriormente tenemos tres bloques. En el primer bloque tenemos lo que viene siendo los pilares de todo. Tenemos los valores globales para helmfile. A continuacion os mostrare mis valores pero si queréis saber más sobre los valores que podeis cambiar dejo [aquí](https://github.com/roboll/helmfile#configuration) el link.

```yaml
helmDefaults:
  wait: true
  timeout: 700
  cleanupOnFail: false
```

Lo único que tengo que comentar aquí es que `cleanupOnFail` por defecto es false pero estamos investigando un poco sobre el caso asi que la tenemos ahi solo para recuerdo.

Ahora vamos a irnos a mirar nuestro `environments.yaml`. Como podéis ver, declaramos los dos entornos que tenemos y luego anadimos los valores, en este caso estamos indicando cuales charts queremos implementar en el entorno.

```yaml
---
environments:
  Production:
    values:
      - alias: production
      - external_dns:
          installed: "true"
      - prometheus:
          installed: "true"
      - ...
  minikube:
    values:
      - alias: minikube
      - external_dns:
          installed: "false"
      - prometheus:
          installed: "true"
      - ...
```

A continuación vais a ver el `helmfile.yaml` que tengo para grafana. Como veis es bastante parecido al otro `helmfile.yaml` ¿Por qué es esto? os preguntaréis pues es muy sencillo si yo estoy trabajando en una sola chart o tengo que implementarla manualmente por alguna razon extraña del destino solo tengo que ir a la carpeta del release y aplicar el fichero `.yaml` con helmfile.  

```yaml
---
bases:
  - "../../bases/helmDefaults.yaml"
  - "../../bases/environments.yaml"

---

repositories:
  - name: grafana
    url: https://grafana.github.io/helm-charts

releases:
  - name:  "grafana"
    chart: "grafana/grafana"
    namespace: "monitoring"
    version: "6.2.0"
    installed: {{ .Values | getOrNil "grafana.installed" | default false }}
    values:
      - values.yaml.gotmpl
      - {{ .Environment.Name }}.gotmpl
    secrets:
      - secrets.yaml
```



## SOPS in helmfile

Hay muchas maneras de usar sops, yo en este caso como estoy usando AWS voy a usar `SOPS_KMS_ARN`. Si queréis que cuente más sobre esto dejarme algún comentario y lo explicaré más en detalle, ya que yo no guardo un `.sops.yaml` en mi repo. Pero vamos a centrarnos en lo que helmfile nos da por defecto con `.sops.yaml` como podéis ver en su [documentación](https://github.com/futuresimple/helm-secrets) necesitáis instalar el plugin `secrets` en helm. Pero antes de eso neceistar saber si tenéis instalado el paquete `sops` en vuestro sistema operativo para saber si o sino ejecutar `sops --version`. Si os sale algo como lo siguiente `sops 3.6.1 (latest)` es que lo tenéis sino... instalalo y luego ejecuta el siguiente comando para instalar el plugin  

```shell
helm plugin install https://github.com/futuresimple/helm-secrets
```

A continuación necesitamos crear el fichero `.sops.yaml` en el root del repo y nos quedaría algo parecido a esto:
```yaml
---
creation_rules:
  - kms: 'arn:aws:kms:eu-west-2:999999999999:key/123a4b56-7c89-0ef1-gh23-i4j5k6lm7npk8'
```

Ahora falta saber como modificamos y creamos los ficheros... Pues tenéis dos opciones:
- Usando sops. De esta manera no tenéis que molestar en encriptar y desencriptar el fichero todo el rato. 
    ```shell
    sops secrets.yaml
    ```
- Con el plugin de helm
    - Crear el fichero con el plugin de helm
        ```shell
        vim secrets.yaml 
        # no intentéis encriptar un fichero vacío porque no os deja. Necesitáis al menos agregar una key y un valor
        helm secrets enc secrets.yaml
        ```
    - Modiciar el fichero con el plugin de helm
        ```shell
        helm secrets dec secrets.yaml
        vim secrets.yaml.dec
        helm secrets enc secrets.yaml
        rm secrets.yaml.dec
        ```
Se me olvidaba si queréis modificarlo sin tener que estar en donde tenéis el fichero `.sops`. Necesitáis tener una variable del entorno agregada en vuestro entrono 
```shell
export SOPS_KMS_ARN="arn:aws:kms:eu-west-2:999999999999:key/123a4b56-7c89-0ef1-gh23-i4j5k6lm7npk8"
```

## Como Usar Helmfile

Helmfile es muy sencillo a la hora de user vamos alli a donde tengáis un helmfile.yaml y yo os recomiendo ejecutar el siguiente comando:
```shell
helmfile -e minikube apply 
```

Tenéis que poner siempre en entorno al que queréis implementar las charts por eso siempre que ejecutéis el comando tenéis que poner `-e` seguido del entorno al que vais a mandarlo, eso si, lo tenéis que tener puesto en la lista que tengáis en el `environments.yaml`.

Y esto ha sido todo senores, espero que os haya gustado y nos leemos en el próximo.