# gravitee-policy-maven-archetype
Maven Archetype to create a Gravitee Policy

[![Build Status](http://build.gravitee.io/jenkins/buildStatus/icon?job=gravitee-policy-maven-archetype)](http://build.gravitee.io/jenkins/job/gravitee-policy-maven-archetype/)

## Get started

A Policy is a step in the Request/Response Gateway proxy chain. A Policy acts as a *proxy controller* by guaranteeing if a given business rule is fulfilled during the Request/Response processing.

Archetype is already registered into the [OSS repositories](http://central.sonatype.org/pages/ossrh-guide.html). Don't forget to activate them in your Maven settings.

## Policy generation

Suppose you want to create a Policy that control if request contains the ``X-Foo`` header. Let's name it the *Foo header check policy*. Then you could generate your Policy like this:

```bash
mvn archetype:generate\
 -DarchetypeGroupId=io.gravitee.maven.archetypes\
 -DarchetypeArtifactId=gravitee-policy-maven-archetype\
 -DarchetypeVersion=1.0.0-SNAPSHOT\
 -DartifactId=foo-header-check-policy\
 -DgroupId=my.gravitee.extension.policy\
 -Dversion=1.0.0-SNAPSHOT\
 -DpolicyName=FooHeaderCheck
```

Once executed and parameters confirmed, the above command will create the ``foo-header-check-policy`` directory containing the following structure:

```
.
├── pom.xml
├── README.md
└── src
    ├── assembly
    │   └── policy-assembly.xml
    ├── main
    │   ├── java
    │   │   └── my
    │   │       └── gravitee
    │   │           └── extension
    │   │               └── policy
    │   │                   ├── FooHeaderCheckPolicyConfiguration.java
    │   │                   └── FooHeaderCheckPolicy.java
    │   └── resources
    │       └── plugin.properties
    └── test
        └── java
            └── my
                └── gravitee
                    └── extension
                        └── policy
                            └── FooHeaderCheckPolicyTest.java
```

Hereafter a description about the different generated files:

| File    | Description | 
|---------|-------------|
| [`pom.xml`](#pom) | The main Maven POM file        |
| [`README.md`](#readme) | The main entry point for documentation of the Policy      |
| [`policy-assembly.xml`](#assembly) | The common Maven assembly descriptor for any Policies |
| [`FooHeaderCheckPolicyConfiguration.java`](#configuration) | The Policy configuration class |
| [`FooHeaderCheckPolicy.java`](#policy) | The Policy class, from which the business behavior is implemented |
| [`plugin.properties`](#descriptor) | The Policy descriptor file |
| [`FooHeaderCheckPolicyTest.java`](#test) | The Policy unit test Java class |

### <a name="pom"></a> pom.xml

Each Policy (and more generally any Gravitee projects) is [Maven](https://maven.apache.org/) managed. Thus, a Policy project is described by using the Maven [Project Object Model](https://maven.apache.org/pom.html) file.

### <a name="readme"></a> README.md

Each Policy should have a dedicated `README.md` file to document it. The `README.md` file should contain everything related to the use of your Policy: *What is its functionality? How can use it? How can configure it?*  

### <a name="assembly"></a> policy-assembly.xml

A Policy is just a kind of Gravitee Plugin.

It  can be plugged to the [Gravitee Gateway](https://github.com/gravitee-io/gravitee-gateway) by using the distribution file built from the `policy-assembly.xml` file.

Based on our *FooHeaderCheck* Policy, the distribution file structure is the following:

```
.
├── foo-header-check-policy-1.0.0-SNAPSHOT.jar
├── lib
└── schemas
    └── urn:jsonschema:my:gravitee:extension:policy:FooHeaderCheckPolicyConfiguration.json
```

Hereafter a description about the different generated files:

| File    | Description | 
|---------|-------------|
| `foo-header-check-policy-1.0.0-SNAPSHOT.jar` | The main Policy jar file         |
| `lib/` | Where the external dependencies are stored (from the [Maven POM file dependencies](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html)          |
| `schemas/` | Where the JSON configuration schemas are stored          |

#### <a name="schema"></a> JSON Configuration schemas

Policy configuration is described inside one or several [Java Bean](http://docs.oracle.com/javase/tutorial/javabeans/) classes (see the [FooHeaderCheckPolicyConfiguration.java](#configuration) example).

During the packaging phase, each Policy configuration classes are processed to generate one or several [JSON schema(s)](http://json-schema.org/) that will be read by the Gateway when the Policy will be plugged in.

JSON schema generation is done thanks to the Gravitee's [json-schema-generator-maven-plugin](https://github.com/gravitee-io/json-schema-generator-maven-plugin) Maven plugin. 

### <a name="configuration"></a> FooHeaderCheckPolicyConfiguration.java

The Policy configuration class.

Policy configuration is described into one or several [Java Bean](http://docs.oracle.com/javase/tutorial/javabeans/) class(es) where each attribute is a configuration parameter.

During the package phase, Policy configuration is compiled into [JSON Configuration schemas](#schemas). These schemas are used to parse [API definitions](https://github.com/gravitee-io/gravitee-gateway).

Policy configuration is finally injected to the Policy class instance at runtime and then can be used during implementation.

### <a name="policy"></a> FooHeaderCheckPolicy.java

The main Policy class. Contains business code that implements the Policy.

A Policy can be applied on several parts of the proxy chain:

 - The Request phase
 - The Response phase
 - Both of them

#### Apply Policy on the Request phase

A Policy can be applied to the proxy Request phase by just implementing a method dealing with the ``io.gravitee.gateway.api.policy.annotations.OnRequest`` annotation. For instance:

```java
@OnRequest
public void onRequest(Request request, Response response, PolicyChain policyChain) {
	// Add a dummy header
    request.headers().set("X-DummyHeader", configuration.getDummyHeaderValue());

	// Finally continue chaining
	policyChain.doNext(request, response);
}
```

> The `PolicyChain` **must always be called to end a *on Request* processing**. Be ware to make a call to the `PolicyChain#doNext()` or  `PolicyChain#failWith()`   to correctly end the *on Request* processing.

#### Apply Policy on the Response phase

A Policy can be applied to the proxy Response phase by just implementing a method dealing with the ``io.gravitee.gateway.api.policy.annotations.OnResponse`` annotation. For instance:

```java
@OnResponse
public void onResponse(Request request, Response response, PolicyChain policyChain) {
    if (isASuccessfulResponse(response)) {
        policyChain.doNext(request, response);
    } else {
        policyChain.failWith(new PolicyResult() {
            @Override
            public boolean isFailure() {
                return true;
            }

            @Override
            public int httpStatusCode() {
                return HttpStatusCode.INTERNAL_SERVER_ERROR_500;
            }

            @Override
            public String message() {
                return "Not a successful response :-(";
            }
        });
    }
}

private static boolean isASuccessfulResponse(Response response) {
    switch (response.status() / 100) {
        case 1:
        case 2:
        case 3:
            return true;
        default:
            return false;
    }
}
```

> The `PolicyChain` **must always be called to end a *on Response* processing**. Be ware to make a call to the `PolicyChain#doNext()` or  `PolicyChain#failWith()`   to correctly end the *on Response* processing.

#### Apply Policy on both of Request and Response phases

A Policy is not restricted to only one Gateway proxy phase. It can be applied on both of the Request and Response phases by just using the both annotations in the same class.

#### Injected parameters

The annotated methods can declare several parameters (but not necessary all of them) which will be automatically injected by the Gateway at runtime.
Available injected parameters are:

| Parameter class   | Mandatory | Description | 
|---------|-------------|-------------|
| `io.gravitee.gateway.api.Request` | No | Wrapper to the Request object containing all information about the processed request (URI, parameters, headers, input stream, ...)        |
| `io.gravitee.gateway.api.Response` | No | Wrapper to the Response object containing all information about the processed response (status, headers, output stream, ...)        |
| `io.gravitee.gateway.api.policy.PolicyChain` | Yes | The current Policy chain that gives control to the Policy to continue (`doNext`) or reject (`failWith`) the current chain.         |
| `io.gravitee.gateway.api.policy.PolicyContext` | No | The Policy context that can be used to get contextualized objects (API store, ...).         |

### <a name="descriptor"></a> plugin.properties

As said, a Policy is a kind of Gravitee Plugin.

Each Plugin is described by the *plugin.properties* descriptor which declare the following parameters:

| Parameter   | Description | Default value |
|---------|-------------|-------------|
| `id` | The Policy identifier     | Policy artifact id |
| `name` | The Policy name     | N/A (mandatory parameter) |
| `version` | The Policy version     | N/A (mandatory parameter) |
| `description` | The Policy description     | "Description of the *Policy name* Gravitee Policy" |
| `class` | The main Policy class     | Path to the generated class file |
| `type` | The type of Gravitee Plugin     | `policy` |

> A Policy is enabled when declared into the API definition. To do so, the Policy identifier is used to, as its name indicate, identify the Policy. Thus, **be ware to correctly choose the Policy identifier** from the beginning. It could be hard to rename it later if there are many of API definitions linked to it.

### <a name="test"></a> FooHeaderCheckPolicyTest.java
 
The [JUnit](http://junit.org/) unit test class for this Policy.

## Tip

Choose a short but clearly name for your Policy, **without precise the Policy suffix**. The `gravitee-policy-maven-archetype` will add it automatically.

For example, **do not** fill the ``policyName`` of your Policy like this:

```
-DpolicyName=AmazingStuffPolicy
```

but like this:

```
-DpolicyName=AmazingStuff
```