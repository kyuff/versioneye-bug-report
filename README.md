# VersionEye Bug Report

This project is build of two modules: The parent project and the child project like this:

parent: versioneye-bug-report

child: app-war

The parent have defined two dependencies in the Dependency Management section:

1. javaee-api
1. validation-api

app-war uses one of these dependencies: javaee-api

When running a standard report with VersionEye it correctly lists the dependencies mentioned in app-war,
but it creates false positives and it's missing transitive dependencies.

The command executed is simply:

    cd app-war
    mvn versioneye:update

A summary can be seen here:

Jar in WAR            | In Report                  | Comment
----------------------|----------------------------|-----------------------------------------------|
activation:1.1        |                            | Missing! Transitive dependency from javaee-api
c3p0:0.9.1.1          |                            | Missing! Transitive dependency from quartz
javaee-api:7.0        | javaee-api:7.0             | Twice in the report, due to also being in DM?
javax.mail:1.5.0      |                            | Missing! Transitive dependency from javaee-api
javax.ws.rs-api:2.0.1 | javax.ws.rs-api:2.0.1      | ✓
quartz:2.2.3          | quartz:2.2.3               | ✓
slf4j-api:1.7.7       |                            | Missing! Transitive dependency from quartz
                      | validation-api:1.1.0.Final | Should not be in the report, taken from DM.

Now, it appears there is an issue with things defined in dependency management.
The false positives that is created by the unused dependencies in dependency management.

So we could try to run this command:

    mvn versioneye:update -Dversioneye.ignoreDependencyManagement=true

Which in return produces this outcome:


Jar in WAR            | In Report                  | Comment
----------------------|----------------------------|-----------------------------------------------|
activation:1.1        |                            | Missing! Transitive dependency from javaee-api
c3p0:0.9.1.1          |                            | Missing! Transitive dependency from quartz
javaee-api:7.0        | javaee-api:7.0             | ✓
javax.mail:1.5.0      |                            | Missing! Transitive dependency from javaee-api
javax.ws.rs-api:2.0.1 | javax.ws.rs-api:2.0.1      | ✓
quartz:2.2.3          | quartz:2.2.3               | ✓
slf4j-api:1.7.7       |                            | Missing! Transitive dependency from quartz
                      |                            | ✓ validation-api correctly left out

So by setting the ignoreDependencyManagement, it can solve two problems:

1. Double listing when artifacts are directly dependent and listed in dependency management.
1. False positives were artifacts are listed in dependency management but not in the project.

It still leaves out a problem with missing transitive dependencies.