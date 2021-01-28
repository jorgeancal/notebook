+++
title= "Configurar tu kluster con Helmfile"
date= "2021-01-24"
comments = true
categories = ["Helm", "kubernetes", "How to", "helmfile"]
description = "Echamos un vistazo atrás de cómo se ha portado helmfile después de 6 meses usando helmfile en el trabajo y mostramos a cómo configurarlo."
author = "Jorge Andreu Calatayud"
tags= ["helm", "helmcharts", "kubernetes", "helmfile" ]
+++

Helmfile es una herramienta que te permite sacar más partido a Helm. Ya que si usas helmfile puedes implementar cualquier cantidad de helmcharts. Básicamente con helmfile declaras las helmcharts y les das los valores que tú quieres a cada una de las helm chart, helmfile creara el correspondiente deploy mediante helm para mandar a tu cluster todo lo que has definido en helmfile, claro esta puedes decirle a helmfile impleméntame solo este grupo de helmchart y de este grupo quiero implementarlas en esta secuencia. 

## ¿Cómo se ha portado helmfile? 
Después de estos 6 meses usando helmfile la verdad es que me ha dado mucha más vida. Si queremos implementar cosas nuevas cosas.... todo va más rápido. 

## ¿Qué me ha gustado de helmfile?
Después estar usando lo un tiempo, esto es lo que más me ha gustado de helmfile:

- Lo que más vas a notar usando helmfile es que el tiempo de implementación de las helmchart habrá descendido. Por ejemplo, en donde trabajo hemos notado una mejora de 15~30 minutos comparado a lo que usábamos anteriormente que era `Landscaper`, actualmente obsoleto. Dejo aquí un enlace al [repositorio](https://github.com/Eneco/landscaper) por si alguien está interesado.  


- Otra cosa que he notado es que gracias a helmfile tengo la posibilidad de tener toda la configuration de diferentes entornos en un mismo sitio. Cuando estoy trabajando con minikube y le indico que me implemente las chart con los valores que tiene para minukube lo hace sin rechistar.


- `diff`. Posiblemente esto es lo mejor de todo... es estúpidamente sencillo y te salva más de una vez cuando estás haciendo actualizaciones de charts o estás cambiando valores en la chart y estás seguro si te has cargado algo.


- Variables de entorno. Puedes implementar `sops` en helmfile pero el plugin que estaban usando lo han jubilado asi que... me pase al uso de variables de entorno es lo mejor actualmente, ya que puedes tener los valores que tú quieres cuando desarrollas localmente y tener unos valores globales en tu CD/CI software.  Si queréis que hable sobre la implementacion de `SOPS` en helmfile solo decirlo y hare otro post sobre ello.

## ¿Cómo lo configuro?

La configuración es bastante sencilla. Lo primero de todo es tener un repo para todo esto o en una carpeta si vas a tener lo en un mismo repo con más cosas en él. Yo actualmente lo tengo en un repo exclusivamente para helmfile. Mi esquema del repo es el siguiente:


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

Ahora os voy a mostrar lo que tenemos en el principal helmfile.yaml:

```yaml
{{ readFile "base/templates/helmfile.yaml" }}

{{ readFile "base/repositories/helmfile.yaml" }}

releases:

```

Como veréis no tengo nada en releases y eso es porque tengo un script que va por todas las carpetas de dentro de releases y me encuentra todo fichero que se llame `helmfile.yaml` y me lo agrega al final del documento. De esa manera nos libramos de tener un fichero de más 200 lineas en el repo y que da más bonito a la hora de leer el fichero y te da más flexibilidad, ya que solo has de crear tu carpeta con la chart que quieres y los valores que tú quieres y listo. Os preguntaréis por qué ha puesto dos readFile ahi arriba. Bueno pues... esta es la manera que encontre de mantener las cosas simples. Ya que en templates puedo crear diferentes templates para los releases y luego decirle a un release tú usa esta y tú usa la otra. Una gozada. Luego también tenemos los repositorios que como todos sabemos para declarar un repositorio en helmfile tenemos que tener dos lineas, esto quiere decir que tendríamos dos lineas más de fichero tontamente, pues mejor es tenerlo en otro fichero no? asi todo queda más limpio. A continuación os mostraré un ejemplo de los ficheros, estar atengo que uno lleva sorpresa.

Empezamos por el más sencillo de todo que es el de los repositorios. 

```yaml
repositories:
  - name: prometheus-community
    url: https://prometheus-community.github.io/helm-charts
  - name: grafana
    url: grafana https://grafana.github.io/helm-charts
```

A continuacion tenemos en de la template
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

Como podéis ver lleva sorpresa no solo tenemos templates tenemos la section base ahi. La verdad es que esto si queremos podríamos agregar otro fichero y poner otro `readFile` esto ya fue a mi gusto, ya que cuando estaba haciendo la template quería agregar la base, por lo que pense que deberían ir juntos.

Ahora pasamos a la parte de los defaults
```yaml
helmDefaults:
  wait: true
  timeout: 300
```

Ahora vamos a irnos a mirar nuestro `environments.yaml`. Como podéis ver, declaramos los dos entornos que tenemos y luego añadimos los valores atraves de un fichero. En este fichero le estamos indicando cuales charts queremos implementar en el entorno.

```yaml
environments:
  production:
    values:
      - base/values/production/values.yaml.gotmpl
  minikube:
    values:
      - base/values/minikube/values.yaml.gotmp
```

Os dejo también un ejemplo de esto. He de comentar que este es un buen lugar para agregar variables globales para el entorno no solo para decirle que es lo que quieres instalar y lo que no.

```yaml
prometheus:
  installed: true
grafana:
  installed: true
```

A continuación vais a ver el `helmfile.yaml` que tengo para grafana. Es de lo mas sencillo ya que llamo a la template en el principio del release y luego agrego los datos para el release, además como veréis tenemos la key `installed` que de esta manera le digo si lo quiero instalar o no y también tengo otra key para decirle que no lo quiero implementar hasta que la lista de charts esté implementada

```yaml
- <<: *defaultTmpl
  name:  "grafana"
  chart: "grafana/grafana"
  namespace: "monitoring"
  version: "3.2.5"
  installed: {{ .Values | getOrNil "grafana.installed" | default false }}
  needs: 
    - fluentd
    - prometheus
    - jaeger
    - istio-operator
  values:
    - values.yaml.gotmpl.gotmpl
```

Y con esto hemos terminado la configuración de helmfile.

## Como Usar Helmfile

Helmfile es muy sencillo a la hora de user vamos alli a donde tengáis un helmfile.yaml y yo os recomiendo ejecutar el siguiente comando:

```shell
helmfile -e minikube apply 
```
Teneis más comandos para el uso de helmfile. Yo suelo usar bastante el `lint` y el `diff`.

Tenéis que poner siempre en entorno al que queréis implementar las charts por eso siempre que ejecutéis el comando tenéis que poner `-e` seguido del entorno al que vais a mandarlo, eso si, lo tenéis que tener puesto en la lista que tengáis en el fichero de entornos.

Para terminar voy a mostraros el script que uso para buscar y agregar los ficheros al fichero principal

```shell
for release in `find releases/ -name "*.yaml"`; do
    release_name=$(cat $release | grep "name: " | cut -d' ' -f2-)
    echo "Templating $release to helmfile.yaml"
    cat $release | sed 's/\(.*\)/  \1/' >> helmfile.yaml
    echo >> helmfile.yaml
done
```

Y esto ha sido todo senores, espero que os haya gustado y nos leemos en el próximo.


