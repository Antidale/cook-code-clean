+++
title = "Updating to .NET 10 and MTP"
date = 2025-11-12
[taxonomies]
tags=["code","testing","ci","dotnet",".net10"]
+++

A bit of a follow up to [this testing ci and mtp](@/blog/testing-ci-mtp.md) post. [.NET 10](https://devblogs.microsoft.com/dotnet/announcing-dotnet-10/) is released, and with that some rejoicing (I really like idea of the new extension stuff, the field change is also cool, and null-conditional assignment is nice to have) is in order, and also there are some changes to having your tests run as you're upgrading.

This post is specific to what I had to do when updating to .net 10 from .net 9 while using xunit. The basic Xunit.v3 package does not work with MTP v2, so the upgrade path is slightly more complicated than just updating to the latest versions of the standard packages.

First up, if you don't have a [global.json](https://learn.microsoft.com/en-us/dotnet/core/tools/global-json) file, which goes at the same level as a `.sln` or `.slnx` file, you'll want to add that. The `dotnet new globaljson` command ([docs here](https://learn.microsoft.com/en-us/dotnet/core/tools/global-json)) gets you started, and then add in this section: 
```json
    "test": {
        "runner": "Microsoft.Testing.Platform"
    }
```

Next, since xunit v3 seems to not support MTP v2 currently (as of 3.2.0) in the main package, you'll need to change to a version that does, so remove the `xunit.v3` or `xunit.v3.core` you're currently using and add in `xunit.v3.mtp-v2` or `xunit.v3.core.mtp-2`. This [github comment](https://github.com/xunit/xunit/issues/3416#issuecomment-3447785145) has more detail about the change, and further down in the discussion is why the change isn't incoporated into the base `xunit.v3` package.

For some specific package version numbers that are working for me today:
 * Microsoft.NET.Test.Sdk 18.0.1
 * xunit.v3.mtp-v2 3.2.0
 * Microsoft.Testing.Extensions.TrxReport 2.0.2 (for creating a .trx as part of a ci build)

And with adding in the global.json file with the test section, changing slightly the xunit package you're referencing, and just generally getting all your packages up-to-date with .net 10, your tests should run and pass just as they did before the update.

### Troubleshooting

These are here mostly to try and catch anyone searching for specific errors, since everything's covered above. But also, [repetition legitimzes](https://www.youtube.com/watch?v=LlmTWlaWs_o&t=150s).

#### TypeLoadException
`System.TypeLoadException: Could not load type 'Microsoft.Testing.Platform.Extensions.TestHost.IDataConsumer' from assembly 'Microsoft.Testing.Platform, Version=2.0.0.0`

This error notes that you're using MTP v2, and your version of xunit doesn't support that. To keep using MTP v2, make sure sure to replace your package reference from `xunit.v3` to `xunit.v3.mtp-v2` ([further information](https://github.com/xunit/xunit/issues/3416#issuecomment-3447785145)). If you're using centralized package managemnet, make sure you're changing both the Directory.Packages.props file, and the test projects .csproj file.

#### VSTest vs MTP mismatch
`.nuget/packages/microsoft.testing.platform.msbuild/2.0.1/buildMultiTargeting/Microsoft.Testing.Platform.MSBuild.targets(263,5): error Testing with VSTest target is no longer supported by Microsoft.Testing.Platform on .NET 10 SDK and later. If you use dotnet test, you should opt-in to the new dotnet test experience. For more information, see https://aka.ms/dotnet-test-mtp-error`

If you run your tests and get the above error, you've forgotten to add in your `global.json` file, or omitted the tests section in that file.