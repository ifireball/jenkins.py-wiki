This tutorial is determined for developers which want to use some Python scripts or even libraries in their Jenkins plugin (written in Java). It's really easy, you just have to follow these steps:


1. Mark the python-wrapper plugin dependency in your plugin's _pom.xml_:
    ```xml
<dependencies>
...
      <dependency>  
        <groupId>org.jenkins-ci.plugins</groupId>
        <artifactId>python-wrapper</artifactId>
        <version>1.0.2</version>
      </dependency>
...
</dependencies>
```

2. Mark all .py files as resource files in your plugin's _pom.xml_:
    ```xml
<build>
      <resources>
      ...
        <resource>
          <directory>src/main</directory>
          <includes>
            <include>**/*.py</include>
          </includes>
        </resource>
      ...
      </resources>
</build>
```

3. Don't forget to determine all other resource directories which you are using (like _resources_ or _webapp_):
    ```xml
<build>
      <resources>
      ...
        <resource>
          <directory>src/main/resources</directory>
          <filtering>false</filtering>
        </resource>
      ...
      </resources>
</build>
```

4. Add your Python files to the _src/main/python_ directory. All scripts should be compatible with the [Jython 2.5.3](http://www.jython.org/).

5. Init and use a PythonExecutor object in your Java class:
    ```java
...
import jenkins.python.DataConvertor;
import jenkins.python.PythonExecutor;
...
class MyClass {
        private transient PythonExecutor pexec;
        ...
        public MyClass() {
            pexec = new PythonExecutor(this);
        }
        ...
        private void someMethod() {
            // call some_function("some example", 12) in my_class.py file and return a result
            boolean result = pexec.execPythonBool("some_function", "some example", DataConvertor.fromInt(12));
        }
        ...
}
```
    [See](https://github.com/jenkinsci/jenkins.py/tree/master/python-wrapper/src/main/java/jenkins/python) other _PythonExecutor.exec*()_ and _DataConvertor.from*()_ methods for all possibilities.

6. Add a Python script to the _src/main/python_ directory, which has the same name as your Java class, but in the Python file name convention. For the previous example it's _my\_class.py_. You can also import and use another modules including all Jenkins packages and classes:
    ```python
...
# file src/main/python/my_class.py
...
import re
import mymodule
import hudson.Util as Util
...
def some_function(text, length):
        if len(text) == length:
            return True
        else:
            return False
...
```

7. Test the plugin with `$ mvn hpi:run`.
8. Done.