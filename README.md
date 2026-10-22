# .NET Migration Assessor

![Status: pre-alpha](https://img.shields.io/badge/status-pre--alpha-orange)
![.NET 10](https://img.shields.io/badge/target-.NET%2010-512BD4)
![License: Apache-2.0](https://img.shields.io/badge/license-Apache--2.0-blue)

**Point it at a .NET Framework or .NET 8 solution. Get a ranked, explained migration plan to .NET 10.**

> **Status: pre-alpha.** The project is under active design and development. There is no installable release yet.
> Follow progress in this repository or join the waitlist at [developzone.dev](https://developzone.dev/#waitlist).

---

## Why this exists

A large share of production .NET code still runs on .NET Framework 4.x, and .NET 8 reaches end of support on November 10, 2026. Teams that need to move to .NET 10 (the current LTS) usually face the same questions:

- Which projects can move now, and which are blocked?
- What is blocking them: packages, APIs, project format, or dependencies on other projects?
- In what order should we migrate so the solution keeps building?
- How much work is this, and where is the risk?

Answering these by hand on a solution with dozens of projects takes days of reading code. .NET Migration Assessor aims to answer them in minutes, with every recommendation traceable to a concrete fact in the code.

## How it works

```
your .sln  →  Graphwright · facts  →  LLM · reasoning  →  ranked migration plan
```

1. **Facts.** [Graphwright](https://github.com/developzoneio/graphwright), a Roslyn-based code graph engine, analyzes the solution: projects, target frameworks, project format (SDK-style or legacy), package references, project-to-project references, API usage and call graph.
2. **Rules.** Deterministic checks turn those facts into findings: incompatible packages, APIs that do not exist on modern .NET, legacy project files, blocking dependencies.
3. **Reasoning.** A language model reads the findings (not the raw source by default) and explains each one: why it matters, how to fix it, and how risky the fix is.
4. **Plan.** The tool produces a ranked plan: migration order (leaf projects first), blockers per project, and an effort and risk estimate.

## Design principles

- **Facts before reasoning.** Every item in the plan links back to a fact from the code graph. The model explains findings; it does not invent them.
- **Your code stays local by default.** Only structured findings are sent to the model. Sending source snippets is opt-in.
- **Explainable output.** Each recommendation says what was found, where, and why it is ranked where it is.
- **Pluggable reasoning provider.** The reasoning step sits behind an interface (`IMigrationReasoner`), so the model provider is a configuration detail, not a dependency of the core.
- **Readable reports.** Output is Markdown for people and JSON for automation and CI.

## Planned scope (v0.1)

- [ ] Solution inventory: projects, target frameworks, SDK-style vs legacy `.csproj`
- [ ] Package compatibility check against .NET 10
- [ ] Detection of APIs unavailable on modern .NET (for example `System.Web`, WCF server, `AppDomain`, .NET Remoting, `BinaryFormatter`)
- [ ] Project dependency graph and leaf-first migration order
- [ ] Per-project blockers with explanations
- [ ] Ranked plan as Markdown and JSON
- [ ] `dotnet` global tool: `dotnet assess ./MySolution.sln`

## Example output (illustrative, format not final)

```text
Migration plan: Contoso.Billing.sln → net10.0
24 projects · 6 ready · 13 need changes · 5 blocked

#  Project                      From      Status        Effort  Risk
1  Contoso.Billing.Domain       net48     ready         S       low
2  Contoso.Billing.Contracts    net48     needs changes S       low    legacy csproj
3  Contoso.Billing.Data         net48     needs changes M       medium EF6 → EF Core
...
21 Contoso.Billing.Web          net48     blocked       L       high   System.Web (14 call sites)
```

## Architecture

The tool follows Clean Architecture. Dependencies point inward only.

```
src/
├─ DevelopZone.MigrationAssessor.Domain/          Findings, severity, migration steps, ranked plan
├─ DevelopZone.MigrationAssessor.Application/     Use cases and ports (ICodeGraphProvider, IMigrationReasoner, IReportWriter)
├─ DevelopZone.MigrationAssessor.Infrastructure/  Graphwright adapter, model provider adapters, report writers
└─ DevelopZone.MigrationAssessor.Cli/             dotnet global tool (presentation layer)
tests/
├─ unit tests per layer
└─ samples/                                       small .NET Framework and .NET 8 solutions used as fixtures
```

## Roadmap

| Phase | Goal |
| --- | --- |
| 0 | Repository, architecture, sample legacy solutions as test fixtures |
| 1 | Inventory and dependency graph via Graphwright |
| 2 | Rule engine for packages, APIs and project format |
| 3 | Reasoning step and ranked plan (Markdown + JSON) |
| 4 | Public preview as a `dotnet` global tool |
| 5 | CI mode: fail a build when new migration blockers are introduced |

## Getting started

Nothing to install yet. To follow along:

- Star or watch this repository for releases.
- Join the waitlist at [developzone.dev](https://developzone.dev/#waitlist).

## Contributing

Contributions are welcome, especially at this early stage:

- **Legacy patterns.** Open an issue describing a .NET Framework pattern that made your own migration hard. Real-world cases become rules and test fixtures.
- **Sample solutions.** Small, anonymized solutions that reproduce a migration problem are very valuable.
- **Design feedback.** Comment on open design issues before the v0.1 API is fixed.

Please open an issue before starting a large pull request.

## Related projects

- [Graphwright](https://github.com/developzoneio/graphwright): Roslyn-based code graph engine and .NET MCP server
- [Specwright](https://github.com/developzoneio/specwright): spec-driven development workflow for AI coding assistants

## Maintainer

Built and maintained by **Trong Tran**, founder of [DevelopZone](https://developzone.dev), a .NET backend engineer with 12+ years of experience.

- Website: [trongtran.developzone.dev](https://trongtran.developzone.dev)
- GitHub: [@developzoneio](https://github.com/developzoneio)
- Contact: [hello@developzone.dev](mailto:hello@developzone.dev)

## License

[Apache-2.0](LICENSE) © 2026 DevelopZone
