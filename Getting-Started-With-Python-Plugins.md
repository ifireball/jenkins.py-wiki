This tutorial is intended for developers who want to develop plugins for Jenkins in Python.


## How to start

1. First read the [Extend Jenkins](https://wiki.jenkins-ci.org/display/JENKINS/Extend+Jenkins)
   document. Everything there in the section _Developing Plugins_ applies also
   for the Python development.
2. Install latest JDK 7 and Maven tools. That is truly all you need for Python
   plugin development (along with your favourite editor or IDE of course).
3. Look at [Existing Python Plugins](https://github.com/jenkinsci/jenkins.py/wiki/Existing-Python-Plugins)
   or generate a new plugin with the [PPSM tool](https://github.com/jenkinsci/jenkins.py/tree/master/ppsm)
   (you will need Python 3 for this) for the quick initiation.
4. Read the following sections.


## How does it work

The goal of the _jenkins.py_ project is not to provide pure Python plugin
development, but to enable to implement every method of the extension or its
descriptor in the Python. It means that there must be at least one Java class in
every Python plugin, but with no functionality at all. Methods executions are
delegated to functions in attached Python scripts.

There is a runtime library plugin called
[python-wrapper](https://wiki.jenkins-ci.org/display/JENKINS/Python+Wrapper+Plugin)
which provides wrappers for every extension point and descriptor in the Jenkins
system. These wrappers have suffix `PW` (e.g. for `Notifier` it is `NotifierPW`)
and they are situated in packages `jenkins.python.expoint.*` (wrappers for
extension points) and `jenkins.python.descriptor.*` (descriptors wrappers).

If your `@Extension` class or its descriptor inherits from the wrapper class
instead of the original one, you can use an ability to implement methods in the
associated Python script. For example if you are implementing some _Recorder_
extension and it is called `MyRecorder` and this class inherits from
`RecorderPW`, there should be attached Python script with name `my_recorder.py`
which may or not contain implementations of `Recorder` methods.

Plugin python-wrapper uses [Jython 2.5.3](http://www.jython.org/docs/) for
executing associated Python scripts.

![Architecture diagram](https://github.com/jenkinsci/jenkins.py/raw/master/concept.png)

### Java and Python names conventions

There is a difference between Java and Python names convention. This fact
concerns filenames as well as methods/functions names.
E.g.: if your `@Extension` class is called `MyClass` and it inherits a method
`someMethod()` from the extension point, there should be the Python script
called `my_class.py` and it may contain a function `some_method()`.

### Manual Python execution delegation

All _public_ and _protected_ methods from the extension point and its parent
classes are automatically found in the Python script and used. However, there
could be more methods in the `@Extension` class whose executions have to be
delegated to the Python script manually. It can be done by the `execPython*`
methods:

```java
public ReturnType someMethod(ArgType1 arg1, ArgType2 arg2) {
    return (ReturnType)execPython("some_method", arg1, arg2);
}

public int someOtherMethod(ArgType1 arg1) {
    return execPythonInt("some_other_method", arg1);
}
```

This manual delegation concerns these methods:
*  Stapler methods (do* and get* methods which are called on demand by the UI)
*  Constructors
*  Interfaces methods (if your extension or its parent class (extension point)
   implements some interface, but does not implement interface methods)

You should use `DataConvertor` utility if some of the method's arguments has a
basic type:

```java
import jenkins.python.DataConvertor;
// ...
public ReturnType someMethod(ArgType1 arg1, boolean arg2) {
    return (ReturnType)execPython("some_method", arg1, DataConvertor.fromBool(arg2));
}
```

### Abstract methods

If there is some abstract method in the original extension point and you forget
to implement it neither in Python or Java, the `PythonWrapperError` runtime
error will occur (with an understandable message). This is different from the
Java programming, where the abstract method implementation is checked in the
compilation time.

### Python plugin project specifics

There are several differences in the Python plugin project from that written
purely in Java:

1. You have to mark the python-wrapper plugin dependency in your plugin's _pom.xml_:

    ```xml
    <dependencies>
        ...
        <dependency>
            <groupId>org.jenkins-ci.plugins</groupId>
            <artifactId>python-wrapper</artifactId>
            <version>1.0.3</version>
        </dependency>
        ...
    </dependencies>
    ```

2. You have to mark all .py files as resource files in your plugin's _pom.xml_:

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

3. Don't forget to determine all other resource directories which you are using
   (like _resources_ or _webapp_):

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

4. As previously mentioned, `@Extension` classes and their descriptors inherits
   from wrappers (`PW` classes) instead of original extension points and
   descriptors.

5. You have to place all Python scripts to the _src/main/python_ project directory.

### Return types of Python functions

There is a little difference between Python and Java basic types. Here is a
table with corresponding Python types for all Java types. Some pairs are more
obvious than others. Every Python function which overrides some extension point
method should return value in the corresponding type.

Java type | Python type
--------- | -----------
boolean   | bool
double    | float
float     | float
long      | long
int       | int
short     | int
byte      | int
char      | str (first char applies)
[]        | [array.array](http://www.jython.org/docs/library/array.html)
String    | str
Object    | object

### Global variable 'extension'

In every associated Python script there is automatically set a global variable
`extension` which points to the parent Java object. You can use it for calling
super methods or for setting/getting fields.

### Using init_plugin() function

You can define `init_plugin()` function in associated Python scripts. This
function is called _after_ the `extension` variable is set, but _before_ any
other function is called.

### Calling super methods

If you have to call super method of the original extension point or descriptor
for some reason, you can do it with special super\* method. E.g. if the original
extension point defines `someMethod()`, you can call it by
`extension.superSomeMethod()`. This was designed due to a workaround of ugly
Java reflection calls.

### Using Java object for a data serialization

If you need to save some data, use for it your Java class or most likely its
descriptor. Just create public fields in a Java class and access them by the
`extension` variable. Descriptors have `save()` and `load()` methods for the
data persistence control. Some extension classes like builders are saved
automatically. You will probably need another fields for data mining from the UI
(set by Stapler's @DataBoundConstructor). See [Existing Python
Plugins](https://github.com/jenkinsci/jenkins.py/wiki/Existing-Python-Plugins)
for an inspiration.


## FAQ

*  _Q: Can I combine Java and Python implementations?_

   A: Sure, you can implement some methods in Python and others in Java.
   Remember that the Java implementation overrides the Python implementation.

*  _Q: What modules can I import in associated Python scripts?_

   A: You can import any of [standard Jython
   modules](http://www.jython.org/docs/library/indexprogress.html). You can also
   import another Python scripts and libraries from your project's
   _src/main/python_ folder. You can even import all [Jenkins packages and
   classes](http://javadoc.jenkins-ci.org/) and work with them like they are
   normal Python objects.

*  _Q: How can I package, test and release my Python plugin?_

   A: Simply use Maven for that:

    ```bash
    # package
    mvn package
    # run and test
    mvn hpi:run
    # release
    mvn release:prepare release:perform
    ```


## Known issues

*  _Types of the terminal plugin are not visible in Python scripts._
   The reason is that Python scripts are executed by the parent plugin,
   python-wrapper, which can not see types of the terminal plugin. This will be
   probably resolved in the future. If you need to create instances of the
   extension class in the Python script, you can workaround that by calling
   standard `type()` function on an `extension` object and its descriptor in the
   `init_plugin()` function:

    ```python
    def init_plugin():
        global MyClass
        global MyClassDescriptor
        MyClass = type(extension)
        MyClassDescriptor = type(extension.getDescriptor())
    ```
