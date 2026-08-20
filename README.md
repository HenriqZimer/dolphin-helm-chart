# Dolphin Helm Chart Repository

<p align="center">
  <img src="https://raw.githubusercontent.com/linuxserver/docker-templates/master/linuxserver.io/img/dolphin-logo.png" alt="Dolphin logo" width="140" />
</p>

[![License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Helm Version](https://img.shields.io/badge/Helm-v3-blue)](https://helm.sh)
[![Artifact Hub](https://img.shields.io/endpoint?url=https://artifacthub.io/badge/repository/dolphin-helm-chart)](https://artifacthub.io/packages/search?repo=dolphin-helm-chart)

This repository contains a production-ready Helm chart for deploying [Dolphin](https://docs.linuxserver.io/images/docker-dolphin/) on Kubernetes.

## About Dolphin

Dolphin is a free and open-source GameCube/Wii emulator. This chart deploys the
[linuxserver.io](https://docs.linuxserver.io/images/docker-dolphin/) build, which serves the full
emulator desktop in your browser over KasmVNC — no local install needed, play from any device on
your network:

- 🎮 Full GameCube/Wii emulator desktop, streamed over the browser (KasmVNC)
- 🖥️ No client install — works from any modern browser
- 🗄️ Config/save persistence via a mounted `/config` volume
- 🎯 Bring your own ROMs/ISO library via `extraVolumes`
- ⚡ Optional VA-API GPU passthrough for hardware-accelerated rendering

## Quick Start

### Add Helm Repository

```bash
helm repo add dolphin-helm-chart https://henriqzimer.github.io/dolphin-helm-chart/
helm repo update
```

### Install Chart

```bash
helm install my-dolphin dolphin-helm-chart/dolphin --version 1.2.5
```

For detailed installation instructions and configuration options, see the [chart README](chart/README.md).

## Repository Structure

```
.
├── chart/              # Helm chart for Dolphin
│   ├── Chart.yaml      # Chart metadata
│   ├── values.yaml     # Default configuration values
│   ├── README.md       # Detailed chart documentation
│   └── templates/      # Kubernetes manifest templates
├── LICENSE             # Repository license
└── README.md           # This file
```

## Documentation

- **[Chart Documentation](chart/README.md)** - Complete installation and configuration guide
- **[linuxserver.io Dolphin Docs](https://docs.linuxserver.io/images/docker-dolphin/)** - Upstream image documentation
- **[Values Reference](chart/values.yaml)** - All available configuration options

## Prerequisites

- Kubernetes 1.19+
- Helm 3.0+
- Persistent storage, if you enable `persistence.config` (NFS, local-path, or cloud storage)
- (Optional) Ingress controller
- (Optional) cert-manager for automatic TLS
- (Optional) A node exposing `/dev/dri` for GPU passthrough

## Features

This Helm chart provides:

- ✅ Production-ready Kubernetes Deployment/Service
- ✅ Optional persistent `/config` volume (settings, saves)
- ✅ `extraVolumes`/`extraVolumeMounts` for a ROMs/ISO library
- ✅ Optional Ingress with TLS support
- ✅ Resource limits and requests
- ✅ Readiness/liveness probes
- ✅ Configurable ServiceAccount
- ✅ Optional VA-API (`/dev/dri`) GPU passthrough

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request. See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

This Helm chart is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

Dolphin itself is licensed under its own terms. See the [Dolphin project](https://dolphin-emu.org/) and the
[linuxserver.io image](https://github.com/linuxserver/docker-dolphin) for more information.

## Support

- 🐛 [Report Issues](https://github.com/HenriqZimer/dolphin-helm-chart/issues)
- 💬 [Discussions](https://github.com/HenriqZimer/dolphin-helm-chart/discussions)
- 📖 [Documentation](chart/README.md)

---

Made with ❤️ for the retro gaming community
