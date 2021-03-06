[![Maven Central Latest](https://img.shields.io/maven-central/v/com.cosium.code/maven-git-code-format.svg)](https://search.maven.org/#search%7Cgav%7C1%7Cg%3A%22com.cosium.code%22%20AND%20a%3A%22maven-git-code-format%22)
[![Build Status](https://travis-ci.org/Cosium/maven-git-code-format.svg?branch=master)](https://travis-ci.org/Cosium/maven-git-code-format)

# Maven Git Code Format

A maven plugin that automatically deploys [google-java-format](https://github.com/google/google-java-format) code formatter as a `pre-commit` git hook.  
On commit, the hook will automatically format staged java files.

### Automatic code format and validation activation

Add this to your maven project **root** pom.xml :

```xml
<build>
  <plugins>
    <plugin>
      <groupId>com.cosium.code</groupId>
      <artifactId>maven-git-code-format</artifactId>
      <version>${maven-git-code-format.version}</version>
      <executions>
        <!-- On commit, format the modified java files -->
        <execution>
          <id>install-formatter-hook</id>
          <goals>
            <goal>install-hooks</goal>
          </goals>
        </execution>
        <!-- On Maven verify phase, fail if any file 
        (including unmodified) is badly formatted -->
        <execution>
          <id>validate-code-format</id>
          <goals>
            <goal>validate-code-format</goal>
          </goals>
        </execution>
      </executions>
    </plugin>
  </plugins>
</build>
```

### Manual code formatting

```console
mvn git-code-format:format-code -DglobPattern=**/*
```

### Manual code format validation

```console
mvn git-code-format:validate-code-format -DglobPattern=**/*
```

### Google Java Format options

The plugin allows you to tweak Google Java Format options :

```xml
<build>
  <plugins>
    <plugin>
      <groupId>com.cosium.code</groupId>
      <artifactId>maven-git-code-format</artifactId>
      <version>${maven-git-code-format.version}</version>
      <executions>
        <!-- ... -->
      </executions>
      <configuration>
        <googleJavaFormatOptions>
          <aosp>false</aosp>
          <fixImportsOnly>false</fixImportsOnly>
          <skipSortingImports>false</skipSortingImports>
          <skipRemovingUnusedImports>false</skipRemovingUnusedImports>
        </googleJavaFormatOptions>
      </configuration>
    </plugin>
  </plugins>
</build>
```

Documentation from the google-java-format CLI tool :

```
--aosp, -aosp, -a
  Use AOSP style instead of Google Style (4-space indentation).
--fix-imports-only
  Fix import order and remove any unused imports, but do no other formatting.
--skip-sorting-imports
  Do not fix the import order. Unused imports will still be removed.
--skip-removing-unused-imports
  Do not remove unused imports. Imports will still be sorted.
```

### Frequently asked questions

#### If I have a multi-module project, do I need to install anything in the sub-projects?
You only need to put the plugin in your *root* project pom.xml. By default all submodules will be handled.

#### Do I need to run mvn initialize or is that a stage that happens automatically when I run mvn compile or mvn test?
`initialize` is the first phase of the Maven lifecycle. Any goal that you perform (e.g. `compile` or `test`) will automatically trigger `initialize` and thus trigger the git pre-commit hook installation.

#### I'm not noticing anything happening.
If after setting up the plugin in your pom, you just executed a maven goal, the only expected output is a pre-commit hook installed in your `.git/hooks` directory. To trigger the automatic formatting, you have to perform a commit of a modified java file.  
You can also manually [format](#manual-code-formatting) or [validate](#manual-code-format-validation) any file.

### How the hook works

On the `initialize` maven phase, `git-code-format:install-hooks` installs a git `pre-commit` hook that looks like this :
```bash
#!/bin/bash
/usr/share/apache-maven-3.5.0/bin/mvn git-code-format:on-pre-commit
```

On `pre-commit` git phase, the hook triggers the `git-code-format:on-pre-commit` which formats the code of the modified java files using `google-java-format`. 
