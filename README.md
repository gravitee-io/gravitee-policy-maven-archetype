# gravitee-policy-maven-archetype
Maven Archetype to create a Gravitee Policy

[![Build Status](http://build.gravitee.io/jenkins/buildStatus/icon?job=gravitee-policy-maven-archetype)](http://build.gravitee.io/jenkins/job/gravitee-policy-maven-archetype/)

## Get started

Archetype is already registered into the [OSS repositories](http://central.sonatype.org/pages/ossrh-guide.html). Don't forget to activate them in your Maven settings.

## Example

```bash
mvn archetype:generate\
 -DarchetypeGroupId=io.gravitee.maven.archetypes\
 -DarchetypeArtifactId=gravitee-policy-maven-archetype\
 -DarchetypeVersion=1.0.0-SNAPSHOT\
 -DartifactId=my-policy\
 -DgroupId=my.project.gravitee.policies\
 -Dversion=1.0.0-SNAPSHOT\
 -DpolicyName=MyPolicyName
```

## Tip

Choose a short but clearly name for your Policy, **without precise the Policy suffix**. The gravitee-policy-maven-archetype will add it automatically.

For example, do not fill the ``policyName`` of your Policy like this:

```
-DpolicyName=AmazingStuffPolicy
```

but like this:

```
-DpolicyName=AmazingStuff
```