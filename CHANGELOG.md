# Changelog

All notable changes to this deployment definition are documented in this file.
The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [Unreleased]

### Changed

- Upstream dashboard image tracks NetBird dashboard v2.92.0 and server image tracks netbird-server 0.79.0 as defined in `service.yml`.

## [0.0.1]

### Added

- Initial App-as-Code service definition for NetBird (dashboard plus combined management, signal, relay and STUN server).
- Traefik routing for the dashboard and gRPC/HTTP routes of the server, with automatic DNS.
- README with configuration reference and project icon.
