# RetroNet

**"The old Internet - without surveillance and corporations."**

RetroNet is a decentralized, Usenet-like communication and file-sharing network built on the [Reticulum Network Stack](https://reticulum.network/). No ISPs, no central servers, no external indexers.

## Features

- **Chunk-based File Sharing** - Binary-native file transfer with self-describing manifests, no yEnc, no PAR2, no NZB
- **`retronet://` URLs** - Magnet-link style addressing for files and peers
- **LXMF Integration** - Messages propagate via LXMF Propagation Nodes with native store-and-forward
- **NomadNet Bulletin Boards** - Usenet-style newsgroups backed by the existing Reticulum ecosystem
- **Web UI** - Browser-based newsreader and node dashboard (Svelte + Vite, embedded in the binary)
- **Interactive TUI** - Browse groups, read threads, compose messages from the terminal
- **Multiple Transports** - LoRa (5-50 km, no ISP), Packet Radio, TCP/IP, Serial
- **End-to-End Encrypted** - Ed25519/X25519 key pairs, addresses derived from public keys
- **Decentralized Search** - Local index + dedicated Index Nodes, no account required
- **Single Binary** - Cross-compiles for Raspberry Pi, LoRa hardware, and more

## Architecture

```
┌─────────────────────────────────────────────────────┐
│  RetroNet App Layer                                 │
│  Web UI │ CLI │ Chunk System │ Manifests │ Search   │
├─────────────────────────────────────────────────────┤
│  LXMF                                               │
│  Messages │ Propagation Nodes │ Store & Forward     │
├─────────────────────────────────────────────────────┤
│  NomadNet                                           │
│  Bulletin Boards │ Micron Pages │ Page Server       │
├─────────────────────────────────────────────────────┤
│  Reticulum (rnsd)                                   │
│  Routing │ E2E Encryption │ Transport               │
├─────────────────────────────────────────────────────┤
│  Physical Medium                                    │
│  TCP/IP │ LoRa (868 MHz) │ Packet Radio │ Serial    │
└─────────────────────────────────────────────────────┘
```

RetroNet builds only what doesn't exist in the Reticulum ecosystem:

- **Chunk system + manifests** -- binary-native file transfer
- **Web UI** -- browser-based newsreader
- **Decentralized search + Index Nodes** -- NZB replacement without accounts

Everything else (gossip, transport, store-and-forward, routing) is handled natively by Reticulum, LXMF, and NomadNet.

## Quickstart

### Docker

```bash
git clone https://github.com/retronet-hq/retronet.git
cd retronet
docker compose up -d
```

This starts a transport node on port 4242. Configuration in `deploy/config.toml`.

### Manual

```bash
# Prerequisites: Go 1.22+, Python 3 with rns and lxmf
pip install rns lxmf nomadnet

# Build
git clone https://github.com/retronet-hq/retronet.git
cd retronet
go build -o retronet ./cmd/retronet/

# Start a node
./retronet node start

# Create a group and post
./retronet groups create retronet.test.hello --type open --description "Test group"
./retronet broadcast --group retronet.test.hello --subject "Hello" --body "First post!"

# Interactive newsreader
./retronet tui

# Upload a file
./retronet upload myfile.zip --group retronet.test.hello

# Download via manifest hash or retronet:// URL
./retronet download retronet://sha256:a3f8c2d9...
```

## CLI

```
retronet
├── node          start, stop, status
├── groups        list, create, subscribe, search
├── read          <group> [--thread <id>]
├── post          <group> --subject --body
├── reply         <message-id>
├── upload        <file> --group <group>
├── download      <hash|retronet://-url> [--peer <addr>]
├── search        <query> [--group] [--index <addr>]
├── manifest      show <hash>
└── identity      show, export, import
```

## Project Structure

```
retronet/
├── cmd/retronet/main.go
├── cli/                  CLI commands (Cobra)
├── internal/
│   ├── bootstrap/        Peer discovery, DNS seeds, PEX
│   ├── chunk/            256 KB chunking engine, storage, LRU eviction
│   ├── chunkgossip/      Chunk propagation between nodes
│   ├── config/           TOML configuration
│   ├── gossip/           Message propagation (SSB/EBT)
│   ├── identity/         Ed25519 key management, addresses
│   ├── manifest/         Manifest format, signing, retronet:// URLs
│   ├── message/          Message format, signing, threading
│   ├── newsgroup/        Group model, announce, discovery
│   ├── node/             Node lifecycle, orchestration
│   ├── packet/           Network packet envelope, routing
│   ├── revocation/       Key revocation
│   ├── rns/              rnsd/lxmd sidecar interface (JSON-RPC bridge)
│   ├── storage/          SQLite (CGO-free via modernc.org/sqlite)
│   ├── transfer/         Chunk transfer protocol, client + server
│   └── tui/              Interactive terminal UI (Bubble Tea)
├── pkg/retronet/         Public API
├── docs/                 Documentation (MkDocs)
├── deploy/               Docker Compose + config
└── examples/             Example configurations
```

## Roadmap

### Current: Phase 1 — Python Bridge

- [] RetroNet communicates with Reticulum via a Python sidecar process (`rnsd` + `rns_bridge.py`) over JSON-RPC on a Unix socket. This requires Python 3 with `rns` installed.

### Planned: Phase 2 — Native Go Integration

- [] A Go implementation of Reticulum exists at [`Reticulum-Go`](https://pkg.go.dev/git.quad4.io/Networks/Reticulum-Go) (v0.6.0). Once it reaches a stable release, RetroNet can drop the Python dependency entirely - resulting in a single, self-contained Go binary with zero external dependencies.

## Documentation

- [Architecture](docs/architecture.md) - Layer model, components, data flow
- [Protocol](docs/protocol.md) - Wire formats, message types, chunk transfer
- [Setup Guide](docs/setup.md) - Installation, first node, LoRa configuration
- [Bootstrap](docs/bootstrap.md) - Peer discovery, DNS seeds, PEX
- [Gossip Design](docs/gossip-design.md) - SSB/EBT analysis and hybrid gossip model
- [Reticulum Internals](docs/rns-internals.md) - Transport node deep-dive
- [Contributing](docs/contributing.md) - Development setup, conventions, how to help

## Development

```bash
# Run tests
go test ./...

# Lint
golangci-lint run

# Build docs
pip install mkdocs-material
mkdocs serve
```

## License

AGPL-3.0 — see [LICENSE](LICENSE)
