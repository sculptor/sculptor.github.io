---
layout: post
title: "Improved Graphviz Visualization"
description: ""
category: 
tags: [Sculptor,Visualization]
author: Patrik Nordwall
navbar_name: blog
---
{% include JB/setup %}

I have improved the visualization with several new features. You can use this new diagram generator right away as described in the end.

Several diagrams with different focus and level of detail are generated. Below are samples of the different types of diagrams. You can click on the images below to see them in full scale.

Module dependencies:

![][1]

Overview:

![][2]

One separate diagram for each module:

![][3]

Diagram with focus on elements marked as core domain (`hint="umlgraph=core"`). Note that core domain objects also have another font color (and background color) in all diagrams.

![][4]

And finally, a diagram with full detail of all objects.

![][5]

All [colors can be configured][8] using the `sculptor-generator.properties` file and it is also possible to set the color for individual elements using a hint in model file (`hint="umlgraph.bgcolor=FFCC99"`). It is also possible to hide elements using `hint="umlgraph=hide"`.

To create images (png) from the generated [Graphviz][6] dot files you can use the goal `generate-images` of [Sculptors maven plugin][7]. Add the following to your `pom.xml`:

~~~ xml
<build>
	<plugins>
		<plugin>
			<groupId>org.sculptorgenerator</groupId>
			<artifactId>sculptor-maven-plugin</artifactId>
			<version>3.0.0</version>
			<executions>
				<execution>
					<id>image-generation</id>
					<goals>
						<goal>generate-images</goal>
					</goals>
				</execution>
			</executions>
		</plugin>
	</plugins>
</build>
~~~

You also need to install the excellent [Graphviz][6] tool.
{: .alert .alert-error}

   [1]: /images/2010-03-20-improved-graphviz-visualization/umlgraph-dependencies.dot.png
   [2]: /images/2010-03-20-improved-graphviz-visualization/umlgraph-overview.dot.png
   [3]: /images/2010-03-20-improved-graphviz-visualization/umlgraph-person.dot.png
   [4]: /images/2010-03-20-improved-graphviz-visualization/umlgraph-core-domain.dot.png
   [5]: /images/2010-03-20-improved-graphviz-visualization/umlgraph.dot.png
   [6]: https://www.graphviz.org/
   [7]: /documentation/maven-plugin
   [8]: /documentation/developers-guide#diagram

