# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- Complete project structure with 8 Ansible roles
- Docker setup automation for Ubuntu, Debian, and Fedora
- Standard (middle/guard) relay deployment
- Exit relay deployment with legal warnings
- SOCKS proxy deployment for local networks
- Comprehensive preflight checks
- Health monitoring and validation
- Nyx monitoring tool integration
- Security hardening (SSH, Fail2Ban, UFW)
- IPv6 support
- Multiple deployment playbooks
- Comprehensive documentation

### Changed
- N/A (Initial release)

### Deprecated
- N/A (Initial release)

### Removed
- N/A (Initial release)

### Fixed
- N/A (Initial release)

### Security
- N/A (Initial release)

## [1.0.0] - TBD

### Added
- Initial release of Ansible Anon Relay Deployment
- Full automation of Anyone Protocol relay nodes
- Multi-distribution support (Ubuntu, Debian, Fedora)
- Docker-based deployment
- Three relay types: Standard, Exit, SOCKS
- Comprehensive monitoring and health checks
- Security hardening by default
- Complete documentation

---

## Version History

### Version Numbering
- **MAJOR** version for incompatible API/playbook changes
- **MINOR** version for backwards-compatible functionality additions
- **PATCH** version for backwards-compatible bug fixes

### Release Process
1. Update CHANGELOG.md with version number and date
2. Update version in all role meta files
3. Tag release in Git: `git tag -a v1.0.0 -m "Release v1.0.0"`
4. Push tags: `git push origin --tags`
5. Create GitHub release with changelog excerpt

---

## [Unreleased] - Development Notes

### In Progress
- Final testing across all supported distributions
- Performance optimization
- Enhanced error handling
- Additional monitoring metrics

### Planned Features
- Molecule testing framework integration
- CI/CD pipeline with GitHub Actions
- Automated backup and restore functionality
- Prometheus/Grafana monitoring integration
- Multi-region deployment support
- Ansible Tower/AWX compatibility

### Known Issues
- None at this time

---

For migration guides and breaking changes, see [docs/MIGRATION.md](docs/MIGRATION.md) (when available).

For security vulnerabilities, see [SECURITY.md](SECURITY.md).
