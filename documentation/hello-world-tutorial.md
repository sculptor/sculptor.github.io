---
layout: page
title: "Hello World Tutorial"
description: ""
navbar_name: docs
---
{% include JB/setup %}

This hands-on tutorial will walk you through the steps of how to create a small application and explore it with some JUnit tests. This example is also used and extended in 3 other tutorials:

This is an introduction to Sculptor. A more extensive example is available in the [Advanced Tutorial][1]. If you would like to see something more exciting than running JUnit tests we can recommend the [CRUD GUI Tutorial][2].

Before you start you must follow the instructions in the ****[Installation Guide**][3]**.

**Table of Contents:**

* toc
{:toc}

## Part 1 - Setup Project

In this first part we will setup the project structure for maven and eclipse.

1\. Use the following command (one line) to create a maven pom and file structure. You can change the groupId and artifactId if you like.

Fill in groupId and artifactId:

There will be warnings like this:

Ignore these warnings and continue with next step if you see no errors.

2\. In the new directory, run `mvn eclipse:eclipse` to create an Eclipse project with the same dependencies as in the pom.

3\. Open Eclipse and import the project.

## Part 2 - Generate Code

In this part we will write a Sculptor DSL file and generate code from it.

1\. Modify the file named `model.btdesign` in the folder
`src/main/resources/`

2\. Open the `model.btdesign` file with Sculptor DSL editor, double-click on it.
Add something like this to the design file.

Try the code completion, error highlight and outline view.
It is a Module containing one Entity, with a Repository. The concepts are taken from [Domain-Driven Design][4].

3\. Run `mvn clean install` to generate code and build. The JUnit test will fail.

If you run maven from the command prompt you have to do a refresh in Eclipse. If you run maven as an external task in Eclipse it can refresh automatically.

4\. Look at the generated code. In `src/main/java`, `src/main/resources`, `src/test/java` and `src/test/resources` folders the code is only generated once, and you can do manual changes. In `src/main/generated/java`, `src/main/generated/resources`, `src/test/generated/java` and `src/test/generated/resources` it is generated each time, i.e. don't touch.

## Part 3 - Fix Failing Test

In this step we will fix the failing JUnit test and add some hand written code.

1\. Run `PlanetServiceTest` as JUnit Test. Red bar.
Adjust the test method `testSayHello` to something like this:

2\. [HSQLDB][5] is used as in memory database when running JUnit. Add test data in `src/test/resources/dbunit/PlanetServiceTest.xml`

3\. Run, still red, but another failure.

4\. Implement method `sayHello` in `PlanetServiceImpl`.

5\. Run. Green bar! ![][6]

6\. Add one more test method to test a failure scenario.

7\. Add `PlanetNotFoundException` in `model.btdesign`.

8\. Regenerate with

9\. Add `throws PlanetNotFoundException` in `PlanetServiceImpl.sayHello`.

10\. Fix the import of PlanetNotFoundException in the test class and run it, red bar. ![][7]

11\. Fix the test. You need to adjust `sayHello` method.

12\. Run. Green bar! ![][6]

13\. Run `mvn clean install`. Build success.

You can use `mvn -o -npu install` to speed up the builds, -o == offline, -npu == no plugin upate.
To regenerate you use `mvn -Dfornax.generator.force.execution=true -o -npu generate-sources`

## Source

The complete source code for this tutorial is available in Subversion.

Web Access (read only):


Anonymous Access (read only):


Unknown macro: {rating}

   [1]: /confluence/pages/viewpage.action?pageId=1141 (3. Advanced Tutorial (CSC))
   [2]: /confluence/pages/viewpage.action?pageId=1783 (5. CRUD GUI Tutorial (CSC))
   [3]: /confluence/pages/viewpage.action?pageId=1138 (1. Installation Guide (CSC))
   [4]: http://domaindrivendesign.org/books/index.html
   [5]: http://hsqldb.org/
   [6]: http://fornax.itemis.de/confluence/images/icons/emoticons/thumbs_up.png
   [7]: http://fornax.itemis.de/confluence/images/icons/emoticons/thumbs_down.png
  
