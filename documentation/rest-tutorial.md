---
layout: page
title: "REST Tutorial"
description: ""
navbar_name: docs
---
{% include JB/setup %}

This hands-on tutorial will walk you through the steps of how to use the REST
support in Sculptor. Sculptor makes it very easy to expose restful services by
using conventions and easy delegation to Services to reduce boilerplate coding.
[Spring MVC][1] is the underlaying REST framework. Spring configuration for
html, json and xml representations are generated.

**Table of Contents:**

* toc
{:toc}

## Setup Project

Run Part1 of [2\. Hello World Tutorial (CSC)][2] to create maven and Eclipse projects.

Modify pom.xml to make it a web application

Add some dependencies to pom.xml

Add jetty plugin in pom.xml

## Initial Model

Add the following to model.btdesign:

Generate code with `mvn generate-sources`. Look at the generated HelloWorldResource.java. The @RequestMapping path and HTTP method are defined by convention, i.e. the path is the name of the resource and the HTTP method is by default GET. You can also explicitly define these in the model:

Complete the code in HelloWorldResource with something like this (most of it should already be there):

The `return` attribute in the model indicates that this operation will show a view with specified name. This also result in a generated JSP file in `src/main/webapp/WEB-INF/jsp/helloworld/hello.jsp`.
The JSP is only generated once and you need to complete it. Maybe showing the msg.

Try it by starting jetty with `mvn jetty:run` and browse to 

Servlet and Spring configuration is generated in web.xml and rest-servlet.xml. By default, 3 content type representations are supported; html, xml and json. When testing xml and json it is convenient to use [cURL][3]. Try this from console:

## CRUD Resource

Let's move on to a simple CRUD service. Add the following to model.btdesign:

Generate code. No hand written code is needed.

Start jetty or wait for jetty reload. Open browser at  and try the simple CRUD functionality. Note that the generated JSPs are working for this simple example, but they are only intended to be a starting point for writing your own user interface.

You can try the xml and json representations also. For example, to POST data for a new Planet

Note that there are two ways to POST to the resource, either with the data in the request body as above, or with form data, as in the html form. Form data can be produced with cURL also:

Retrieve the new Planet as xml:

Note that fully qualified class names are used in the xml. Probably not what you want. Easily fixed, by adding the following to your sculptor-generator.properties @XStreamAlias will be generated.

There is a shortcut for generating CRUD operations. The above example can be made shorter by using the the `scaffold` keyword.

## A note regarding update - PUT

It is easy to define operation for PUT:

However, this will not work if the underlaying domain object (Planet) is an Entity or ValueObject, because those have some attributes that are not possible to modify (no setter methods). id, uuid, key and other not changeable attributes. This means that the data that is PUT can't be automatically mapped to the domain object instance. Therefore, for update you must use DataTransferObject with all attributes defined and do the mapping manually.

In the Resource gap code you need to write the mapping between Entity and DataTransferObject.

## Data Transfer Object

When you provide a service to external parties you probably want to expose DataTransferObjects (DTO) instead of domain objects, to have the freedom of changing the internal implementation without having to change the published API. You should also use DTOs when the external representation need to be different than the internal domain, which is often the case. The drawback is of course the additional mapping layer between DTOs and domain objects.

You have the choice of implementing the mapping in the Resource, as described in the section about Update/PUT, or in a separate Service. Using a separate Service has the benefit that it can be used for other communication protocols also, such as Spring remoting, which is supported by Sculptor.

It can be defined like this in the model:

This means that you need to write minimal hand written code in the Resource class. The hand written mapping between DTOs and domain objects is implemented in the service (PlanetService). [Dozer][4] is a tool that might be useful for doing the mapping.

## Additional Spring MVC Features

[Spring MVC][1] has powerful possibilities to declare the methods in the controllers (resources). Some of these are supported by Sculptor, others require that you write the method yourself in the gap class of the resource.

To define matching headers you can use the hint `headers`.

You can narrow path mappings through parameter conditions and that is supported with hint `params`. The [sample from Spring MVC reference][5] would look like this:

