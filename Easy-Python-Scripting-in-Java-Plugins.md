This tutorial is determined for developers which want to use some Python script or even a library in their Jenkins plugin (written in Java). It is really easy to do, you just have to follow these steps:
1. Mark the python-wrapper plugin dependency in your plugin's _pom.xml_:
`<dependency>
    <groupId>org.jenkins-ci.plugins</groupId>
    <artifactId>python-wrapper</artifactId>
    <version>1.0.2</version>
</dependency>
`