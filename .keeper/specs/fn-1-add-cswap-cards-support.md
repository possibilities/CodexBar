## Overview

Extend the one-shot `codexbar cards` terminal UI to render multiple claude-swap accounts with the same `accountCount > 1` precedence as the menu-bar GUI. The implementation stays cards-only and read-only: it reuses the existing Core reader/parser/projection, preserves explicit CLI intent, keeps `usage` and `serve` unchanged, and reports adapter failures without discarding useful ambient Claude output.

## Quick commands

- `swift test --filter CLICardsClaudeSwapTests`
- `swift test --filter CLICardsRendererTests`
- `make test`
- `make check`

## Acceptance

- [ ] With claude-swap enabled and more than one parsed account, `codexbar cards` renders every account active-first and replaces the ambient/token-account Claude presentation without changing other providers.
- [ ] Zero or one parsed account, disabled integration, explicit account selection, and explicit non-auto source selection preserve existing cards behavior and do not alter `codexbar usage` or `serve`.
- [ ] A blank configured executable path or adapter-level failure preserves ambient Claude output, adds a distinct claude-swap failure diagnostic, and returns nonzero; valid per-account sentinel states remain successful account cards.
- [ ] Full and brief layouts expose active state, no-usage/sentinel reasons, and terminal-safe display text without a misleading claude-swap plan badge.
- [ ] Automated macOS/Linux CLI tests, the full test suite, formatting, and strict lint checks pass without invoking real provider credentials, Keychain, or a live claude-swap installation.

## Early proof point

Task that proves the approach: task 1. If it fails: keep the integration at the cards command boundary and simplify concurrency while preserving the same result-combination contract.

## References

- `Sources/CodexBarCLI/CLICardsCommand.swift:68` — cards-only orchestration boundary.
- `Sources/CodexBarCore/Providers/Claude/ClaudeSwap/ClaudeSwapAccountProjection.swift:3` — reusable active-first provider-neutral projection.
- `Sources/CodexBar/StatusItemController+AccountMenuDisplay.swift:4` — GUI `accountCount > 1` precedence.
- `docs/claude.md:116` — existing claude-swap display and failure-isolation contract.

## Docs gaps

- **`docs/cli.md`**: document automatic multi-account cards, precedence, explicit overrides, and adapter-failure behavior.
- **`docs/claude.md`**: extend claude-swap display/precedence guidance to the cards terminal UI while keeping the scope cards-only.
- **`CHANGELOG.md`**: add an Unreleased entry for claude-swap multi-account terminal cards.

## Best practices

- **Coherent snapshots:** enumerate once and project all rows from the same versioned claude-swap snapshot to avoid active/usage races. [claude-swap snapshot source](https://github.com/realiti4/claude-swap/blob/main/src/claude_swap/snapshot_source.py)
- **Explicit intent wins:** account selectors and explicit source flags bypass automatic discovery so scripts retain their established meaning. [AWS CLI precedence](https://docs.aws.amazon.com/cli/latest/topic/config-vars.html)
- **Terminal-safe external text:** strip control sequences, normalize multiline diagnostics, and bound externally sourced labels/errors before rendering. [OWASP Logging Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html)
- **Drained concurrency:** if ambient usage and adapter enumeration run concurrently, collect both child outcomes explicitly so optional adapter failure cannot destabilize the required ambient path.
