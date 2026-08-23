# Changelog

All notable changes to this Helm chart will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [1.5.3] - 2026-08-23

### Changed
- `livenessProbe.initialDelaySeconds` 90s->120s and `failureThreshold` explicitly set to `6` (was the implicit default `3`) - confirmed in production that under heavy I/O contention from a shared NFS-backed PVC (several emulator pods cold-booting simultaneously after a node reboot), the old 150s total liveness tolerance wasn't enough for DOCKER_MODS's ephemeral apt install + KasmVNC desktop startup to finish; kubelet kept killing the container mid-boot, and each kill wiped the ephemeral DOCKER_MODS progress, creating a self-sustaining restart loop that never converged even as node load dropped. New tolerance: 120 + 6*20 = 240s.

## [1.5.2] - 2026-08-23

### Changed
- `readinessProbe.initialDelaySeconds` 10s->30s, `livenessProbe.initialDelaySeconds` 30s->90s, and `resources.requests.cpu` 500m->1 - confirmed in production that under CPU contention on a shared GPU node, the old tight timing killed containers mid-boot (KasmVNC desktop hadn't finished starting) causing a self-sustaining restart storm; the higher CPU request also gives the scheduler an honest signal instead of letting far more pods pack onto a node than it can actually sustain under load.

## [1.4.1] - 2026-08-21

### Fixed
- `sunshine.*` now also mounts `/dev/uhid` - without it, Sunshine falls back to emulating gamepads as a generic Xbox One controller when the client reports a PlayStation-type pad (DualShock/DualSense), and that fallback does not correctly forward face buttons/triggers (confirmed: analog stick axes work, buttons don't). The real virtual DualShock/DualSense device needs kernel `uhid` access to be created.

## [1.4.0] - 2026-08-21

### Changed
- **BREAKING**: `sunshine.ports.*` (8 independent fields) replaced by a single `sunshine.port` (default `47989`). Sunshine only accepts one configurable base port ("port" in `sunshine.conf`) - every other port it listens on is a fixed offset from it, calculated by its own `net::map_port()`. The old `ports.*` fields only ever affected the Deployment's `containerPort` entries (documentation/hostPort binding) - Sunshine itself always listened on its hardcoded defaults regardless, so setting them to anything else silently did nothing. Also required a matching `dolphin-sunshine-mod` update to write `port = $SUNSHINE_PORT` into `sunshine.conf` - confirmed broken end-to-end on the real cluster (both dolphin and azahar needing hostNetwork on the same GPU node) before this fix.

## [1.3.1] - 2026-08-20

### Fixed
- `replicaCount` schema now allows `0` (pause the emulator, e.g. to free node resources temporarily) in addition to `1` - previously restricted to exactly `1`, so `0` was rejected outright.

## [1.3.0] - 2026-08-20

### Added
- `sunshine.*` (POC) - runs Sunshine alongside the default Selkies/KasmVNC streaming for lower-latency Moonlight access, requiring a Docker Mod that installs Sunshine and swaps the base image's Xvfb (not linked against libudev, so it never hotplugs Sunshine's uinput-created input devices) for a real Xorg + "dummy" driver. Needs `hostNetwork: true` (Moonlight is raw TCP/UDP, can't go through the Traefik Ingress, and the pod needs the host's network namespace for uinput hotplug uevents to reach udev) and `privileged: true` (no native Kubernetes knob for the device cgroup rule the pod needs to open `/dev/input/eventN` nodes created at runtime). See the mod's own README for the full list of issues found and fixed getting input to work.

## [1.2.5] - 2026-08-17

### Fixed
- README install commands now use a unique repo alias (`dolphin-helm-chart`) instead of the bare chart name, and pin `--version` explicitly.

## [1.2.4] - 2026-08-17

### Changed
- README now shows the emulator's logo at the top.

## [1.2.3] - 2026-08-17

### Added
- Chart releases are now GPG-signed (`helm package --sign`) - see `artifacthub.io/signKey` in `Chart.yaml` for the public key URL and fingerprint. Powers the "Signed" badge on ArtifactHub.

## [1.2.2] - 2026-08-16

### Added
- `chart/values.schema.json` validating `values.yaml` - powers the "Values schema" feature on ArtifactHub, previously absent since no chart in this project ever had one.

## [1.2.1] - 2026-08-16

### Added
- `icon` in Chart.yaml, pointing at linuxserver.io's own logo image for this app - was missing entirely before, which is why no image ever showed up on ArtifactHub for this chart.

## [1.2.0] - 2026-08-15

### Added
- `gpu.vendor: nvidia` mode (alongside the existing `intel-amd` VA-API mode), requesting `nvidia.com/gpu`
  via the NVIDIA Kubernetes device plugin and setting `NVIDIA_VISIBLE_DEVICES`/`NVIDIA_DRIVER_CAPABILITIES`.
  Documented the `nvidia-drm.modeset=1` kernel parameter (and, on headless nodes, a dummy HDMI/DP plug)
  needed on the node for the compositor to render anything instead of a solid black desktop.
- `seccompUnconfined` to work around a JIT recompiler crashing with `SIGBUS` under the default seccomp
  profile on some kernel/libseccomp combinations.

### Changed
- Bumped the default `shmSize` from `1Gi` to `2Gi`, matching the fix confirmed on this chart's sibling
  `pcsx2-helm-chart` for a JIT recompiler crashing with `SIGBUS` under Docker/Kubernetes' small default
  `/dev/shm`.

## [1.1.1] - 2026-08-12

### Changed
- Clarified the streaming README section: `broker_host` is server-to-server (cluster-internal DNS is fine, keep it off any externally-reachable Ingress/LoadBalancer), while `host` is opened directly by the end user's browser and needs real external reachability + TLS. Documented that the container's self-signed cert has no SAN, which some browsers (notably Safari) refuse outright instead of showing the usual clickthrough warning.

## [1.1.0] - 2026-08-12

### Added
- `streaming.enabled`/`streaming.brokerPort` to expose the RomM emulator-streaming broker sidecar port. The chart doesn't install the broker itself — pair with `env.DOCKER_MODS: ghcr.io/loneangelfayt/dolphin-romm-integration-mod:latest` and mount your ROMs library at the same path RomM uses. See [Emulator Streaming](https://docs.romm.app/latest/using/emulator-streaming/).

## [1.0.0] - 2026-08-12

### Added
- Initial release of the Dolphin Helm chart
- Deployment, Service, optional Ingress and PVC for the linuxserver.io Dolphin KasmVNC webtop image
- Configurable `serviceAccount` (create/name/annotations)
- Readiness and liveness probes (TCP check on the KasmVNC HTTP port)
- Optional VA-API (`/dev/dri`) GPU passthrough via `gpu.enabled`
- `extraVolumes`/`extraVolumeMounts` for mounting an existing ROMs/BIOS library
