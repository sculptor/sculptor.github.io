---
layout: page
title: "Hello World Tutorial"
description: ""
navbar_name: docs
---
{% include JB/setup %}

This hands-on tutorial will walk you through the steps of how to create a small application and explore it with some JUnit tests. This example is also used and extended in 3 other tutorials:

* [Archetype Tutorial](archetype-tutorial)
* [Pure EJB3 Tutorial](pure-ejb3-tutorial)
* [REST Tutorial](rest-tutorial)

This is an introduction to Sculptor. A more extensive example is available in the [Advanced Tutorial][1]. If you would like to see something more exciting than running JUnit tests we can recommend the [Developers Guide][2].

Before you start you must follow the instructions in the **[Installation Guide][3]**.
{: .alert}

**Table of Contents:**

* toc
{:toc}


## Part 1 - Setup Project

In this first part we will setup the project structure for Maven and Eclipse.

1. Use [Sculptors Maven archetype][8] with the following command (**one line** - indicated by the trailing `\`) to create a new project with [Maven POM](http://maven.apache.org/guides/introduction/introduction-to-the-pom.html) and file structure. You can change the groupId and artifactId if you like.

   ~~~
   mvn archetype:generate -DarchetypeGroupId=org.sculptorgenerator \
      -DarchetypeArtifactId=sculptor-maven-archetype \
      -DarchetypeVersion={{site.sculptor_version}} \
      -Dstandalone=true
   ~~~

   The property `-Dstandalone=true` indicates that the project is **not** part of a [Maven multi-module project](http://maven.apache.org/guides/mini/guide-multiple-modules.html) and is **not** refering to a common parent project `helloworld-parent`.
   {: .alert} 

   Fill in groupId and artifactId:

   ~~~
   Define value for groupId: : org.helloworld
   Define value for artifactId: : helloworld
   Define value for version:  1.0-SNAPSHOT: :
   Define value for package:  org.helloworld: :
   ~~~

2. Open Eclipse and import the project via (via [m2e Eclipse plugin](http://www.eclipse.org/m2e/)) with "File > Import... > Existing Maven Projects".

3. Create an Eclipse m2e launch configuration for executing Maven (e.g. with the goal `generate-sources`) from within Eclipse (right-clicking the project or Maven POM and selecting "Run as > Maven build..." from the context menu).

   To refresh the Eclipse workspace after Sculptors code generator created new code enable the option "Refresh > Refresh resources upon completion > The project containing the selected resource" in the launch configuration.
   {: .alert}


## Part 2 - Generate Code

In this part we will write a Sculptor DSL file and generate code from it.

1. Open the file `model.btdesign` in the folder `src/main/resources/` with Sculptor DSL editor.
Add something like this to the design file:

   ~~~
   Application Universe {
       basePackage=org.helloworld
    
       Module planet {
           Service PlanetService {
               String sayHello(String planetName);
               protected findByExample => PlanetRepository.findByExample;
           }
    
           Entity Planet {
               String name key
               String message
    
               Repository PlanetRepository {
                   findByExample;
               }
           }
       }
   }
   ~~~

   Try the code completion, error highlight and outline view.
It is a [Module](advanced-tutorial#module) containing one [Entity](advanced-tutorial#entity), with a [Repository](advanced-tutorial#repository). The concepts are taken from [Domain-Driven Design][4].

2. Run `mvn clean install` to generate code and build. The JUnit test will fail.

   <span class="badge badge-important">!</span>
   If the Maven build aborts with the error message `Executing 'dot' command failed` then the GraphViz package is not installed as described in the [installation guide](installation#graphviz).
   {: .alert .alert-success }

   If you run Maven from the command prompt you have to refresh the Eclipse workspace. If you start the Maven build from within Eclipse with the aforementioned m2e launch configuration then the workspace is refreshed automatically.
   {: .alert }

3. Look at the generated code. In `src/main/java`, `src/main/resources`, `src/test/java` and `src/test/resources` folders the code is only generated once, and you can do manual changes. In `src/main/generated/java`, `src/main/generated/resources`, `src/test/generated/java` and `src/test/generated/resources` it is generated each time, i.e. don't touch.


## Part 3 - Fix Failing Test

In this step we will fix the failing JUnit test and add some hand written code.

1. Run `PlanetServiceTest` as JUnit Test. Red bar.
Adjust the test method `testSayHello` to something like this:

   ~~~ java
   public void testSayHello() throws Exception {
       String greeting = planetService.sayHello(getServiceContext(), "Earth");
       assertEquals("Hello from Earth", greeting);
   }
   ~~~

2. [HSQLDB][5] is used as in memory database when running JUnit. Add test data in `src/test/resources/dbunit/PlanetServiceTest.xml`:

   ~~~ xml
   <?xml version="1.0" encoding="UTF-8"?>
    
   <dataset>
     <PLANET id="1" name="Earth" message="Hello from Earth"
       LASTUPDATED="2006-12-08" LASTUPDATEDBY="dbunit" version="1" />
     <PLANET id="2" name="Mars" message="Hello from Mars"
       LASTUPDATED="2006-12-08" LASTUPDATEDBY="dbunit" version="1" />
   </dataset>
   ~~~

3. Run, still red, but another failure.

4. Implement method `sayHello` in `PlanetServiceImpl`:

   ~~~ java
   public String sayHello(ServiceContext ctx, String planetName) {
       Planet planetExample = new Planet(planetName);
       List<Planet> foundPlanets = findByExample(ctx, planetExample);
       Planet planet = foundPlanets.get(0);
       return planet.getMessage();
   }
   ~~~

5. Run. Green bar! ![][6]

6. Add one more test method to test a failure scenario:

   ~~~ java
   @Test
   public void testSayHelloError() throws Exception {
       try {
           planetService.sayHello(getServiceContext(), "pluto");
           fail("Expected PlanetNotFoundException");
       } catch (PlanetNotFoundException e) {
           // as expected
       }
   }
   ~~~

7. Add `throws PlanetNotFoundException` in `model.btdesign`:

   ~~~ java
   String sayHello(String planetName) throws PlanetNotFoundException;
   ~~~

8. Regenerate with `mvn generate-sources -Dsculptor.generator.force=true`

9. Add `throws PlanetNotFoundException` in `PlanetServiceImpl.sayHello`.

10. Fix the import of `PlanetNotFoundException` in the test class and run it, red bar. ![][7]

11. Fix the test. You need to adjust `sayHello` method:

    ~~~ java
    public String sayHello(ServiceContext ctx, String planetName)
            throws PlanetNotFoundException {
        Planet planetExample = new Planet(planetName);
        List<Planet> foundPlanets = findByExample(ctx, planetExample);
        if (foundPlanets.isEmpty()) {
            throw new PlanetNotFoundException("Didn't find any planet named " + planetName);
        }
        Planet planet = foundPlanets.get(0);
        return planet.getMessage();
    }
    ~~~

12. Run. Green bar! ![][6]

13. Run `mvn clean install`. Build success.

<span class="label label-info">Tip</span>
You can use `mvn -o install` to speed up the builds (-o == offline).
To enforce regeneration you use `mvn generate-sources -Dsculptor.generator.force=true`
{: .alert}


   [1]: advanced-tutorial
   [2]: developers-guide
   [3]: installation
   [4]: http://domaindrivendesign.org/books/index.html
   [5]: http://hsqldb.org/
   [6]: /images/emoticons/thumbs_up.png
   [7]: /images/emoticons/thumbs_down.png
   [8]: maven-archetypes#sculptor-maven-archetype