There are some types in the method signature that are special, and useful when you need more fine grained control of the request/response. All these can be defined as parameters of the Resource operations in the model.

  * Model, ModelMap, Map
  * BindingResult
  * HttpServletRequest, HttpServletResponse, HttpSession, WebRequest
  * Locale
  * java.io.InputStream, java.io.Reader, java.io.OutputStream, java.io.Writer
  * java.security.Principal
  * org.springframework.validation.Errors

Example:

## Hypermedia

So far we have not talked about hypermedia, i.e. navigation by following links. Hypermedia as the engine of application state (HATEOAS) was coined to describe a core tenet of the REST architectural style. One can claim that without using links it is not restful. You have to decide if links are important for your application. Sculptor and Spring MVC doesn't have any special support for links, since they are very application specific and should be defined in runtime depending on things like user's roles and current state. It would be wrong to try to define the rules for the links in the static `model.btdesign`.

A quick example, inspired by the Restbucks sample in the excellent book [REST in Practice][6], of how to define a Resource with links. The links are simply part of the resource representations.

## Customization

### Conventions

The annotations that are generated for the controller methods are by default based on conventions. These conventions are not hard coded. They are defined in sculptor-default-generator.properties, which makes it easy for you to redefine and add your own conventions.

The complete set of properties for this:

Note that you can redefine individual properties in your sculptor-generator.properties to adjust the conventions.

You can define your own conventions. For example, in case you would like to define a generic filter operation. Add the following to your sculptor-generator.properties:

In the model you can then simply use:

You need to implement the findAllMatching method yourself in the Service, but the delegation mechanism in the resource is completely generated.

### XStreamAlias

You should add the following property to your sculptor-generator.properties to avoid fully qualified class names the xml representations. @XStreamAlias will be generated.

There is one quirk regarding the detection of the alias. Everything works fine when serializing object to xml, i.e. result of ordinary GET operation, but if the first usage of class is deserialization of xml to object the alias is not initialized yet. This is explained in [XStream documentation][7]. Therefore you need to explicitly define the aliases for classes that are used in POST or PUT parameters. Define those in the xstreamMarshaller in rest-servlet.xml

### JAXB

By default, [XStream][8] is used for the XML representation. You can use JAXB instead. In `rest-servlet.xml` you can remove the comment for jaxb2Marshaller and replace references to xstreamMarshaller with jaxb2Marshaller. Note that you must define all classes that are to be supported in the classesToBeBound definition of the Jaxb2Marshaller.

JAXB annotations are generated in DataTransferObjects. If you expose other types of domain objects and use JAXB you can turn on generation of JAXB annotations with these properties in sculptor-generator.properties.

### Separate module

This tutorial explained how to adjust the business tier project to include the Resources and web app configuration. If you like you can define this in a separate project and reference the services in the business tier services with the ordinary import mechanism in model.btdesign.

It is also possible to combine [JSF web presentation tier][9] project with REST. In that case you need to define the following properties in sculptor-gui-generator.properties.

### Location of webapp directory

By default the web configuration and jsp pages are generated in src/main/webapp. You can change that by defining `outlet.webroot.dir` in src/main/resources/generator/Workflow.mwe2. For Google Appengine projects it should be located in the `war` directory:

### Skip generation of JSP

If you don't need html representation or use another view technology than JSP (e.g. Facelet xhtml) you can omit generation of JSPs by defining the following property in sculptor-generator.properties.

Unknown macro: {rating}

   [1]: http://static.springsource.org/spring/docs/3.0.x/reference/mvc.html
   [2]: /confluence/pages/viewpage.action?pageId=1139 (2. Hello World Tutorial (CSC))
   [3]: http://curl.haxx.se/
   [4]: http://dozer.sourceforge.net/
   [5]: http://static.springsource.org/spring/docs/3.0.x/reference/mvc.html#mvc-ann-requestmapping-advanced
   [6]: http://restinpractice.com/
   [7]: http://xstream.codehaus.org/annotations-tutorial.html#AutoDetect
   [8]: http://xstream.codehaus.org/
   [9]: /confluence/pages/viewpage.action?pageId=2508 (5.1 Web CRUD GUI Tutorial (CSC))
  
