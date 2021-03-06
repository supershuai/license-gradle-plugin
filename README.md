# License Gradle Plugin

[![Build Status](https://travis-ci.org/hierynomus/license-gradle-plugin.svg?branch=master)](https://travis-ci.org/hierynomus/license-gradle-plugin)

This plugin will scan and adapt your source files to include a provided header, e.g. a LICENSE file.  By default it will scan every source set and report warnings. It will also create format tasks, which will properly format and apply the specified header. A bulk of the logic comes from the maven-license-plugin.

This plugin will also report on the licenses of your dependencies.

## Usage
From v0.11.0 onwards the `license-gradle-plugin` will be published to http://bintray.com and will be available through the [http://plugins.gradle.org/](Gradle plugin exchange). This means that there are a few different usage scenarios listed below.


### Gradle 2.1
In your _build.gradle_ file add:
```
plugins {
    id "com.github.hierynomus.license" version "0.11.0"
}
```

### Gradle 1.x/2.0, latest license-gradle-plugin
In your _build.gradle_ file add:
```
    buildscript {
        repositories {
            jcenter()
        }

        dependencies {
            classpath 'nl.javadude.gradle.plugins:license-gradle-plugin:0.11.0'
        }
    }
```

### Gradle 1.x/2.0, gradle-license-plugin 0.10.0 (and earlier)
In your _build.gradle_ file add:

```
    buildscript {
        repositories {
            mavenCentral()
        }

        dependencies {
            classpath 'nl.javadude.gradle.plugins:license-gradle-plugin:0.10.0'
        }
    }

    apply plugin: 'license'
```

This will add two types of tasks for each source set (created by BasePlugin) in your project: one for checking for consistency and one to apply the header, e.g.

- licenseMain        : checks for header consistency in the main source set
- licenseFormatMain  : applies the license found in the header file in files missing the header
- licenseTest        : checks for header consistency in the test source set

The check tasks are added to the standard Gradle _check_ task.

This will also add a task to manage the downloading and reporting of licenses of your dependencies.

- downloadLicenses   : generates reports on your runtime dependencies

## License Task
The license task has a properties, most can be set in the extension:

- header -- Specify location of header to use in comparisons, default to project.file('LICENSE')
- ignoreFailures -- Prevent tasks from stopping the build, defaults to false
- dryRun -- Show what would happen if the task was run, defaults to false but also inherits from --dryRun
- skipExistingHeaders -- Skip over files that have some header already, which might not be the one specified in the header parameter, defaults to false
- useDefaultMappings -- Use a long list of standard mapping, defaults to true. See http://code.google.com/p/maven-license-plugin/wiki/Configuration#Default_mappings for the complete list
- strictCheck -- Be extra strict in the formatting of existing headers, defaults to false
- sourceSets -- List of sourceSets to create tasks for, will default to all sourceSets created by Java Plugin
- mapping(String ext, String style) -- Adds a mapping between a file extension and a style type
- mapping(Map<String,String> mappings) -- Adds mappings between file extensions and style types
- mapping(Closure) -- Adds mappings between file extensions and a style types, see example below

### License Extension
A license extension is added to the project, which can be used to configure all license tasks. E.g.
 
```
license {
    header rootProject.file('codequality/HEADER')
    strictCheck true
}
```

Here is a general overview of the options:

- header -- Specify location of header to use in comparisons, default to project.file('LICENSE')
- ignoreFailures -- Prevent tasks from stopping the build, defaults to false
- dryRun -- Show what would happen if the task was run, defaults to false but also inherits from --dryRun
- skipExistingHeaders -- Skip over files that have some header already, which might not be the one specified in the header parameter, defaults to false
- useDefaultMappings -- Use a long list of standard mapping, defaults to true. See http://code.google.com/p/maven-license-plugin/wiki/Configuration#Default_mappings for the complete list
- strictCheck -- Be extra strict in the formatting of existing headers, defaults to false
- sourceSets -- List of sourceSets to create tasks for, will default to all sourceSets created by Java Plugin
- mapping(String ext, String style) -- Adds a mapping between a file extension and a style type
- mapping(Map<String,String> mappings) -- Adds mappings between file extensions and style types
- mapping(Closure) -- Adds mappings between file extensions and a style types, see example below
- exclude(String pattern) -- Add an ANT style pattern to exclude files from license absence reporting and license application
- excludes(Collection<String> patterns) -- Add ANT style patterns to exclude files from license absence reporting and license application
- include(String pattern) -- Add an ANT style pattern to include files into license absence reporting and license application
- includes(Collection<String> patterns) -- Add ANT style patterns to include files into license absence reporting and license application

### File Types
Supported by default: java, groovy, js, css, xml, dtd, xsd, html, htm, xsl, fml, apt, properties, sh, txt, bat, cmd, sql, jsp, ftl, xhtml, vm, jspx, gsp, json. Complete list can be found in <a href="http://code.google.com/p/maven-license-plugin/wiki/SupportedFormats">SupportedFormats</a> page of the parent project.

### Recognizing other file types.
An extensive list of formats and mappings are available by default, see the SupportedFormats link above. Occasionally a project might need to add a mapping to a unknown file type to an existing comment style.

```
license {
    mapping {
        javascript='JAVADOC_STYLE'
    }
}
// or
license.mapping 'javascript' 'JAVADOC_STYLE'
// or
license.mapping('javascript', 'JAVADOC_STYLE')
// or directly on the task
licenseMain.mapping 'javascript' 'JAVADOC_STYLE'
```

Defining new comment types is not currently supported, but file a bug and it can be added.

### Variable substitution
Variables in the format ${} format will be substituted, as long as they're values are provided in the extension or the task.

    Copyright (C) ${year} ${name} <${email}>

Will be completed with this extension block, the key is adding them via extra properties:

```
license {
    ext.year = Calendar.getInstance().get(Calendar.YEAR)
    ext.name = 'Company'
    ext.email = 'support@company.com'
}
// or
licenseMain.ext.year = 2012
```

### Include/Exclude files from license absence reporting and license application
By default all files in the sourceSets configured are required to carry a license. Just like with Gradle `SourceSet` you can use include/exclude patterns to control this behaviour.

The semantics are:
- no `includes` or `excludes`: All files in the sourceSets will be included
- `excludes` provided: All files except those matching the exclude patterns are included
- `includes` provided: Only the files matching the include patterns are included
- both `includes` and `excludes` provided: All files matching the include patterns, except those matching the exclude patterns are included.

For instance:
```
license {
    exclude "**/*.properties"
    excludes(["**/*.txt", "**/*.conf"])
}
```
This will exclude all `*.properties`, `*.txt` and `*.conf` files.

```
license {
    include "**/*.groovy"
    includes(["**/*.java", "**/*.properties"])
}
```
This will include only all `*.groovy`, `*.java` and `*.properties` files.

```
license {
    include "**/*.java"
    exclude "**/*Test.java"
}
```
This will include all `*.java` files, except the `*Test.java` files.


## License Reporting
The downloadLicense task has a set of properties, most can be set in the extension:

- includeProjectDependencies -- true if you want to include the transitive dependencies of your project dependencies
- licenses -- a pre-defined mapping of a dependency to a license; useful if the external repositories do not have license information available
- aliases -- a mapping between licenses; useful to consolidate the various POM definitions of different spelled/named licenses
- excludeDependencies -- a List of dependencies that are to be excluded from reporting
- dependencyConfiguration -- Gradle dependency configuration to report on (defaults to "runtime").

A 'license()' method is made available by the License Extension that takes two Strings, the first is the license name, the second is the URL to the license.
```
downloadLicenses {
    ext.apacheTwo = license('Apache License, Version 2.0', 'http://opensource.org/licenses/Apache-2.0')
    ext.bsd = license('BSD License', 'http://www.opensource.org/licenses/bsd-license.php')
    
    includeProjectDependencies = true
    licenses = [
        (group('com.myproject.foo')) : license('My Company License'),
        'org.apache.james:apache-mime4j:0.6' : apacheTwo,
        'org.some-bsd:project:1.0' : bsd
    ]

    aliases = [
        (apacheTwo) : ['The Apache Software License, Version 2.0', 'Apache 2', 'Apache License Version 2.0', 'Apache License, Version 2.0', 'Apache License 2.0', license('Apache License', 'http://www.apache.org/licenses/LICENSE-2.0')],
        (bsd) : ['BSD', license('New BSD License', 'http://www.opensource.org/licenses/bsd-license.php')]
    ]

    excludeDependencies = [
        'com.some-other-project.bar:foobar:1.0'
    ]
    
    dependencyConfiguration = 'compile'
}
```

# Changelog

## v0.11.0
- Added support for uploading to bintray (Fixes [#46](https://github.com/hierynomus/license-gradle-plugin/issues/46) and [#47](https://github.com/hierynomus/license-gradle-plugin/issues/47))
- Upgraded to Gradle 2.0

## v0.10.0
- Fixed build to enforce Java6 only for local builds, not on BuildHive
- Added `exclude` / `excludes` to extension (Fixes [#39](https://github.com/hierynomus/license-gradle-plugin/issues/39))
- Added `include` / `includes` to extension (Fixes [#45](https://github.com/hierynomus/license-gradle-plugin/issues/45))

## v0.9.0
- Fixed build to force Java6 (Fixes [#35](https://github.com/hierynomus/license-gradle-plugin/issues/35))
- Added example test for [#38](https://github.com/hierynomus/license-gradle-plugin/issues/38)

## v0.8.0
- Merged pull-requests [#31](https://github.com/hierynomus/license-gradle-plugin/pull/31), [#33](https://github.com/hierynomus/license-gradle-plugin/pull/33), [#42](https://github.com/hierynomus/license-gradle-plugin/pull/42)
- 
