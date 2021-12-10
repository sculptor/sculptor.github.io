---
layout: post
title: "Sculptor Shipping example and overriding templates"
description: ""
category: 
tags: [Sculptor,Xtext,Xtend,Eclipse,Maven,Examples,MongoDB,Extension]
author: Sculptor Team
navbar_name: blog
---
{% include JB/setup %}

This post will describe the Sculptor Shipping example project, which is one of the [MongoDB example projects].  In particular, I'll describe how Sculptor generator templates are overridden in the Sculptor Shipping project.  This pattern can be applied to any project using Sculptor that needs to override Sculptor templates or transformations.

The Sculptor Shipping project is divided into two projects:

* [sculptor-shipping] - This is the Shipping project itself, containing the Shipping model, and generated and manually written application code
* [sculptor-shipping-generator] - This project contains code related to overriding Sculptor templates for the Shipping project.

The overriding-related code is in a separate project for a couple reasons:

* Separation of concerns: to keep code generation code from mingling with or being packaged with the actual application code.  The code generation code is only needed at build-time, and does not need to be bundled with the application.
* To fit in with the build lifecycle: the code generation-related code must be compiled, then used by the Sculptor generator to generate the application code.  It's much easier to make these two separate phases if the generation-related code is a separate project.

The sculptor-shipping project depends on the sculptor-shipping-generator project, and not vice-versa, because the Sculptor code generation process in sculptor-shipping must be able to use the overridden templates defined in sculptor-shipping-generator.  So based on the direction of this project dependency, sculptor-shipping-generator cannot refer to any sculptor-shipping resources or classes, and sculptor-shipping-generator must be built first, which happens automatically in the Eclipse and Maven builds.

## Shipping Generator Maven Project

The Shipping Generator Maven POM is configured to compile XTend code, and has a dependency on the following Sculptor libraries:

* sculptor-generator-library: Sculptor library containing Sculptor templates, transformations, and utility classes
* sculptor-generator-test: Utility clasess used for unit testing Sculptor generation code

You can see the above dependencies in the [Shipping Generator pom file], which otherwise is a basic Maven project with support for compiling XTend files.

### Overriding of Sculptor template

The sculptor-shipping-generator project includes a single template override, [DomainObjectReferenceTmplOverride], which overrides the Sculptor core [DomainObjectReferenceTmpl] class.  The [DomainObjectReferenceTmplOverride] override class generates an additional 'addTo\*' method for each one-to-many bidirectional and unidirectional reference in the model.  Below is a fragment of the override class showing how the 'addTo\*' method is added for bidirectional references.

~~~
@ChainOverride
/**
 * Override DomainObjectReferenceTmpl to add 'addTo*' methods for bidirectional and unidirectional references.
 */
class DomainObjectReferenceTmplOverride extends DomainObjectReferenceTmpl {

    @Inject extension HelperBase helperBase
    @Inject extension Helper helper


