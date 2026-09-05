# AGENTS.md — vm-bootstrap

This directory was bootstrapped from a **public** gist:
https://gist.github.com/twick00/6a7c3afaf0f906121fb62ced13d0cb84 (mirror: https://github.com/PlaneshiftDev/vm-bootstrap).
`mise.toml` there is the single source of truth for how these VMs are set up. It is meant
to **evolve**: when you learn something on this host that would help the next VM, send it
back upstream.

## What lives where

| Thing | Where | Published? |
|---|---|---|
| Tool versions, tasks, generic setup logic | `mise.toml` | yes (gist + mirror) |
| Agent guidance (this file), `CLAUDE.md`, `README.md` | same dir | yes |
| Host-specific values: `SEARXNG_URL`, anything with an IP/hostname/token | `mise.local.toml` | **never** |
| Runtime state (systemd unit, `~/.omp/agent/*`, Orca profile) | `$HOME` | no |

`mise.local.toml` is auto-loaded by mise and overrides `mise.toml`. If a value differs per host,
it belongs there — the public file only documents *how to obtain it*.

## When to propose a gist update

Ask the user (show the diff, then run `mise run publish "<message>"` on approval) whenever you:

- pin or bump a tool version (`oh-my-pi`, `github:stablyai/orca`, a new runtime) — "latest" is
  the default, but once a specific version matters, pin it here so every VM agrees;
- add a dependency every VM will need;
- fix or harden a task (a flag that changed, a distro quirk, a race);
- change how pairing codes, the service unit, or omp config are generated.

Do **not** upstream: one-off workarounds specific to this machine, anything under `[env]` that
came from `mise.local.toml`, or LLM provider configuration for omp (the operator chooses that
per host and it must stay out of the gist).

## Before every publish — scrub

The gist is public. `mise run publish` depends on `mise run scan-secrets`, but the scan is a
regex net, not a guarantee. Read the diff yourself and remove:

- private IPs (10/8, 172.16/12, 192.168/16, Tailscale 100.64/10), MagicDNS names (the
  Tailscale `ts` dot `net` domain), LAN hostnames;
- Orca pairing URLs/codes (`orca:` scheme), API keys, tokens, long base64 blobs;
- usernames, home paths (`/home/<user>`), email addresses;
- anything the operator handed you as "the value for X".

Replace with a placeholder and a one-line note on how to get the real value. If the scan
flags a false positive, restructure the text rather than weakening the pattern.

## Conventions

- Keep `mise.toml` light and readable; tasks are bash with `set -euo pipefail`.
- Every task is idempotent — `mise run setup` must be safe to re-run on a configured host.
- Never configure an omp model provider unless the operator asks; if one exists, report it.
- Pairing codes: always all four (LAN/Tailscale × desktop/mobile). Orca is single-instance per
  profile, so `orca-pair` stops the service while generating and restarts it on exit.
- `mise run pull` refreshes the public files from the gist without touching `mise.local.toml`.
