---
layout: post
title:  "Maven Client too old to build"
date:   2015-08-19 23:24:58
categories: Java
---

#### [maven] Update SVN version
----

In the .pom.xml file,  find the following plugin in.

>```<groupId> com.google.code.maven-svn-revision-number-plugin</groupId>```

Add the following dependency, which upgrade the svn version to 1.8 (my current version)

~~~xml
<dependencies>
    <dependency>
        <groupId>org.tmatesoft.svnkit</groupId>
        <artifactId>svnkit</artifactId>
        <version>1.8.10</version>
    </dependency>
</dependencies>
~~~
