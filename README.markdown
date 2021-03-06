jasmine-maven-plugin
====================
**A Maven Plugin for processing JavaScript sources, specs, and executing Jasmine**

Put this in your POM...
-----------------------

    <project>
      <build>
        ...
        <plugins>
          ...
          <plugin>
            <groupId>searls</groupId>
            <artifactId>jasmine-maven-plugin</artifactId>
            <version>0.11.1-SNAPSHOT</version>
            <executions>
              <execution>
                <goals>
                  <goal>generateManualRunner</goal>
                  <goal>resources</goal>
                  <goal>testResources</goal>
                  <goal>test</goal>
                  <goal>preparePackage</goal>
                </goals>
              </execution>
            </executions>
          </plugin>
          ...
        </plugins>
      </build>
      ...
      <repositories>
        <repository>
          <id>searls-maven-thirdparty</id>
          <url>http://searls-maven-repository.googlecode.com/svn/trunk/thirdparty</url>
        </repository>
      </repositories>
      <pluginRepositories>
        <pluginRepository>
          <id>searls-maven-releases</id>
          <url>http://searls-maven-repository.googlecode.com/svn/trunk/releases</url>
        </pluginRepository>
        <pluginRepository>
          <id>searls-maven-snapshots</id>
          <url>http://searls-maven-repository.googlecode.com/svn/trunk/snapshots</url>
        </pluginRepository>
      </pluginRepositories>
      ...
    </project>
    
...and Smoke It
---------------

    mvn package

Executing any Maven lifecycle phase after prepare-package will show off everything this plugin has to give. However, the results will only be useful once you've added some JavaScript and specs

###src/main/javascript 
Store your project's JavaScript (i.e. `ninja.js`) and dependencies (i.e. `lib/prototype.js`) in `src/main/javsacript`. 

###src/test/javascript 
Store your Jasmine specs (i.e. `ninjaSpec.js`) in `src/test/javascript`. No need to create an HTML spec runner, one will be generated and executed for you by the **jasmine:test** goal!
    
Example test Output
-------------------
jasmine-maven-plugin halts on spec failures (unless haltOnFailure is set to false). An example of some failure output follows:

    -------------------------------------------------------
     J A S M I N E   T E S T S
    -------------------------------------------------------
    [INFO] describe FailSpec <<< FAILURE!
    [INFO]   describe NestedFail <<< FAILURE!
    [INFO]     it should fail deeply <<< FAILURE!
    [INFO]       * Expected true to be false.
    [INFO]   it should fail <<< FAILURE!
    [INFO]     * Expected true to be false.
    [INFO] describe HelloWorld <<< FAILURE!
    [INFO]   it should say hello
    [INFO]   it should say goodbye <<< FAILURE!
    [INFO]     * Expected 'Hello, World' to be 'Goodbye, World'.
    [INFO]   it should fail <<< FAILURE!
    [INFO]     * Expected 5 to be 6.
    [INFO] 
    Results:
    
    5 specs, 4 failures in 0.144s
    
    [INFO] ------------------------------------------------------------------------
    [ERROR] BUILD FAILURE
    [INFO] ------------------------------------------------------------------------
    [INFO] There were test failures.
    [INFO] ------------------------------------------------------------------------

