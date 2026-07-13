## Description

**Size:** M
**Files:** Sources/CodexBarCLI/CLICardsCommand.swift, Sources/CodexBarCLI/CLIClaudeSwapCards.swift, Sources/CodexBarCLI/CLICardsRenderer.swift, Sources/CodexBarCLI/CLICardsBriefRenderer.swift, Sources/CodexBarCLI/CLIHelp.swift, Tests/CodexBarTests/CLICardsClaudeSwapTests.swift, Tests/CodexBarTests/CLICardsRendererTests.swift, TestsLinux/CLICardsClaudeSwapTests.swift, TestsLinux/CLICardsRendererTests.swift, docs/cli.md, docs/claude.md, CHANGELOG.md

### Approach

Add a cards-private coordinator that is eligible only for Claude when claude-swap is enabled and no explicit account selector or non-auto source override is present. On every supported CLI platform, including an explicit `--provider claude` request, concurrently collect the normal Claude result and one bounded `ClaudeSwapAccountReader.readAccountList` result using a fully drained nonthrowing task group; project the list through `ClaudeSwapAccountProjection` and apply the GUI's `accountCount > 1` precedence atomically. A valid multi-account result replaces the complete ambient Claude result, including hidden ambient failures; zero/one account retains it. Blank executable paths and adapter-level failures merge a single-line, bounded `Claude (claude-swap)` footer after any ambient `Claude` failure and preserve nonzero status, while valid sentinel rows remain successful metrics-less cards.

Extend the existing card model rather than adding another renderer: carry active and account-problem state into both grid and brief projections, suppress the inferred `PLAN Claude-Swap` badge, and sanitize cswap-derived labels/errors by removing terminal controls, normalizing CR/LF to spaces, and bounding labels to 256 and diagnostics to 512 Unicode scalar values before rendering. Full cards append `[active]` to the account presentation and show canonical sentinel text inline; brief rows retain `[active]`, place sentinel text in the Usage column, and use an em dash where no reset exists. Repeat provider status consistently with existing multi-account card behavior. Never call the mutating `switchAccount` API, log/persist raw account output, or use email as stable identity.

### Investigation targets

*Verify before relying — these file:line refs are planner-verified at authoring time, but the repo moves.*

**Required** (read before coding):
- Sources/CodexBarCLI/CLICardsCommand.swift:68 — command-specific config, explicit-option parsing, provider loop, result aggregation, rendering, and exit status.
- Sources/CodexBarCLI/CLIUsageCommand.swift:193 — existing ambient/token-account fetch behavior that cards calls but `usage` and `serve` also share.
- Sources/CodexBarCLI/CLICardsRenderer.swift:28 — card model, build input, account/plan/source projection, grid layout, and failure footer.
- Sources/CodexBarCLI/CLICardsBriefRenderer.swift:36 — card-to-row projection that currently drops free-form sentinel information.
- Sources/CodexBarCore/Providers/Claude/ClaudeSwap/ClaudeSwapAccountReader.swift:18 — fixed read-only list invocation, timeout, and output bounds.
- Sources/CodexBarCore/Providers/Claude/ClaudeSwap/ClaudeSwapAccountProjection.swift:3 — canonical ordering, stable slot identity, optional usage, and sentinel wording.
- Sources/CodexBarCore/ProviderAccountSnapshot.swift:3 — provider-neutral active/optional-usage/account-error contract.

**Optional** (reference as needed):
- Sources/CodexBarCLI/TokenAccountCLI.swift:5 — `usesOverride` and existing account selector semantics.
- Sources/CodexBarCLI/CLIHelpers.swift:263 — explicit source parsing, including the `--web` alias.
- Sources/CodexBarCore/Config/CodexBarConfig.swift:109 — shared nullable enable/path configuration and sanitized path.
- Sources/CodexBar/StatusItemController+AccountMenuDisplay.swift:4 — GUI precedence to mirror without importing the macOS target.
- Tests/CodexBarTests/ClaudeSwapAccountReaderTests.swift:5 — fake-executable process-boundary test pattern.
- Tests/CodexBarTests/ClaudeSwapAccountProjectionTests.swift:6 — active-first, sentinel, and fallback-label expectations.
- Tests/CodexBarTests/CLICardsRendererTests.swift:6 — canonical macOS renderer suite mirrored in Linux.

### Risks

The 30-second Core timeout can delay one-shot output; retain the accepted bound initially and keep the reader injectable so a cards-specific bound can be introduced only with evidence. Concurrent result replacement must fully drain both children and suppress hidden ambient errors only after a valid multi-account snapshot wins. Nil-usage rows and long sentinel text can disappear or damage narrow layouts unless grid and brief behavior are independently pinned. The macOS/Linux renderer suites are duplicated and must remain synchronized.

### Test notes

Add a pure/injectable orchestration seam plus fake-executable coverage; never run a real cswap binary, provider probe, browser import, Keychain query, or credential read. Cover disabled integration, enabled blank path, 0/1/2+ accounts, no active account, active sentinel accounts, all known/unknown statuses, explicit account/source bypass with zero adapter invocations, multi-provider ordering, successful cswap with failed ambient fetch, adapter fallback, dual failures, cancellation, terminal-control sanitization, full/brief narrow layouts, repeated provider status, and proof that no switch command is issued. Keep macOS and Linux renderer expectations byte-aligned, then run focused tests followed by `make test` and `make check`.

## Acceptance

- [ ] Eligible `codexbar cards` requests render every row from a valid two-or-more-account claude-swap list in active-first/slot order, replace all ambient/token Claude cards and hidden ambient failures, preserve other-provider ordering, and invoke the list adapter exactly once.
- [ ] Disabled integration, valid zero/one-account lists, explicit account selectors, and explicit non-auto source flags preserve existing ambient behavior; bypass cases invoke no claude-swap subprocess, while `--source auto` and an explicitly selected Claude provider remain eligible.
- [ ] Enabled integration with a blank path or any reader/parser/timeout failure retains ambient output, appends a distinct `Claude (claude-swap)` footer after an ambient Claude failure when both fail, and returns nonzero without duplicate or ambiguous diagnostics.
- [ ] Valid API-key, expired, Keychain, re-login, unavailable, unknown, and no-window account rows count toward precedence, render canonical problem text without fabricated metrics in both grid and brief modes, and do not independently fail the command.
- [ ] Active state renders independently as `[active]` in full and brief account presentation, inactive/no-active lists remain unmarked, provider status follows existing multi-account behavior, and no card displays a claude-swap plan badge.
- [ ] Cswap-derived labels and diagnostics are single-line, terminal-control-free, bounded to the specified limits, ephemeral, and keyed internally only by stable numeric slot identity; no switching API or credential-bearing output is invoked, logged, or persisted.
- [ ] CLI help and user documentation describe cards-only precedence, explicit override behavior, sentinels, and adapter failure; `codexbar usage` and `serve` retain their existing output cardinality and semantics.
- [ ] Focused macOS/Linux tests, `make test`, and `make check` pass using only fakes and stubs with no live provider, browser, Keychain, or credential access.

## Done summary
Added a cards-only claude-swap multi-account coordinator (CLIClaudeSwapCards) that concurrently fetches ambient Claude output and the claude-swap account list, applies accountCount>1 precedence to replace ambient/token Claude cards in active-first order, and falls back to ambient output plus a distinct sanitized failure footer on blank path or adapter failure. Extended CLICardModel/brief rows with active-state and sanitized account-problem text, added mirrored macOS/Linux coverage, and updated CLI help and docs.
## Evidence
