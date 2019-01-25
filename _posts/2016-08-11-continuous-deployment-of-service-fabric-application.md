---
title: Continuous Deployment of Service Fabric Application
date: 2016/08/11 21:21:00 +530
layout: single
categories: 
   - Azure
tags:
   - azure
   - cloud
   - .net
   - csharp
   - servicefabric
---

Once an application is deployed to a service fabric cluster, you have two options to deploy your existing code.

1. Redeploy – this would remove the existing application and wipe out the State of the services in the application. you wouldn’t want that now, do you?.

2. Upgrade the existing services – This is most apt of pushing your latest code and still maintain the state of the actors and services. Esp. when you have a continuous deployment in place.

The configurations for each services is stored in “ServiceManifest.xml” of each service, placed under the folder of PackageRoot.

![Manifest](/assets/images/servicemanifest.png)

A sample service manifest would be as shown below

```xml
<?xml version="1.0" encoding="utf-8"?>
<ServiceManifest xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" Name="SampleActorPkg" Version="1.0.0" xmlns="http://schemas.microsoft.com/2011/01/fabric">
  <ServiceTypes>
    <StatefulServiceType ServiceTypeName="SampleActorServiceType" HasPersistedState="true">
      <Extensions>
        <Extension Name="__GeneratedServiceType__" GeneratedId="533af60b-39a4-4a78-9fc9-ae78dc9d1abf|Persisted">
          <GeneratedNames xmlns="http://schemas.microsoft.com/2015/03/fabact-no-schema">
            <DefaultService Name="SampleActorService" />
            <ServiceEndpoint Name="SampleActorServiceEndpoint" />
            <ReplicatorEndpoint Name="SampleActorServiceReplicatorEndpoint" />
            <ReplicatorConfigSection Name="SampleActorServiceReplicatorConfig" />
            <ReplicatorSecurityConfigSection Name="SampleActorServiceReplicatorSecurityConfig" />
            <StoreConfigSection Name="SampleActorServiceLocalStoreConfig" />
          </GeneratedNames>
        </Extension>
      </Extensions>
    </StatefulServiceType>
  </ServiceTypes>
  <CodePackage Name="Code" Version="1.0.0">
    <EntryPoint>
      <ExeHost>
        <Program>SampleActor.exe</Program>
      </ExeHost>
    </EntryPoint>
  </CodePackage>
  <ConfigPackage Name="Config" Version="1.0.0" />
  <Resources>
    <Endpoints>
      <Endpoint Name="SampleActorServiceEndpoint" />
      <Endpoint Name="SampleActorServiceReplicatorEndpoint" />
    </Endpoints>
  </Resources>
  <!-- The content will be generated during build -->
</ServiceManifest>
```

To upgrade the service the manifest version should be bumped to the next higher version, for example in above file ServiceManifest.Version =1.0.1 (actual version of the Service), ServiceManifest.CodePackage.Version=1.0.1 (Assembly version),  ServiceManifest.ConfigPackage.Version=1.0.1 (Configuration version).

Along with the manifest version bump, the assemblyversion of the package should also be bumped.

To summarize following are the steps to do a continuous deployment of any service fabric application. (Most of the steps mentioned below is possible in any CD tool)

1. Download Source Code to Working Directory
2. Update AssemblyInfo.cs files of all the services in the solution to the next version. You can get the powershell script to do that here.
3. Update the ServiceManifest.xml of all the services in the solution to next version. You can get the powershell script here.
4. Build the Solution file. (With x64 Configuration only as Service fabric works with x64 assemblies)
5. Run Unittests – If any
6. Deploy the application using powershell script  Deploy-FabricApplication.ps1 which is generated by Visual studio by default.
```powershell
..\Scripts\Deploy-FabricApplication.ps1 -PublishProfileFile ..\PublishProfiles\Cloud.xml -ApplicationPackagePath ..\pkg\$env:RELEASE_MODE 
-OverrideUpgradeBehavior $env:OverrideUpgradeBehavior -OverwriteBehavior Always
```

If there is a service which is removed or added in the latest version then the OverrideUpgradeBehavior should be set to VetoUpgrade else set to None.

For our example I have used Jenkins as the CD tool.

Happy Coding!!!

### References
[https://azure.microsoft.com/en-in/documentation/articles/service-fabric-application-upgrade-tutorial/](https://azure.microsoft.com/en-in/documentation/articles/service-fabric-application-upgrade-tutorial/)
[https://azure.microsoft.com/en-in/documentation/articles/service-fabric-cluster-upgrade/](https://azure.microsoft.com/en-in/documentation/articles/service-fabric-cluster-upgrade/)