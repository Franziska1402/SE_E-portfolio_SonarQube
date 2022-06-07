# SE_E-portfolio_SonarQube

![sonarQube logo](https://www.sonarqube.org/logos/index/logo-hero@2x.png)

[![Quality Gate Status](https://sonarcloud.io/api/project_badges/measure?project=Franziska1402_SE_E-portfolio_SonarQube&metric=alert_status)](https://sonarcloud.io/summary/new_code?id=Franziska1402_SE_E-portfolio_SonarQube)

## Table of content
1. [Introduction](#introduction)
2. [SonarQube](#sonarqube)
   1. [Features](#features)
   2. [SonarCloud](#sonarcloud)
   3. [SonarLint](#sonarlint)
3. [Getting Started](#getting-started)
4. [build.yml](#buildyml)
5. [Testcoverage](#testcoverage)
6. [More](#more)

## Introduction
To avoid the following situation, static code analysis tools can be used to write cleaner and safer code.

![WTFs/minute](https://miro.medium.com/max/1000/0*HaqsruEqnR5pxQD4)

## SonarQube
SonarQube is one of the many possible analysis tools for static code analysis. It's free & open source within the Community Edition and provides support for 29 programming languages.
For some projects you can use automatic analysis, but SonarQube can also be integrated in your CI/CD tool. The summary of the results is available via a web interface.

### Features
- Open Source
- supports 29 programming languages
- CI/CD integration
- Feedback during Code Review
- Creates Issues (Bugs, Vulnerabilities, Code-Smells) with effort and category
- Security hotspots
- Project releasability with quality gates
- Calculates metrics
- Visualization in web interface
- Detailed documentation

### SonarCloud
SonarCloud is the cloud version of SonarQube with little less functionality. It's hosted by SonarSource in AWS and is the easiest path to start scanning your code in minutes. Therefore, this e-portfolio works with SonarCloud.
If you're fine with self-hosting and maintenance or see value in the management capabilities, then you can also use SonarQube instead.

### SonarLint
For the sake of completeness, SonarLint is also mentioned here. This is a plugin for the own IDE, where a URL to the SonarQube or SonarCloud server can be stored, so that the same rules are used.
The developer already gets feedback in his IDE, but no security hotspots or coverage issues are checked.

## Getting Started
To use SonarCloud as analysis tool for your public github project, you must have installed the github application for SonarCloud.
Then you have to make sure, that the application has access to the project.

In [sonarcloud](https://sonarcloud.io/) you add a new project, if you can't see it, check the visibility and the access of the application.
The tutorials for the different analysis methods can be found under Administration/ Analysis Method.
Below you find the steps for GitHub Action with Gradle project.

## Build.yml
After the creation of a GitHub Secrect  you can update your build.gradle file with the following code:

```gradle
plugins {
  id "org.sonarqube" version "3.3"
}

sonarqube {
  properties {
    property "sonar.projectKey", "AddYourProjectKey"
    property "sonar.organization", "AddYourOrganization"
    property "sonar.host.url", "https://sonarcloud.io"
  }
}
```

Additionaly you have to create or update your .github/workflows/build.yml file. The code herefore is:

```yaml
name: Build
on:
  push:
    branches:
      - master
  pull_request:
    types: [opened, synchronize, reopened]
jobs:
  build:
    name: Build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
        with:
          fetch-depth: 0  # Shallow clones should be disabled for a better relevancy of analysis
      - name: Set up JDK 11
        uses: actions/setup-java@v1
        with:
          java-version: 11
      - name: Cache SonarCloud packages
        uses: actions/cache@v1
        with:
          path: ~/.sonar/cache
          key: ${{ runner.os }}-sonar
          restore-keys: ${{ runner.os }}-sonar
      - name: Cache Gradle packages
        uses: actions/cache@v1
        with:
          path: ~/.gradle/caches
          key: ${{ runner.os }}-gradle-${{ hashFiles('**/*.gradle') }}
          restore-keys: ${{ runner.os }}-gradle
      - name: Build and analyze
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}  # Needed to get PR information, if any
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
        run: ./gradlew build sonarqube --info
```

Make sure, your master-branch has the correct name.

If you face a problem, that gradlew permission is denied, the following command can help you:

```command
git update-index --chmod=+x gradlew
```

## Testcoverage
For the test coverage the plugin jacoco can be used.
You need to add the following in your build.gradle file:

```gradle
plugins {
   id "jacoco"
}

jacocoTestReport {
   reports {
   xml.enabled true
   }
}

sonarqube {
 properties {
   property "sonar.login", findProperty("APIKey") as String
 }
}
```
Generate a new token under your account/ security/ token, add it in new gradle.properties file and exclude it in your .gitignore:

```properties
APIKey = YourToken
```

Tell SonarCloud under administration/ General settings/ JaCoCo, where it can find the test reports: 

```
sonar.coverage.jacoco.xmlReportPaths
```

Now you only need to generate your test report and the coverage should be visible in SonarCloud:

```
gradle test jacocoTestReport sonarqube
```

## More
For more information have a look on the official documentation:

[SonarQube](https://www.sonarqube.org/)

[SonarCloud](https://docs.sonarcloud.io/)
