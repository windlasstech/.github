<div align="center">

<!-- markdownlint-disable MD026 -- "Inc." is an abbreviation, not trailing punctuation -->

# Windlass Inc.

<!-- markdownlint-enable MD026 -->

[![Contributor Covenant 3.0](https://img.shields.io/badge/Contributor%20Covenant-3.0-4baaaa.svg)](https://github.com/windlasstech/.github/blob/main/CODE_OF_CONDUCT.md)
[![Website](https://img.shields.io/website?logo=data:image/svg%2bxml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCAxNjAgMTYwIiB3aWR0aD0iMTAwJSIgaGVpZ2h0PSIxMDAlIj4KICA8ZGVmcz4KICAgIDxsaW5lYXJHcmFkaWVudCBpZD0iYnJpZ2h0TmV0d29ya0dyYWQiIHgxPSIwJSIgeTE9IjEwMCUiIHgyPSIxMDAlIiB5Mj0iMCUiPgogICAgICA8c3RvcCBvZmZzZXQ9IjAlIiBzdG9wLWNvbG9yPSIjNjZBNkZGIi8+CiAgICAgIDxzdG9wIG9mZnNldD0iMTAwJSIgc3RvcC1jb2xvcj0iIzg5RjdGRSIvPgogICAgPC9saW5lYXJHcmFkaWVudD4KICA8L2RlZnM+CgogIDxnIHRyYW5zZm9ybT0idHJhbnNsYXRlKDMzLCAzMCkgc2tld1goLTIwKSIgZmlsbD0idXJsKCNicmlnaHROZXR3b3JrR3JhZCkiPgogICAgPHJlY3QgeD0iMCIgeT0iMjAiIHdpZHRoPSIyMCIgaGVpZ2h0PSI1NSIgLz4KICAgIDxyZWN0IHg9IjI3IiB5PSI0MCIgd2lkdGg9IjIwIiBoZWlnaHQ9IjQ1IiAvPgogICAgPHJlY3QgeD0iNTQiIHk9IjAiIHdpZHRoPSIyMCIgaGVpZ2h0PSIxMDAiIC8+CiAgICA8cmVjdCB4PSI4MSIgeT0iNDAiIHdpZHRoPSIyMCIgaGVpZ2h0PSI0NSIgLz4KICAgIDxyZWN0IHg9IjEwOCIgeT0iMjAiIHdpZHRoPSIyMCIgaGVpZ2h0PSI1NSIgLz4KICA8L2c+Cjwvc3ZnPgo=&up_color=2b6cd4&url=https%3A%2F%2Fwindlasstech.com&style=flat-square&label=windlasstech.com)](https://windlasstech.com/)
[![Bluesky](https://img.shields.io/badge/-windlasstech.com-1185FE?style=flat-square&logo=bluesky&logoColor=white)](https://bsky.app/profile/windlasstech.com)
[![X](https://img.shields.io/badge/-%40Windlass__inc-000000?style=flat-square&logo=x&logoColor=white)](https://x.com/Windlass_inc)

</div>

Windlass builds open standards, trusted distribution, and developer tooling for agentic software. We
provide supply chain infrastructure for agentic software: shared packaging, identity, and provenance
so developers can inspect, verify, and reuse agent components across runtimes without platform
lock-in.

## Projects

| Project                                                                                  | Description                                                                                              | Status                                                                                                                                                                                                                                                                                                                                                                                                               |
| :--------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [Agent Volumes](https://github.com/agent-volumes/agent-volumes-spec)                     | An open standard for packaging and distributing agent components                                         | [![Agent Volumes spec release](https://img.shields.io/github/v/release/agent-volumes/agent-volumes-spec?include_prereleases&sort=semver)](https://github.com/agent-volumes/agent-volumes-spec/releases)<br>[![GitHub Release Date](https://img.shields.io/github/release-date-pre/agent-volumes/agent-volumes-spec)](https://github.com/agent-volumes/agent-volumes-spec/releases)                                   |
| Windlass Tabulane                                                                        | Team distribution and change management for agent components                                             | In Development                                                                                                                                                                                                                                                                                                                                                                                                       |
| [shelf](https://github.com/windlasstech/shelf)                                           | Install, publish, and verify Agent Volumes from the CLI                                                  | In Development                                                                                                                                                                                                                                                                                                                                                                                                       |
| [vers-js](https://github.com/windlasstech/vers-js)                                       | TypeScript library to parse, validate, and canonicalize VERS declarations, with no runtime dependencies  | [![@windlass/vers-js NPM Version](https://img.shields.io/npm/v/@windlass/vers-js)](https://www.npmjs.com/package/@windlass/vers-js)<br>[![@windlass/vers-js NPM Last Update](https://img.shields.io/npm/last-update/%40windlass%2Fvers-js)](https://www.npmjs.com/package/@windlass/vers-js)                                                                                                                         |
| [actionlint-hardened-action](https://github.com/windlasstech/actionlint-hardened-action) | Reusable GitHub Action that runs actionlint in a digest-pinned container image                           | [![actionlint-hardened-action GitHub Release](https://img.shields.io/github/v/release/windlasstech/actionlint-hardened-action)](https://github.com/windlasstech/actionlint-hardened-action/releases)<br>[![actionlint-hardened-action GitHub Release Date](https://img.shields.io/github/release-date/windlasstech/actionlint-hardened-action)](https://github.com/windlasstech/actionlint-hardened-action/releases) |
| [slsa-builder](https://github.com/windlasstech/slsa-builder)                             | Reusable provenance-builder foundation targeting SLSA Build L3+ (pre-release; not a certification claim) | Experimental                                                                                                                                                                                                                                                                                                                                                                                                         |
| [Shared guidelines & security workflows](https://github.com/windlasstech/.github)        | Company policy documents & Reusable OpenSSF Scorecard, OSV Scanner, and dependency-review workflows      | Current                                                                                                                                                                                                                                                                                                                                                                                                              |

> [!NOTE]  
> [Agent Volumes](https://github.com/agent-volumes/agent-volumes-spec) is an independent,
> vendor-neutral open standard originally developed and released by Windlass. Implementations and
> contributions from other vendors and developers are welcome.

## Community

- [Website](https://windlasstech.com/)
- [Bluesky (@windlasstech.com)](https://bsky.app/profile/windlasstech.com)
- [X (@Windlass_inc)](https://x.com/Windlass_inc)
- [Code of Conduct (Contributor Covenant 3.0)](https://github.com/windlasstech/.github/blob/main/CODE_OF_CONDUCT.md)
- [Contributing guidelines](https://github.com/windlasstech/.github/blob/main/CONTRIBUTING.md)
- [Security policy (vulnerability reporting, SLSA, supply chain integrity)](https://github.com/windlasstech/.github/blob/main/SECURITY.md)
