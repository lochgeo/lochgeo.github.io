---
layout: post
title:  "Spring Boot for a .NET Brain - Same Job, Different Dialect"
date:   2023-05-09 10:18:00
comments: True
categories: [Architecture]
excerpt_separator: "<!--more-->"
---

For years my muscle memory has been C# and ASP.NET Core. Controllers, middleware, `IServiceCollection`, `appsettings.json`, NuGet, ship it. That is still how I think when nobody is watching. Lately the day job is pulling me into Java land — Spring Boot, Maven, a different IDE, and a package namespace that used to be `javax` and is now stubbornly `jakarta`.

I am not rewriting my career overnight. I am learning the dialect so I can read the codebase, review a PR without guessing, and stop treating the Java services on our map as black boxes. If you are a .NET person staring at Spring Boot 3 for the first time, this is the cheat sheet I wish someone had handed me on day one.

<!--more-->

### Why this feels familiar (and why that is dangerous)

Spring Boot and ASP.NET Core are solving the same problem: get a web service running with less ceremony than the old frameworks. Both give you dependency injection, an embedded HTTP server, opinionated project layout, and a pile of starters so you are not hand-wiring every library.

That familiarity is a trap. The *shape* of the solution matches. The *idioms* do not. If you write Java that is just C# with semicolons in different places, you will fight the framework and your teammates will gently hate your pull requests.

Spring Boot 3.0 went GA in November 2022 — Java 17 baseline, Spring Framework 6, Jakarta EE packages, first-class GraalVM native images, and observability wired through Micrometer. That is the generation I am learning against. Starting on Boot 2.x "because it is what the wiki says" is how you learn the wrong defaults.

### The mental map I keep on a sticky note

| .NET thing I reach for | Spring Boot cousin |
| --- | --- |
| `dotnet new webapi` | [start.spring.io](https://start.spring.io) (or the IDE wizard that hits the same service) |
| `.csproj` + NuGet | `pom.xml` (Maven) or `build.gradle` |
| `Program.cs` / minimal hosting | `@SpringBootApplication` + `SpringApplication.run` |
| `IServiceCollection` registration | `@Component` / `@Service` / `@Repository` + constructor injection |
| `[ApiController]` + `[HttpGet]` | `@RestController` + `@GetMapping` |
| `appsettings.json` + env vars | `application.properties` or `application.yml` |
| EF Core | Spring Data JPA |
| Serilog / `ILogger<T>` | SLF4J + Logback (Boot's default stack) |
| Health checks | Spring Boot Actuator |
| Kestrel | Embedded Tomcat by default (Jetty/Undertow if you switch) |

None of these are one-to-one. They are enough to stop panicking when someone says "just add a starter."

### What actually slowed me down

**Constructor injection is the happy path.** In ASP.NET Core I got used to the container being explicit in `Program.cs`. In Spring, a lot of wiring is "the class is annotated, the constructor parameters are beans, Spring figures it out." That felt like magic until I broke a bean and spent an hour staring at a startup failure that was really a missing component scan.

**Configuration is flatter than you think.** `application.yml` profiles map cleanly to the idea of `appsettings.Development.json`, but the property names and the way overrides cascade are their own sport. I stopped inventing clever hierarchical objects and started reading how the existing services already name things.

**`javax` vs `jakarta` will bite you if you copy old samples.** Boot 3 moved with Jakarta EE. Stack Overflow answers from 2019 still show `javax.persistence` and `javax.validation`. Those compile errors are not mysterious — they are time traveling. Prefer docs and samples written for Boot 3 / Framework 6.

**Build tools are a second language.** Maven's lifecycle (`compile`, `test`, `package`) is not hard, but it is not `dotnet build`. I stopped fighting Gradle vs Maven holy wars and matched whatever the repo already used. Consistency beats preference when you are the new person.

**The IDE switch is real.** Visual Studio / Rider muscle memory does not transfer cleanly to IntelliJ. Debugging a failed context refresh is a different ritual than debugging a failed host build. Budget a weekend to learn the debugger and the "bean is not a candidate" breadcrumbs, not just the language syntax.

### How I am learning without boiling the ocean

I am not trying to become a Java architect in a month. I am trying to be useful.

1. **One tiny service end to end.** Generate from start.spring.io with Web + whatever persistence the team actually uses. One GET, one POST, one test. No domain masterpiece.
2. **Read production code before writing opinions.** Style guides and "how we do transactions here" live in the repo, not in my head from C#.
3. **Map failures, not just happy paths.** Startup errors, missing beans, profile not active, test slice that does not load the security config — that is where the real learning is.
4. **Keep shipping in .NET where I still own things.** Context switching is expensive. I am not abandoning the stack I know; I am adding a second one on purpose.

The goal is dual literacy. Plenty of shops run both ecosystems. Pretending one is morally superior is a great way to stay unhelpful in architecture reviews.

### What I am not doing yet

Native images with GraalVM are interesting — Boot 3 made them a first-class path — but I am not optimizing cold starts before I can write a boring REST controller without googling annotations. Same with the deeper Spring Security filter chain. I know enough OAuth and JWT from the .NET side to be dangerous; I will learn the Spring Security DSL when a real ticket forces it, not as a weekend vanity project.

Observability is the one area I am leaning into early. Actuator plus Micrometer is the same conversation we already have with OpenTelemetry and metrics backends. The names change; the need to see latency and error rates does not.

### My take

Crossing from ASP.NET Core to Spring Boot is less "learn programming again" and more "unlearn which ceremony is load-bearing." The concepts transfer. The defaults, package names, and failure modes do not.

If you are a .NET developer being asked to touch Java services in 2023, start with Boot 3, Java 17, and one small vertical slice. Steal the mental map, ignore the framework wars, and treat every confusing startup log as a teacher. I still reach for C# when I sketch an idea on paper. The point is I can open a Spring service on Monday morning and not feel like a tourist with a phrasebook.
