---
layout: post
title: "How to remove Sculptor?"
description: ""
category: 
tags: [Sculptor]
author: Patrik Nordwall
navbar_name: blog
---
{% include JB/setup %}

When taking the decision to try a tool, such as Sculptor, it is important to know what it takes to stop using it if doesn't turn out as expected. Therefore I will in this post explain how to remove Sculptor. You don't have to do all steps and you don't have to do everything at once.

**Step 1**

The first step is to remove the code generation. This will take you less than 10 minutes.

a) In the maven pom.xml you remove the following:

  * plugin fornax-oaw-m2-plugin
  * plugin maven-clean-plugin
  * plugin fornax-checksum-m2-plugin
  * dependency fornax-cartridges-sculptor-generator

b) Remove these files in src/main/resources:

  * model files, .btdesign, .guidesign
  * sculptor-generator.properties
  * templates/SpecialCases.xpt
  * extensions/SpecialCases.ext

c) Move generated files

  * src/generated/java -> src/main/java
  * src/generated/resources -> src/main/resources
  * src/test/generated/java -> src/test/java
  * src/test/generated/resources -> src/test/resources

d) Run mvn eclipse:eclipse to generate eclipse project without generated source directories

**Step 2**

You don't need to do more right away, but later you would probably like to refactor the gap classes. Separation of generated and manually written code is done by a generated base class and manually written subclass. E.g. Person and PersonBase. You would probably like to collapse those into one class. To do this you can use the 'Push Down' refactoring in Eclipse, or simply copy paste the code from the base class to the subclass, and remove the extension.

This takes a few minutes per class, so you should be able to do a whole project within a few hours. It is a trivial and safe refactoring.

**Step 3**

Sculptor is a development tool, but it also has a small runtime library, with classes of utility and general character. This [blog post][1] explains why we have a runtime library. You can continue to use those classes as is. If you feel uncomfortable with that you can copy the source and remove classes that you don't use. The runtime library is not advanced at all, and you will have no problems to understan, maintain and change it as you need.

   [1]: /2010/01/21/why-we-combine-code-generator-with-runtime-library