    override String bidirectionalReferenceAdd(Reference it) {
        '''
            «IF !isSetterPrivate()»
                /**
                 * Adds an object to the bidirectional many-to-one
                 * association in both ends.
                 * It is added the collection {@link #get«name.toFirstUpper()»}
                 * at this side and the association
                 * {@link «getTypeName()»#set«opposite.name.toFirstUpper()»}
                 * at the opposite side is set.
                 */
                «getVisibilityLitteralSetter()»void addTo«name.toFirstUpper().plural()»(«getTypeName()» «name.singular()»Element) {
                    add«name.toFirstUpper().singular()»(«name.singular()»Element);
                };
            «ENDIF»
            
            // Delegate to next template in chain so regular add method gets generated as well
            «next.bidirectionalReferenceAdd(it)»
        '''
    }
~~~

Note that for the template override class to be automatically detected and used by Sculptor, it must meet the requirements documented in the [Overrides and extension mechanism] section of the Adavanced tutorial.  Specifically, the class must:

* Follow the *\<template or transformation to be overridden\>*Override naming convention
* Be placed in the generator package
* Extend the template or transformation to be overridden
* Be marked with the `@ChainOverride` annotation


### Unit testing the Sculptor override template

Of course, we want to unit test all code, including template code.  This is easily done using XTend and Sculptor Generator test utilities.

The unit test for DomainObjectReferenceTmplOverride consists of a couple elements:

* Test model .btdesign files and sculptor-generator properties.  For the sculptor-shipping-generator tests, these are in [src/test/resources/generator-tests/library](https://github.com/sculptor/sculptor/tree/develop/sculptor-examples/mongodb-samples/sculptor-shipping-generator/src/test/resources/generator-tests/library).
* The [DomainObjectReferenceTmplOverrideTest] unit test itself

Let's take a look at [DomainObjectReferenceTmplOverrideTest], to see how it validates the template override code.

~~~
package generator

import org.junit.BeforeClass
import org.junit.Test
import org.sculptor.generator.test.GeneratorTestBase

import static org.sculptor.generator.test.GeneratorTestExtensions.*

class DomainObjectReferenceTmplOverrideTest extends GeneratorTestBase {

    static val TEST_NAME = "library"

    new() {
        super(TEST_NAME)
    }

    @BeforeClass
    def static void setup() {
        runGenerator(TEST_NAME)
    }

    @Test
    def void assertOverridenTemplateInMediaBase() {
        val mediaCode = getFileText(TO_GEN_SRC + "/org/sculptor/example/library/media/domain/MediaBase.java");
        assertContains(mediaCode, 'public void addToEngagements(Engagement engagementElement) {');
    }
}
~~~

In the above code, note a couple things:

* The test class extends [GeneratorTestBase], which provides the runGenerator() method used to execute the Sculptor generator using the test model
* The test indicates the location of the test model files and Sculptor properties by passing the name, 'library' into runGenerator(), which is called once before any tests run in order to generate all of the code for the library test model.
* [GeneratorTestExtensions] is statically imported, which provides helper assert methods to validate generated code, such as assertContains
* Since all of the code has already been generated in the setup() method, the assertOverridenTemplateInMediaBase() can simply load the generated code, and assert its expected contents.


[MongoDB example projects]: https://github.com/sculptor/sculptor/tree/develop/sculptor-examples/mongodb-samples
[Shipping Generator pom file]: https://github.com/sculptor/sculptor/blob/develop/sculptor-examples/mongodb-samples/sculptor-shipping-generator/pom.xml
[sculptor-shipping]: https://github.com/sculptor/sculptor/tree/develop/sculptor-examples/mongodb-samples/sculptor-shipping
[sculptor-shipping-generator]: https://github.com/sculptor/sculptor/tree/develop/sculptor-examples/mongodb-samples/sculptor-shipping-generator
[DomainObjectReferenceTmplOverride]: https://github.com/sculptor/sculptor/blob/develop/sculptor-examples/mongodb-samples/sculptor-shipping-generator/src/main/java/generator/DomainObjectReferenceTmplOverride.xtend
[DomainObjectReferenceTmpl]: https://github.com/sculptor/sculptor/blob/develop/sculptor-generator/sculptor-generator-templates/src/main/java/org/sculptor/generator/template/domain/DomainObjectReferenceTmpl.xtend
[Overrides and extension mechanism]: https://sculptorgenerator.org/documentation/advanced-tutorial.html#overrides-and-extension-mechanism
[DomainObjectReferenceTmplOverrideTest]: https://github.com/sculptor/sculptor/blob/develop/sculptor-examples/mongodb-samples/sculptor-shipping-generator/src/test/java/generator/DomainObjectReferenceTmplOverrideTest.xtend
[GeneratorTestBase]: https://github.com/sculptor/sculptor/blob/develop/sculptor-generator/sculptor-generator-test/src/main/java/org/sculptor/generator/test/GeneratorTestBase.xtend
[GeneratorTestExtensions]: https://github.com/sculptor/sculptor/blob/develop/sculptor-generator/sculptor-generator-test/src/main/java/org/sculptor/generator/test/GeneratorTestExtensions.xtend

