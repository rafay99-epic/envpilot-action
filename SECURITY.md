# Security Policy

## Reporting

Email **security@envpilot.dev**. Do not open public issues for security vulnerabilities.

## Design

- This action never logs or prints your token or variable values.
- All fetched values are registered with `core.setSecret` before export, so accidental echoes show `***`.
- Dotenv files are written with mode `0600` (owner read/write only).
- Tokens are read-only and scoped per project + environment.