Usage Notes
-----------
### Project layout
The jasmine-maven-plugin sports a default project directory layout that should be convenient for most green field projects. The included example project (which is in [src/test/resources/jasmine-webapp-example](http://github.com/searls/jasmine-maven-plugin/tree/master/src/test/resources/jasmine-webapp-example/) demonstrates this convention and looks something like this:

    |-- pom.xml
    |-- src
    |   |-- main
    |   |   |-- javascript
    |   |   |   |-- HelloWorld.js
    |   |   |   `-- vendor
    |   |   |       `-- jquery-1.4.2.min.js
    |   |   |-- resources
    |   |   `-- webapp
    |   |       |-- META-INF
    |   |       |   `-- MANIFEST.MF
    |   |       |-- WEB-INF
    |   |       |   |-- lib
    |   |       |   `-- web.xml
    |   |       `-- index.jsp
    |   `-- test
    |       `-- javascript
    |           |-- FailSpec.js
    |           `-- HelloWorldSpec.js
    `-- target
        |-- classes
        |-- jasmine
        |   |-- SpecRunner.html
        |   |-- spec
        |   |   |-- FailSpec.js
        |   |   `-- HelloWorldSpec.js
        |   `-- src
        |       |-- HelloWorld.js
        |       `-- vendor
        |           `-- jquery-1.4.2.min.js
        |-- jasmine-webapp-example
        |   |-- META-INF
        |   |   `-- MANIFEST.MF
        |   |-- WEB-INF
        |   |   |-- classes
        |   |   `-- web.xml
        |   |-- index.jsp
        |   `-- js
        |       |-- HelloWorld.js
        |       `-- vendor
        |           `-- jquery-1.4.2.min.js
        `-- jasmine-webapp-example.war

As seen above, production JavaScript is placed in `src/main/javascript`, while test specs are each in `src/test/javascript`. The plugin does support nested directories and will maintain your directory structure as it processes the source directories.

### Goals

####jasmine:resources
This goal binds to the process-resources phase and copies the `src/main/javascript` directory into `target/jasmine/src`. It can be changed by configuring a parameter named `jsSrcDir` in the plugin execution section of the POM.

####jasmine:testResources
This goal binds to the process-test-resources phase and copies the `src/test/javascript` directory into `target/jasmine/spec`. It can be changed by configuring a parameter named `jsTestSrcDir` in the plugin execution section of the POM.

####jasmine:test
This goal binds to the test phase and generates a Jasmine runner file in `target/jasmine/SpecRunner.html` based on the sources processed by the previous two goals and Jasmine's own dependencies. It will respect the `skipTests` property, and will not halt processing if `haltOnFailure` is set to false.

####jasmine:preparePackage 
This goal binds to the prepare-package phase and copies the production JavaScript sources from `target/jasmine/src` to `/js` within the package directory (e.g. `target/your-webapp/js`). The sub-path can be cleared or changed by setting the `packageJavaScriptPath` property

####jasmine:generateManualRunner
This goal will generate an extra spec runner HTML alongside your JavaScript source and spec directories. This way, you can easily run your specs in the browser as you develop them, while still leaning on the plugin to keep the HTML up-to-date for you. Note that this HTML file is separate from the one generated during **jasmine:test**, as it points to the source directories directly.


You can run the plugin with all four goals or only the first three. For instance, if you run your application in Eclipse WTP and you want to keep your production JavaScript in `src/main/webapp` to facilitate easier iterative development, you could skip the preparePackage goal and configure the `jsSrcDir` property to point at `src/main/webapp/[your-js-directory]` instead. Example POM follows:

    <execution>
      <goals>
        <goal>resources</goal>
        <goal>testResources</goal>
        <goal>test</goal>
      </goals>
      <configuration>
        <srcDir>${project.basedir}/src/main/webapp/js</srcDir>
      </configuration>
    </execution>

### Configuration
Among configurations listed elsewhere, you can configure jasmine-maven-plugin to load a specified list of JavaScript sources (relative to ${jsSrcDir}, which defaults to `src/main/javascript`) before the other ones. So, for instance, if you need to load jQuery and then jQuery plugins before your production sources get included in the runner, you can specify those sources you want to preload like so:

    <configuration>
      ...
      <preloadSources>
        <source>vendor/jquery.js</source>
        <source>vendor/jquery-ui.js</source>
      </preloadSources>				
    </configuration>
    
In the example above, `vendor/jquery.js` and `vendor/jquery-ui.js` are still added to the generated SpecRunner.html once, just before all other sources in the project.

## Maintainers
* [Justin Searls](http://twitter.com/Searls), Pillar Technology

## Contributions
Pull requests are, of course, very welcome! A few todos as of 6/27, if anyone is interested in tackling them:

* Parse & format ignored tests (currently only passing & failing tests are parsed)
* A report mojo that generates JUnit-style XML, so results can easily be rolled up by CI servers.
* A facility that automatically executes the other goals if only `test` or `preparePackage` is configured to run.

## Acknowledgments
* Thanks to Pivotal Labs for authoring and publishing [Jasmine](http://github.com/pivotal/jasmine)
* Thanks to christian.nelson and sivoh1, owners of the [javascript-test-maven-plugin](http://code.google.com/p/javascript-test-maven-plugin/) project, which provided a similar implementation from which to glean several valuable lessons.