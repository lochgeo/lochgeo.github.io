---
layout: post
title:  "Fat Images and Thin Runtimes - Multi-stage Docker for .NET Core"
date:   2019-09-19 19:34:00
categories: [Software, Containers & Kubernetes]
excerpt_separator: "<!--more-->"
---

I spent part of last weekend trying to containerize a small ASP.NET Core API on my Windows laptop. The first image I produced "worked" — `docker run` answered on port 5000 — and then I looked at the size. Over a gigabyte for a service that is basically a couple of controllers and EF Core. That is not a deployable unit; that is me shipping the compiler to production by accident.

The fix is not exotic. Multi-stage Docker builds have been around since Docker 17.05, and the .NET team has been pushing the pattern hard as .NET Core 3.0 gets close to GA. Build with the SDK image, run with the ASP.NET runtime image, copy only the published output across. Once you see it, you cannot unsee the single-stage Dockerfile you wrote last month.

<!--more-->

### The accidental SDK in production

My first Dockerfile looked roughly like this (I am paraphrasing the shameful bits):

```
FROM mcr.microsoft.com/dotnet/core/sdk:2.2
WORKDIR /app
COPY . .
RUN dotnet publish -c Release -o out
ENTRYPOINT ["dotnet", "out/MyApi.dll"]
```

It builds. It runs. It also keeps the entire SDK — compilers, MSBuild, NuGet client, the lot — sitting next to your app at runtime. Microsoft publishes images through the Microsoft Container Registry now (`mcr.microsoft.com/dotnet/core/...`), and they deliberately split SDK from runtime for this reason. If your final `FROM` is an SDK tag, you are doing it wrong.

### Two stages, one image

A multi-stage Dockerfile is just multiple `FROM` lines in one file. Each `FROM` starts a fresh filesystem. Only the last stage is what you push. Everything earlier is scaffolding.

```
FROM mcr.microsoft.com/dotnet/core/sdk:2.2 AS build
WORKDIR /src
COPY MyApi.csproj .
RUN dotnet restore
COPY . .
RUN dotnet publish -c Release -o /app --no-restore

FROM mcr.microsoft.com/dotnet/core/aspnet:2.2 AS final
WORKDIR /app
COPY --from=build /app .
ENTRYPOINT ["dotnet", "MyApi.dll"]
```

A few things that matter in practice:

- Copy the `.csproj` first, restore, then copy the rest of the source. Docker layer caching means a code-only change does not re-download every NuGet package.
- `--no-restore` on publish because you already restored in the previous layer.
- Final stage uses `aspnet` (or `runtime` for a console worker), never `sdk`.
- `COPY --from=build` is the whole trick — only published binaries cross the boundary.

On my machine the runtime image landed in the low hundreds of megabytes instead of north of a gig. Pulls got faster. The attack surface got smaller because the compiler is no longer in the box.

### Windows quirks I actually hit

I develop on Windows, so a few local frictions showed up before the Dockerfile even mattered.

Docker Desktop still makes you pick a side: right-click the whale tray icon and **Switch to Linux containers** or **Switch to Windows containers**. You cannot mix them in one daemon session. For .NET Core APIs I stay on Linux containers — the images are smaller and match what most Linux hosts will run in CI and production. Windows containers are the path when you are stuck on full Framework or need Windows-only dependencies.

Hyper-V has to be on. If the Moby VM will not start, that is usually where I look first. Bind-mounting source from `C:\Users\...` into a Linux container works, but file-watch and I/O performance can feel sluggish; for day-to-day edit-run I still prefer `dotnet run` on the host and reserve Docker for build-and-smoke-test.

Also add a `.dockerignore`. Without it, `COPY . .` drags in `bin/`, `obj/`, `.git/`, and whatever else is lying around. Mine is boring and effective:

```
**/bin/
**/obj/
.git/
.vs/
*.user
```

### Resource limits are not optional homework

One reason I care about image shape right now is that .NET Core 3.0 is tightening how the runtime behaves inside containers. The team has been writing about honouring Docker memory and CPU limits properly — cgroups on Linux, job objects on Windows — instead of the runtime assuming it owns the whole machine. Even on 2.2 today, if you run a container without a memory limit in a shared environment, you are trusting every other process to be polite. I have started putting explicit `--memory` and `--cpus` on local `docker run` just to force the same shape I expect in real hosts.

Smaller images do not replace good limits. They just mean you are not also paying to pull and store a compiler you never invoke.

### What I am standardising on

For new .NET Core services I am treating this as the default checklist:

- Multi-stage Dockerfile: SDK build stage, ASP.NET/runtime final stage
- Project file copied before source for restore caching
- `.dockerignore` checked in next to the Dockerfile
- MCR image names (`mcr.microsoft.com/dotnet/core/...`), pinned to a major.minor tag I intend to service
- Linux containers locally unless the workload truly needs Windows
- A quick `docker images` glance after every "it works" moment — size is a smell

None of this is clever. It is the difference between a container that is a packaging format and a container that is a suitcase full of build tools. If your production image still has `dotnet build` available on the PATH, go rewrite the Dockerfile before someone else notices the gigabyte pull.
