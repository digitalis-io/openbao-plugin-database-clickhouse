# Changelog

All notable changes to this project are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Changed

- Container images now also publish a combined
  `{openbao-version}-{plugin-version}` tag (for example `2.4.4-1.2.3`, and
  `2.4.4-1.2.3-ubi` for the UBI flavour) so a deployment can pin both the
  OpenBao base version and the plugin version.
- `golangci-lint` is pinned to v2.13.1 in CI so a new linter release cannot
  fail a branch that changed nothing.
- Go toolchain bumped to 1.27.0 (`go.mod`, CI workflows and the container build)
  and all Go module dependencies updated to their latest releases, including
  `github.com/ClickHouse/clickhouse-go/v2` v2.48.0 and
  `github.com/openbao/openbao/sdk/v2` v2.6.2.
- Source modernised with `go fix`: `interface{}` replaced by `any`. No
  behaviour change.

### Fixed

- `google.golang.org/grpc` updated to v1.83.2 to pick up the fix for
  CVE-2026-84445, a denial of service in gRPC-Go xDS servers where a request
  missing both the `:authority` and `Host` headers caused an out-of-bounds
  panic. The plugin does not run an xDS server, so it was not exploitable here,
  but the vulnerable version was still present in the dependency tree.
  `golang.org/x/crypto` (v0.57.0), `golang.org/x/net` (v0.59.0),
  `golang.org/x/sys` (v0.48.0) and `golang.org/x/text` (v0.42.0) were updated in
  the same pass.
- `testhelpers`: `BuildConnString` built the host with `string(rune(port))`,
  which produced a garbage host instead of `host:port`. It now uses
  `net.JoinHostPort`.

### Added

- Container image bundling OpenBao with the ClickHouse database plugin
  preinstalled at `/openbao/plugins`, in alpine (`openbao/openbao`) and UBI
  (`openbao/openbao-ubi`) flavours.
- GitHub Actions `Docker` workflow that builds both flavours for
  `linux/amd64` and `linux/arm64` and publishes them to GHCR on every `v*`
  tag, with an SBOM, a build provenance attestation and a post-push smoke
  test. The plugin version is taken from the tag and the build fails if the
  tag is not valid semver.
- `docker-compose.yml` test stack: ClickHouse with SQL access management plus a
  dev-mode OpenBao, with the database secrets engine, a ClickHouse connection
  and a `readonly` role configured automatically.
- CI `e2e` job that brings up the `docker-compose.yml` stack, configures the
  secrets engine, issues a dynamic credential, authenticates to ClickHouse with
  it and verifies the user is dropped when the lease is revoked.
- Makefile targets `docker-build`, `docker-build-ubi`, `docker-buildx`,
  `docker-run`, `compose-up`, `compose-down`, `compose-logs` and
  `compose-test`.
