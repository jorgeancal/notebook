+++
comments= true
author = "Jorge Andreu Calatayud"
categories = ["Java", "Docker"]
tags = ["java", "openjdk", "jdk", "DockerImages", "Docker", "jvm", "jre", "maven", "Spring Boot"]
date = "2020-09-28"
description = "Here, you will learn about how to create a small JRE for your microservices. "
title= "jlink with Spring Boot services"
+++

This always has been my priority because... I don't want to pay extra in ECR so... I have to create small docker images.

we need to list the classpath of all the libraries that we are going to use the follow command in maven ot save all of them in a file

```shell
mvn dependency:build-classpath -Dmdep.includeScope=runtime -Dmdep.outputFile=classpath
```

once we have all the libraries in a file... we need are going to put everything in a environment variable that way we can use it later on. To be able to do that... we need to run the following command

```shell
export SERVICE_CLASSPATH=$(cat classpath)
```

Once we have the variable in place let's go to list all the java modules. For that we are going to run the following command:
```shell
jdeps -cp $SERVICE_CLASSPATH --multi-release $JDK --print-module-deps --ignore-missing-deps -R target/classes
```
Being `$JDK` the number of java version. Once we have all the modules it's time to create out slim jdk... for that we are going to need to run the following command (Being `$JDEPMODULES` the list of modules from the previous command)

```Shell script
$ jlink --module-path /opt/java/jmods --compress=2 --strip-debug \
  --no-header-files --no-man-pages \
  --add-modules $JDEPMODULES --output /opt/jlink 
```

Now, we have our runtime in `/opt/jlink` folder that we can use to run our application and this folder is going to be around 30 Mb instead of 240 Mb which is the normal size of the JDK. 

Nowadays we need to create automated tasks to make everything easier. These steps are really easy to set in a Dockerfile. If you do this you could create an image that is smaller than 100 Mb, maybe a bit more if you have a lot of dependencies. It may look difficult but it's not. I'll try to create another post showing how to do this in an automated way with the dockerfile. I hope you like this post so far and I'll see you in the next one. 


