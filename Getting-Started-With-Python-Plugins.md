This tutorial is determined for developers which want to develop plugins for Jenkins in Python.
### How to start
1. First read the [Extend Jenkins](https://wiki.jenkins-ci.org/display/JENKINS/Extend+Jenkins) document. Everything there in the section _Developing Plugins_ applies also for the Python development.
2. Install latest JDK 7 and Maven tools. That is truly all you need for Python plugin development (with you favourite editor or IDE of course).
3. Look at [Existing Python Plugins](https://github.com/jenkinsci/jenkins.py/wiki/Existing-Python-Plugins) or generate a new plugin with the [PPSM tool](https://github.com/jenkinsci/jenkins.py/tree/master/ppsm) (you will need Python 3 for this) for the quick initiation.
4. Read following sections.


### How does it work
The concept of the jenkins.py project is not to provide pure Python plugin development, but it enables to implements every method of the extension or its descriptor in the Python. It means that there must be at least one Java class in every Python plugin, but with no functionality at all. Methods executions are delegated to functions in attached Python scripts.  
There is a runtime library plugin called [python-wrapper](https://wiki.jenkins-ci.org/display/JENKINS/Python+Wrapper+Plugin) which provides wrappers for every extension point and descriptor in the Jenkins system.