# vm-bootstrap

One `mise.toml` that turns a fresh Linux VM into an **Orca headless server** (paired over LAN
and Tailscale) plus **oh-my-pi (`omp`)** searching through a private **SearXNG** instance.

```sh
# 1. mise:  curl https://mise.run | sh   (then activate it in your shell rc)
mkdir -p ~/vm-bootstrap && cd ~/vm-bootstrap
for f in mise.toml AGENTS.md CLAUDE.md; do curl -fsSLO "https://gist.githubusercontent.com/twick00/6a7c3afaf0f906121fb62ced13d0cb84/raw/$f"; done
printf '[env]\nSEARXNG_URL = "http://<searxng-host>:8888"\n' > mise.local.toml   # host-specific, never published
mise trust && mise install && mise run setup
```

`mise run setup` installs the tools, runs `orca-ide serve` as a `systemd --user` service,
prints four pairing codes (LAN/Tailscale × desktop/mobile), points omp at SearXNG, and
leaves a note for coding agents. See `mise tasks` for the individual steps and `AGENTS.md`
for how agents should evolve this file. No secrets or private addresses live here — see
`mise.local.toml`.
