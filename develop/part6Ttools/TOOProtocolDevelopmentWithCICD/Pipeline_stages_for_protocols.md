---
uid: Pipeline_stages_for_protocols
---

# Pipeline stages for protocols

Currently, the pipeline for protocol development consists of the following steps:

- Loading Jenkinsfile

    See [Loading Jenkinsfile](#loading-jenkinsfile).

- Declarative checkout from SCM

    See [Declarative checkout from SCM](#declarative-checkout-from-scm).

- Detect solution

    See [Detect solution](#detect-solution).

- Validate solution

    See [Validate solution](#validate-solution).

- Validate tag

    See [Validate tag](#validate-tag).

- Prepare solution

    See [Prepare solution](#prepare-solution).

- Sync DataMiner feature release DLLs

    See [Sync DataMiner feature release DLLs](#sync-dataminer-feature-release-dlls).

- Sync DIS version

    See [Sync DIS version](#sync-dis-version).

- Build QuickActions on latest feature release

    See [Build QuickActions on latest feature release](#build-quickactions-on-latest-feature-release).

- Scan test projects

    See [Scan Test Projects](#scan-test-projects).

- Run unit tests

    See [Run Unit Tests](#run-unit-tests).

- Run integration tests

    See [Run Integration Tests](#run-integration-tests).

- SonarQube analysis

    See [SonarQube analysis](#sonarqube-analysis).

- Convert solution to XML

    See [Convert Solution to XML](#convert-solution-to-xml).

- Initialize validator

    See [Initialize validator](#initialize-validator).

- Run validator

    See [Run Validator](#run-validator).

- Run major change checker

    See [Run Major Change Checker](#run-major-change-checker).

- Verify developer checklist.

    See [Verify developer checklist](#verify-developer-checklist).

- Quality Gate

    See [Quality Gate](#quality-gate).

- Prepare Driver Passport Platform Scheduling

    See [Prepare Driver Passport Platform Scheduling](#prepare-driver-passport-platform-scheduling).

- Schedule Driver Passport Platform

    See [Schedule Driver Passport Platform](#schedule-driver-passport-platform).

- (Development) DCP registration

    See [(Development) DCP registration](#development-dcp-registration).

- (Release) DCP registration

    See [(Release) DCP registration](#release-dcp-registration).

- (Release) Prepare for SVN

    See [(Release) Prepare for SVN](#release-prepare-for-svn).

- (Release) Push to SVN

    See [(Release) Push to SVN](#release-push-to-svn).

- Declarative post actions

    See [Declarative Post Actions](#declarative-post-actions).

## Loading Jenkinsfile

When a new Git repository is created using the SLC SE Repository Manager tool, the repository initially contains a .gitignore file and a Jenkinsfile. This Jenkinsfile in turn refers to another "master" Jenkins file. In this step, the Jenkinsfile gets loaded.

## Declarative checkout from SCM

In this step, Jenkins loads the current repository from Git.

## Detect solution

In this step, the repository is scanned for the presence of a Visual Studio solution (.sln) file.

The pipeline will only continue if exactly one solution file has been detected in the repository.

## Validate solution

This step verifies whether the QAction projects in the solution make use of NuGet packages. If they do, the pipeline will fail, as using NuGet packages in QAction projects is currently not supported.

This step also retrieves information from the protocol such as the integration ID, developer initials, DCP task ID, etc.

Based on the DCP task ID that is provided in the protocol, the pipeline will:

- Fill in the “Developer” in a task.

- Fill in the Task ID in the driver version record.

- Fill in the “Released In” version in the task on release.

- Create/update the driver installation record automatically on release.

Please note the following

- Multiple task IDs are supported. If there is no task ID, the pipeline will fail early and provide a comment indicating the problem.

- In case of special situations where a fix needs to be applied to over 5 drivers, a task still needs to be assigned to the version with the fix. This task cannot be a Driver Development, Driver Issue or New Driver Feature task because those types of tasks can only be linked to a single driver. You can get around this limitation by adding a task of a different type (e.g. Consultancy or Support). Tasks that are not of type Driver Development, Driver Issue or New Driver Feature will allow the pipeline to run, but the task and driver installation record registration will not be performed automatically.

- Remaining manual actions:

    - Pre-development:

        - Creating the new driver/vendor/customer records (also for DVEs).

        - Selecting the right driver in the task.

    - Workflow:

        - Adjusting completion percentage during development.

        - Changing the task status (in development, code review, QA, etc.)

        - Changing the task assignee.

    - Administration:

        - Registering the DVEs.

        - Create the driver help file(s).

## Validate tag

This step is only executed for pipeline runs for a tag. It will verify whether the specified tag meets the following conditions:

- The tag is in the expected branch. For example, a tag "1.0.0.1" provided on a commit that is part of the "1.0.0.X" branch will succeed, while a tag "1.0.0.1" provided on a commit belonging to branch 1.0.1.x will fail.

- All expected previous minor versions of the tag are present. For example, if a commit has been tagged with "1.0.0.4", the tags "1.0.0.1", "1.0.0.2" and "1.0.0.3" are expected to be present already.

## Prepare solution

In this step, the solution is configured to build against a recent version of the .NET Framework. This is to allow compiling against the latest feature release of DataMiner, which could require a new .NET Framework version compared to the one specified in the protocol solution. Note that this is just a local change, it does not change anything to the solution in the Git repository hosted by Gerrit.

## Sync DataMiner feature release DLLs

This step ensures that the next build step will build against the latest feature release of DataMiner. It will verify on DCP whether a new feature release has been released and, if so, Jenkins will make sure to use that feature release to build against from that point onwards.

## Sync DIS version

This step ensures that the pipeline uses the latest version of DIS. It verifies whether a new version has been released, and if that is the case, the new version is obtained.

## Build QuickActions on latest feature release

During this step, the solution is built against the latest DataMiner feature release.

## Scan Test Projects

This step scans the solution for the presence of any test projects. Projects with a name that end with "Integration Tests" or "IntegrationTests" (case insensitive) will be considered integration test projects. All other projects that end with "Tests" will be considered unit test projects.

## Run Unit Tests

This step executes the unit test projects. If no unit test projects were detected, this step is skipped.

> [!NOTE]
> In case the tests fail, the unit tests will be executed against DataMiner 10.0.3 CU1 (if the protocol supports this version). The purpose of this is to support unit tests that were created using the SLProtocol API up to version 10.0.3 CU1. RN 27995 introduced changes to the API that could make a unit test fail if it depends on the prior implementation of the API. If unit tests using the DataMiner DLLs of 10.0.3 CU1 are re-executed, tests that are failing because of the changed API will succeed in the second execution.

## Run Integration Tests

This step executes the integration test projects. If no integration test projects were detected, this step is skipped.

## SonarQube analysis

This step performs SonarQube C# code analysis on the QAction code.

## Convert Solution to XML

This step converts the protocol Visual Studio solution back to a protocol XML file.

## Initialize validator

This step initializes the validator settings by obtaining the previous version from SVN and running the validator on the previous version.

To indicate on which version a protocol is based, the *basedOn* attribute can be used. In case you create a new minor version, e.g. A.B.C.D where D \> 1, and you do not specify the previous version explicitly, the previous version will be assumed to be the previous minor version: A.B.C.D-1.

In case you create a new major or system range (i.e. D equals 1, and B or C do not equal 0), e.g. 1.1.0.1 or 1.0.1.1, then you are required to explicitly specify the version this protocol is based on.

In case you create a new branch version, e.g. 2.0.0.1, and you do not specify a based on version, then this will be assumed to be a brand-new development. The Validator quality gate settings for an initial version will therefore be applied.

## Run Validator

This step runs the validator on the protocol XML file that was generated in the previous step.

## Run Major Change Checker

For every new minor version, the pipeline will execute the DIS Major Change Checker to verify whether the new version does not have any major changes.

If it does, the major change checker stage will be marked as unstable

The results of the major change checker are also archived as artifacts.

## Verify developer checklist

Verifies whether the protocol implementation checklist is present. The checklist should have ".docx" as its extension and one of the following names (disregarding spaces and capitalization):

- Protocol Development Checklist

- Protocol Development Quick Checklist

- Checklist

Additionally, the following information in the checklist itself should correspond with the info provided in the protocol:

- Protocol name

- Protocol version

- Initials of integration engineer

> [!NOTE]
> You should always use the latest version of the checklist, which is available on Dojo: <https://community.dataminer.services/documentation/protocol-development-checklists/>

## Quality Gate

This step verifies the results of different previous pipeline steps and checks whether the results are according to some preconfigured quality level.

### Unit/Integration Tests

The quality gate will fail as soon as one test fails.

### Validator

The quality gate verifies whether the protocol does not exceed any of the limits set for the validator quality gate. For initial protocols (i.e. version 1.0.0.1), the following limits are configured:

- Critical: 0

- Major: 0

- Minor: 0

- Warning: No limitation

For non-initial versions, the validator quality gate settings will be configured based on the results of the previous version (i.e. the version this protocol version is based on):

- Critical issues: 0 allowed

- Major issues: Must not exceed the major issue count of the previous version (i.e. the version this protocol version is based on).

- Minor issues: Must not exceed the minor issue count of the previous version (i.e. the version this protocol version is based on).

- Warnings: Unlimited.

> [!NOTE]
> The following error codes are currently ignored:
> -  1401: "x% of monitored parameters do not have default alarm values set”

### SonarQube

This quality gate verifies whether the protocol does not exceed any of the limits set for the SonarQube code analysis. Currently, the following limits have been configured:

- Blocker Issues: 0

- Bugs: 20

- Critical Issues: 10

- Code Smells: 100

- Major Issues: 100

- Minor Issues: 100

- Duplicated Blocks: 200

> [!NOTE]
> The Quality Gate will currently only verify SonarQube analysis results for initial developments (i.e. protocols with version 1.0.0.1).

## Prepare Driver Passport Platform Scheduling

This stage is responsible for creating a DataMiner Test Package (.dmt). The test package includes references to the simulation files to use when running the test package.

This stage will be skipped if the protocol version is not an initial version of a range (i.e. the Minor value of the Version is not equal to 1).

No test package will be created if any of the following is applicable:

- The protocol has only virtual connections.

- The protocol has at least one connection type that is not supported by the Driver Passport Platform. The connection types that are currently supported by the Driver Passport platform are SNMP and HTTP.

- No simulation files could be found for the connections. This will mark the stage as unstable.

#### Simulation files

In order for the pipeline to generate a test package, all you need to do is provide the required simulation files. The simulation files must be provided on the shares in **the following folder**:

*S:\\Public\\Simulations\\Protocol Simulations\\*\<Vendor>*\\*\<ProtocolName>*\\*\<version>*\\*\<customer>

- **Vendor**: The name of the vendor as mentioned in the *Vendor* tag of the protocol.

- **ProtocolName**: The name of the protocol as mentioned in the *Name* tag of the protocol.

- **version**: The version of the protocol as mentioned in the *Version* tag of the protocol.

- **customer**: The name of the customer. The expected customer name is retrieved by the CI/CD pipeline via the task mentioned in the protocol.

> [!NOTE]
> -  In case multiple tasks are defined, which results in multiple customers, providing simulation files for only one customer is sufficient.
> -  In case no simulation files were found by the CI/CD for any of the expected customers, the pipeline will perform a fallback to a simulation of another customer (if present) for this version.

A simulation must be provided for each connection of the protocol. The **name of the simulation file** must be *Connection\_**\<connectionNumber>*, where *\<connectionNumber>* denotes the zero-based connection number, e.g. *Connection_0*.

For **SNMP simulations**, two files should be provided:

- *Connection\_\<connectionNumber>.txt*: This file contains the dynamic data.

- *Connection\_\<connectionNumber>.xml*: This is the SNMP simulation file.

    For more information on how to create this file, see [Realistic dynamic simulations](xref:Realistic_dynamic_simulations):

    - How to generate the file that holds the dynamic data (.txt): [Retrieving real device data](xref:Realistic_dynamic_simulations#retrieving-real-device-data)

    - How to create the file the SNMP simulation file (.xml): [Configuring the simulation file to poll the database](xref:Realistic_dynamic_simulations#configuring-the-simulation-file-to-poll-the-database)

The Driver Passport Platform will use these files to automatically start a QA Device Simulator instance and ingest the dynamic data into the database.

> [!NOTE]
> A simulation must also be provided for an SNMP connection that only processes traps. You can use the empty simulation file available in *S:\\Public\\Simulations\\DummySnmpSimulation_ForNonPollingConnections.zip* as the simulation file for such a connection.

For **HTTP simulations**, the following file should be provided:

- *Connection\_\<connectionNumber>.pdml*

    For more information on how to create this file, see [Creating HTTP simulations](xref:Creating_HTTP_simulations).

## Schedule Driver Passport Platform

This stage will push the created DataMiner Test (.dmt) package to the Driver Passport Platform. It is only executed for protocol versions that are an initial version of a range (i.e. the Minor value of the Version is equal to 1).

If in the previous stage no test package could be created, this stage will be marked as unstable.

For more information about the Driver Passport Platform, see [Skyline Driver Passport Platform](xref:TOODriverPassportPlatform#skyline-driver-passport-platform).

## (Development) DCP registration

This step takes care of the automatic registration on DCP in case the protocol is still in development.

It first verifies whether the protocol has an integration ID specified. If this is not the case, the pipeline will fail.

Next, based on the integration ID specified in the protocol, the DCP driver record is obtained from DCP. In case DCP does not have a DCP driver record with the specified integration ID, the pipeline will fail.

Next, it will check the status of the driver in this driver record. If it has been marked as "Released", the pipeline will fail.

At this point, information about this driver version is obtained from the *VersionHistory* tag in the protocol. Note that the protocol must have a *VersionHistory* tag and this *VersionHistory* tag must hold an entry for the current version. Also, the *Author* tag must be filled in with the correct initials of the developer and a valid date must be specified.

The comments that will be provided for this driver version on DCP are based on the information provided in the version history.

The driver version state of the DCP driver version record will be set to “Development”.

> [!NOTE]
> if there is no record for this driver version yet on DCP, a new one will be created. Otherwise, the existing one will be updated.

## (Release) DCP registration

This step takes care of the automatic registration on DCP in case the protocol is released.

It performs the same actions as the development registration except that the driver version state of the DCP driver version record will be set to “Released”, and the *LiveUpdate* flag will be set.

## (Release) Prepare for SVN

In case a tag was detected and the version should therefore be pushed to SVN, some preparatory steps are performed.

## (Release) Push to SVN

This step performs the actual push to SVN. Once this step is executed, you should find a new version of the protocol on SVN in the corresponding folder, together with the required DLLs, which were originally provided in the DLLs folder in the Visual Studio project.

## Declarative Post Actions

This step performs cleanup of the workspace and send an email containing a report giving an overview of the number of issues detected in DIS and SonarQube.

The report also contains an overall quality score, which is calculated using the following metrics:

- Number of Critical Issues reported by the DIS Validator

- Number of Major Issues reported by the DIS Validator

- Number of Minor Issues reported by the DIS Validator

- Number of Blocker Issues reported by SonarQube

- Number of Critical Issues reported by SonarQube

- Number of Major Issue reported by SonarQube

![](~/develop/images/PipelineEquation.png)
