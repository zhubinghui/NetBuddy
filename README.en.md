# NetCare

[简体中文](README.md) | **English**

> An AI agent that cares for your network.

NetCare is an AI agent harness for network operations. It operates network devices safely and reliably, performs offline fault diagnosis, and continuously evolves as an intelligent O&M tool.

## Status

**Early development** — architecture design is under discussion. Contributions are welcome.

## Vision

- **Online fault diagnosis**: Connects to the live network and, based on the provided network topology (auto-discovery is also supported), automatically collects and analyzes logs and provides troubleshooting recommendations. Following the human-in-the-loop principle, it can make configuration adjustments once fully authorized. The entire diagnosis process is controllable, manageable, and traceable. After diagnosis is complete, all diagnostic operations can be cleaned up.
- **Offline fault diagnosis**: Given fault-related logs, diagnostic logs, topology, and other information offline, it automatically performs causal inference on the root cause and provides the root cause or recommendations for further action.
- **Inspection**: Based on a given network topology and the devices to be inspected, it automatically logs in to devices and, once fully authorized, inspects them for hidden risks.
- **Change assurance**: Leveraging multi-dimensional digital-twin capabilities, it simulates the risks of change operations, provides deterministic predictions of operation outcomes, and supports rollback of all operations.

## Architecture

TBD (under discussion).

## Quick Start

TBD.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

[Apache-2.0](LICENSE)
