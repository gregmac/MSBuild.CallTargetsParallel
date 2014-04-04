MSBuild CallTargetsParallel
===========================

This is an MSBuild task to invoke multiple targets in parallel. It invokes a new instance of msbuild for each target. 


Use
---

#### Simple target invocation

A simple use is:

```xml
<CallTargetsParallel Targets="Target1;Target2" GlobalProperties="PropertyName1;PropertyName2" />
```

Runs Target1 and Target2 in this project file, passing along the current values of the properties PropertyName1 and PropertyName2. 


#### Installation

Include the CallTargetsParallel.tasks file somewhere in your project, then in the MSBuild file include the line:

```xml
<Import Project="CallTargetsParallel.tasks"/>
```

#### All features

```xml
<ItemGroup>
    <ParallelTargets Include="_global">
        <Property1>value1</Property1>
    </ParallelTargets>
    <ParallelTargets Include="Target1">
        <Property2>value2</Property2> 
    </ParallelTargets>
    <ParallelTargets Include="TargetTwo">
        <Targets>Target2</Targets>
        <Property1>some "value"</Property1>
    </ParallelTargets>
    <ParallelTargets Include="DualExample">
        <Targets>Target1;Target2</Targets>
    </ParallelTargets>
    <ParallelTargets Include="OtherProject">
        <MSBuildFile>myproject.proj</MSBuildFile>
        <Targets>Build</Targets>
        <Property3>$(Property3)</Property3>
    </ParallelTargets>
</ItemGroup>
<CallTargetsParallel Targets="@(ParallelTargets)" GlobalProperties="Property5" />
```


This will invoke 4 msbuild instances in parallel:
* instance "Target1" will run target Target1, with values Property1=value1, Property2=value2, and Property5 (defined elsewhere)
* instance "TargetTwo" will run target Target2, with values Property1=some "value", and Property5 (defined elsewhere)
* instance "DualExample" will run targets Target1 and Target2, with values Property1=value1, and Property5 (defined elsewhere)
* instance "OtherProject" will run the Build target in myproject.proj, with the current values of Property3 and Property5 (both defined elsehwhere)



#### Item definitions:

* Include= must have a unique name in the group
* The name "_global" is reserved to define property values that apply to all other items in the group. No targets will be run even if specified.
* The metadata in each include are used as property names, and they cannot be MSBuild reserved names (http://msdn.microsoft.com/en-us/library/ms164313.aspx)
* The <Targets> metadata item specifies which target(s) to actually invoke. If not specified, the item name (Include= value) is used
* The <MSBulidFile> metadata item specifies which msbuild project file to run. If not specified, the current file is used.
    
    
#### Limitations:
* Property values defined in the target where <CallTargetsParallel/> is invoked cannot be passed. Use DependsOnTargets if other targets need to be called to set property values.
    
Inspiration
-----------

It's quite similar to the MSBuildExtensionPack Parallel task (http://www.msbuildextensionpack.com/help/4.0.8.0/html/23e4198a-c266-e8a3-4aea-7cf131a0837c.htm) but differs in the way it is invoked, making it significantly easier to pass existing properties without worrying about escaping values for command-line execution. I wrote this because I was trying to use parallel tasks in a complex project, and MSBuildExtensionPack's implementation meant I had giant easy-to-mess-up blocks of parameters to be passed to each invocation, and it quickly became unreadable.

License
-------
"New" BSD: http://opensource.org/licenses/bsd-license.php
