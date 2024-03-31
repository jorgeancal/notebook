+++
date = 2020-01-23T23:08:17Z
title = "What’s going on with Java?"
description = "Another version of java? See what it's going on with java and what they have to say about it."
tags = ["AdoptOpenJDK", "java", "JDK", "JDK 11", "JDK 11 LTS", "JDK 6", "JDK 7", "JDK 8","OpenJDK", "opinion"]
categories = ["Java", "Opinion"]
comments = true
images = ['images/oracleversion.png']
+++

The other day there were a few news articles about the new version of Java. I was thinking another one? Really? Like a lot of people in the beginning of their careers, no-one checks the differences between Java versions because they are focused on the projects which they are working on, or maybe it’s just me. But, I had a break between jobs so I decided to update all my knowledge and one thing from my list was this one, so I hope you find this information interesting. As everyone knows, Java has another JDK version which is the JDK 11.

One year ago, Oracle released JDK 9 which a lot of people said was really bad, Oracle released JDK 11 after JDK 10, that’s pretty obvious. But I learned to develop with JDK 7 which was released in July 2011 while I was studying vocational training. Three years later, I started university and I had to use the JDK 8 which a lot of people said was something gorgeous. If I have to tell you whether I realised about the differences between them, I didn’t. Furthermore, I didn’t mind, but now I’ve started to do some research about which would be better for all the applications which I want to do, so now I mind. I’m going to tell you what’s going to happen with Java in the next few years and which one I chose for my apps.


## The Future

It’s madness, but we’re developers so it’s something which you find out when you start to work in this crazy world. The future of Java is open source, but it depends on you or the business which you’re working for because if you want to use it… get your wallet out. I think for a business it wouldn’t be a problem, but for a person like me who cares about each penny that I spend, it’s a problem because I hadn’t planned to spend money on this. The good thing is, if you don’t want to pay, you can use OpenJDK, check the price of JDK, maybe it’s worth it for you:

|  Products/Metrics | N up | License | CPU  | Support |
|---|---|---|---|---|
| Java SE Advanced Desktop  | $40  | $8.80  | - | -  |
| Java SE Advanced | $100  | $22  | 5000  | $ 1100  |
| Java SE Suite  | $300 | $66  | 15000  |  $3300 |

Maybe you can say “well it’s not expensive” and I think it’s not really expensive if you have a successful application and you earn a ton of money with it. But, as I don’t know if my applications are going to be successful, I prefer to save some money because I don’t know when I’m going to need it. I guess in a business you’re not going to have any problem buying the license. I don’t know how it is in the rest in the world but, in Spain, clients don’t want to spend a bit more money on developing their applications, for that reason we cannot spend a lot of money on extra libraries, plugins or raise your salaries or that’s what our bosses tell us when we ask them for it.

## About Versions

When I read about this new version and that I would have to pay if I want to use JDK for my applications, I just started to laugh. After laughing a lot, I saw a Java version timeline. I hope no-one tells me you have to pay for this image too.

![Oracle Versions #incenter](/images/oracleversion.png)

As you can see in the image, there’s a good thing, which is OpenJDK which is public and free for life. If you have an application and you’re using JDK which is not OpenJDK, sorry but you’re falling down into a bottomless pit because you need to upgrade it urgently. If I were you, I would use OpenJDK, but I love to use Open Source.

Now I’m going to give you some information about the JDK 11 LTS, furthermore I’m going to give you some information about openJDK too.

## JDK 11 LTS

Well, it’s not a really good thing when you see LTS on a development kit and you don’t want to see it near Java. I cannot understand what exactly Oracle is thinking, but I think it’s not going a good way. When you see the versions picture and you see the updates and everything which they want to do, the first thing you think is… this is weird. They spent three years releasing one version and now every few months you’re going to have a ton of updates and versions. I’m a bit worried about that because I saw three versions of Java in one year, two of which no-one is using, and even Oracle said forget them are not good versions.

I hope everything will go well. At the moment, Oracle tell us about a few of the bugs which have been fixed in JDK 11 and they are:

- [181](http://openjdk.java.net/jeps/181): Nest-Based Access Control
- [309](http://openjdk.java.net/jeps/309)309: Dynamic Class-File Constants
- [315](http://openjdk.java.net/jeps/): Improve Aarch64 Intrinsics
- [318](http://openjdk.java.net/jeps/): Epsilon: A No-Op Garbage Collector
- [320](http://openjdk.java.net/jeps/): Remove the Java EE and CORBA Modules
- [321](http://openjdk.java.net/jeps/): HTTP Client (Standard)
- [323](http://openjdk.java.net/jeps/): Local-Variable Syntax for Lambda Parameters
- [324](http://openjdk.java.net/jeps/): Key Agreement with Curve25519 and Curve448
- [327](http://openjdk.java.net/jeps/): Unicode 10
- [328](http://openjdk.java.net/jeps/): Flight Recorder
- [329](http://openjdk.java.net/jeps/): ChaCha20 and Poly1305 Cryptographic Algorithms
- [330](http://openjdk.java.net/jeps/): Launch Single-File Source-Code Programs
- [331](http://openjdk.java.net/jeps/): Low-Overhead Heap Profiling
- [332](http://openjdk.java.net/jeps/): Transport Layer Security (TLS) 1.3
- [333](http://openjdk.java.net/jeps/): ZGC: A Scalable Low-Latency Garbage Collector (Experimental)
- [335](http://openjdk.java.net/jeps/): Deprecate the Nashorn JavaScript Engine
- [336](http://openjdk.java.net/jeps/): Deprecate the Pack200 Tools and API

If you want to know more about it without reading a lot, you can go to YouTube, Java has its own channel [here](https://www.youtube.com/channel/UCmRtPmgnQ04CMUpSUqPfhxQ).


## OpenJDK

You should use it, it’s the one I’m going to use. Why? Because I’m using Linux and I don’t want to use something which isn’t free. Free things are always the best and even more so if it’s free food. I have been reading a lot about it and it looks like I’m not going to have any problem when I change everything, but this is in theory. I’ll let you know if I have any problems with them, but I don’t think I’m going to have a problem.

If you want to install it, you have to go to the official [website](http://openjdk.java.net/), or this [one](https://adoptopenjdk.net/index.html?variant=openjdk11&jvmVariant=hotspot). AdoptOpenJDK provides prebuilt OpenJDK binaries from a fully open source set of build scripts and infrastructure.

In conclusion, if you don’t want to pay, use OpenJDK.
