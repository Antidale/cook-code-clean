+++
title = "make code"
date = 2025-08-10
[taxonomies]
tags=["code","tools"]
+++

One of the things I find to be small hassle in using the dotnet cli for running projects via `dotnet run` is having to specify the target project as soon as I add some testing projects to the solution. I know I could always `cd` to the folder of the relevant `.csproj` file, but that's roughly equal hassle, and I generally prefer to just stay at the root of a project most of the time. I took a little time recently to see if maybe things had changed and you could specify a default project in a `.sln` or `.slnx` file, but alas. 

However, this [stack overflow answer suggesting using a Makefile](https://stackoverflow.com/a/59942079) is a really good solution for me. As a solution, I think it is very customizable, less of a bother than making shell scripts for it, and clear what's going on when you look at it. My favorite little thing is that running `make watch-test` doesn't actually change the directory of the terminal, so when I stop running tests under the watch, I'm still at the directory root. Delightful!

Here's what I currently have going on in for the one place I'm currently using it (folder names redacted)
```make
watch:
	dotnet watch run --project ./SomeFolder/SomeFolder.csproj
run:
	dotnet run --project ./SomeFolder/SomeFolder.csproj
watch-test:
	cd ./SomeFolder.UnitTests && dotnet watch test
outdated:
	dotnet outdated -exc FluentAssertions
outdated-upgrade:
	dotnet outdated -exc FluentAssertions -u
```