# gravitee-policy-maven-archetype
Maven Archetype to create a Gravitee Policy

[![Build Status](http://build.gravitee.io/jenkins/buildStatus/icon?job=gravitee-policy-maven-archetype)](http://build.gravitee.io/jenkins/job/gravitee-policy-maven-archetype/)

## Example

```
mvn archetype:generate\
 -DarchetypeGroupId=io.gravitee.maven.archetypes\
 -DarchetypeArtifactId=gravitee-policy-maven-archetype\
 -DarchetypeVersion=1.0.0-SNAPSHOT\
 -DartifactId=my-policy\
 -DgroupId=my.project.gravitee.policies\
 -Dversion=1.0.0-SNAPSHOT\
 -DpolicyName=MyPolicy
```

More documentation will come...