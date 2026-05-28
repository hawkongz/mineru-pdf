# Security Policy

## Reporting a Vulnerability

If you discover a security issue in this skill, please report it by [opening an issue](https://github.com/20kiki/mineru-pdf/issues) with the `security` label.

**Do not** include sensitive details in public issues. For critical vulnerabilities, contact the maintainer directly via GitHub.

## Scope

This repository is a Claude Code skill — a Markdown file with instructions for AI-assisted PDF extraction. The security surface is minimal:

- No bundled executables
- No network services
- No authentication or user data handling

The underlying [MinerU](https://github.com/opendatalab/MinerU) engine handles actual PDF processing. For MinerU-specific security issues, report to the [MinerU repository](https://github.com/opendatalab/MinerU).

## Supported Versions

| Version | Supported |
|:---|:---|
| Latest `master` | ✅ |
| Previous releases | ❌ |
