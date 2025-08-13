+++
title = "Unit Testing, CI, & Microsoft Test Platform"
date = 2025-08-12
[taxonomies]
tags=["code","testing","ci","dotnet"]
+++

## The TL,DR:
If you want to have tests that are using Microsot.Testing.Platform (MTP) instead of VSTest and you want a `.trx` file so that you can have your CI pipeline have all what it needs to report on the tests that are running, you need to make sure to add the [Microsoft.Testing.Extensions.TrxReport](https://www.nuget.org/packages/Microsoft.Testing.Extensions.TrxReport/) nuget package. If you want a little more direction, you can check out the code examples below.

## The story up to that point
[Microsoft Testing Platform](https://learn.microsoft.com/en-us/dotnet/core/testing/microsoft-testing-platform-intro?tabs=dotnetcli)
is an alternative/update to VSTest, which up to now had been the underlying mechanism for how tests generally ran. You can check out some differences [between them](https://learn.microsoft.com/en-us/dotnet/core/testing/microsoft-testing-platform-vs-vstest), but for the most part, I was just looking to check out a new tool and try some stuff out. Also, I'd heard about [Tunit](https://tunit.dev/docs/intro), so I used it for a quick little project I had and figured that I could use a little practice setting up some CI pipelines. 

Reading documentation, it seemed mostly straightfoward to make the switch. Take out some NuGet packages you didn't need anymore, update your project to output as an EXE, maybe add a couple of properties to your .csproj and things should Just Work. Except they didn't. By default, stuff using MTP doesn't output a `.trx` file, instead putting a `.log` file in a TestResults file where all the binaries are for that compilation. CI stuff seems like it still expects to see `.trx` files, which: totally understandable and correct. Not only has it been around for ages, it also has lots of detail, which the `.log` files don't; they're just pretty much the same high level output as you see on the console when running tests through `dotnet test`.

The main trouble I ran into was getting `MSBUILD : error MSB1001: Unknown switch.` when trying arguments that each TUnit and XUnit suggest should work to get a `.trx` to be generated. Sleeping, more trial and error, and just doing the right searching on TUnit's documentation site got me to where I saw that I was missing a required NuGet package, and after adding it both of the projects I was tinkering with started working as I wanted in my ci pipelines. I've since seen a note that I must have just glossed over in Tunits docs that would have saved me some time, but it was divorced enough from the context in my head for what I wanted ("I need something to make this work on CI!"), that I was blind to the context where the information was offered, and where it truthfully fits a lot better than where I imagined I wanted it. 

## Example

### Azure Pipelines
```yml
- script: dotnet run --project ./FreeEnterprise.Api.UnitTests/FreeEnterprise.Api.UnitTests.csproj -c Release --report-trx --results-directory . --report-trx-filename results.trx
  displayName: 'Run tests'
  continueOnError: false

- task: PublishTestResults@2
  displayName: 'Publish Test Results'
  inputs:
    testResultsFormat: VSTest
    testResultsFiles: 'results.trx'
    searchFolder: '.'
    failTaskOnFailedTests: true
    failTaskOnMissingResultsFile: true
```

I could almost certainly specify things a bit less stringently. I'd started with sample code that I *mostly* copied from [Tunit's Documentation](https://tunit.dev/docs/intro) for [TUnit in CI pipelines](https://tunit.dev/docs/examples/tunit-ci-pipeline) and very much missed the part in the script step where the `--results-directory` gets set, but included the specified folder in the publish step, and the step kept failing, because it was looking in the wrong place. For what I'm doing, having the test results populate in the current directory is fine. I also had trouble having the script use `dotnet test` over `dotnet run`, even with using `-- --report-trx` as expected (and after adding the TrxReport nuget package) and work out on the CI servers, so `dontnet run` it is.


### .csproj files
For reference, here are examples of the .csproj files for two different of my test projects, the first using TUnit, the second using XUnit. 
#### TUnit
```xml
  <PropertyGroup>
    <!-- Minimim required -->
    <OutputType>Exe</OutputType>
  </PropertyGroup>
  <!-- This is actually my entire ItemGroup for NuGet pacakges in this project -->
  <ItemGroup>
    <PackageReference Include="Microsoft.Testing.Extensions.TrxReport" Version="1.8.2" />
    <PackageReference Include="TUnit" Version="0.55.6" />
  </ItemGroup>
```

### XUnit
```xml
    <PropertyGroup>
        <!-- 
            Removing some not relevant things in the group
        -->
        <OutputType>Exe</OutputType>
        <UseMicrosoftTestingPlatformRunner>true</UseMicrosoftTestingPlatformRunner>
        <TestingPlatformDotnetTestSupport>true</TestingPlatformDotnetTestSupport>
    </PropertyGroup>
    <ItemGroup>
        <!-- 
            Again, minimal entries required are shown
        -->
        <PackageReference Include="Microsoft.Testing.Extensions.TrxReport" />
        <PackageReference Include="xunit.v3" />
    </ItemGroup>
```
