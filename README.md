# Sentrix

Sentrix is a distributed security operations platform designed to connect endpoint agents, centralized orchestration, and threat analytics into a cohesive operating model for modern SOC and XDR workflows.

It provides a practical foundation for deploying lightweight monitoring agents, coordinating service provisioning, and integrating threat intelligence and detection workflows in a containerized environment.

## Why this project exists

Security teams increasingly need a unified approach to collect telemetry, manage distributed sensors, and coordinate response actions across environments that are heterogeneous and operationally complex. Sentrix was designed to address that need by combining:

- a lightweight agent model for endpoint or host-based participation,
- a centralized control plane for session management and provisioning,
- a threat-processing layer for ingesting and correlating signals,
- and an operational stack based on containerized services for deployment flexibility.

The repository reflects an architecture that is intentionally modular: agents can be deployed to target systems, orchestration services can manage their registration and lifecycle, and detection services can ingest structured events for downstream analysis.

## Problem statement

Many organizations struggle with fragmented security tooling that is difficult to deploy, hard to operationalize, and expensive to scale. Sentrix aims to reduce that friction by offering an opinionated reference architecture for:

- distributed agent onboarding,
- centralized service association,
- event ingestion and correlation,
- role-aware access and operational oversight,
- and container-based deployment.

## Architecture at a glance

The platform is organized around four primary layers:

1. Agent layer — lightweight collectors and runtime components that register with the control plane.
2. Control plane — session management, authentication, role access, and provisioning orchestration.
3. Analytics and detection layer — threat-processing services that consume events and generate operational insights.
4. Data and observability layer — logs, dashboards, and supporting infrastructure for monitoring and investigation.

A simplified view looks like this:

```text
Clients / Agents
    ↓
Session Management & Provisioning
    ↓
Threat Processing & Detection
    ↓
Observability, Dashboards, and Investigations
```

## Core capabilities

- Distributed agent registration and lifecycle coordination
- Session-based service provisioning for connected hosts
- Role-based access controls for operators and analysts
- Integration with common security stack components such as Suricata, Wazuh, Elasticsearch, and Kibana
- Containerized deployment for rapid environment setup
- Threat event ingestion and processing through a dedicated engine layer
- Operational verification and health assessment for deployment readiness

## Technology stack

Sentrix is built around a modern, cloud-native security stack:

- Python for orchestration and service logic
- FastAPI for API endpoints and control-plane services
- Redis for session and state management
- Keycloak for identity and role-based authentication
- Docker and Docker Compose for deployment
- Elasticsearch, Kibana, and Filebeat for search and observability
- Suricata and Wazuh for security telemetry and detection integrations
- TheHive for case and incident-oriented workflows

## Repository structure

```text
Sentrix_SOC/
  agent/                # client-side runtime and orchestration logic
  cleint/               # agent deployment, registration, and container management
  connectors/           # integration adapters for external security systems
  docker/               # compose stack, session manager, and service definitions
  docs/                 # architecture references and documentation assets
  engines/              # threat engine and related execution components
  keycloak_bootstrap/   # identity bootstrap and role initialization
  libs/                 # shared libraries and support modules
  rules/                # detection content and rule assets
  tests/                # verification and regression tests
```

## Installation

### Prerequisites

- Docker Desktop or a compatible Docker environment
- Python 3.10+ (recommended)
- Network access to pull container images
- A host with sufficient resources for the monitoring and analytics stack

### Clone the repository

```bash
git clone <repository-url>
cd Sentrix_SOC
```

## Quick start

### 1. Start the platform services

```bash
cd docker
docker compose up -d
```

### 2. Access the main services

- Session manager: http://localhost:8000
- Keycloak: http://localhost:8080
- Kibana: http://localhost:5601
- TheHive: http://localhost:9000
- Threat engine: http://localhost:8001

### 3. Register and run the agent

The agent workflow is designed to register with the control plane, retrieve provisioning data, and begin participating in the platform workflow. Update the server address in the agent configuration before launching it in your environment.

```bash
cd ../cleint
python main.py
```

## Configuration

The repository uses environment-driven configuration for identity, deployment, and service integration.

### Environment variables

Key configuration values include:

- CLIENT_SECRET — shared secret for service authentication
- KEYCLOAK_ADMIN and KEYCLOAK_ADMIN_PASSWORD — local identity bootstrap credentials
- KEYCLOAK_URL — URL for the Keycloak service
- ELASTICSEARCH_HOST — Elasticsearch endpoint for the session manager and related services
- SERVER_URL — control-plane endpoint used by the agent runtime

The default values are defined in the deployment and service configuration files under the Docker and client directories.

## Running the platform

The runtime model is designed around a coordinated control plane and distributed endpoint participation:

1. An agent boots and registers with the session manager.
2. The control plane provisions the appropriate service context.
3. The agent begins sending heartbeat, telemetry, and operational state.
4. Threat-processing services ingest and correlate events.
5. Security operators can review alerts, sessions, and service activity through the supporting dashboards and interfaces.

## Docker deployment

Docker Compose provisions the core platform services, including:

- the session manager,
- Redis,
- Keycloak,
- Elasticsearch/Kibana/Filebeat,
- Wazuh,
- Suricata,
- TheHive,
- and the threat engine.

This makes it feasible to run a representative SOC/XDR-style environment locally or in a lab setting without tightly coupling every component to a host-specific installation.

## API overview

The platform exposes a small control-plane surface for registration, health checks, and event ingestion. Representative endpoints include:

- /api/register
- /api/heartbeat
- /api/command
- /api/session/services
- /api/session/containers
- /api/v1/threat/events/ingest
- /ready
- /metrics

These interfaces are intended to support a secure and extensible operational workflow rather than a one-off demo deployment.

## Workflow

A typical Sentrix workflow is:

1. Deploy the control-plane services.
2. Register a target host or agent.
3. Provision the relevant service context.
4. Collect and normalize security telemetry.
5. Route events into the threat engine.
6. Review and investigate the resulting signals through the operational dashboards.

## Security

Security is a foundational design objective. The repository implements a layered model that emphasizes:

- identity-based access through Keycloak,
- role-based permissions for operator workflows,
- session integrity checks to detect tampering or unexpected divergence,
- and a controlled operational surface for agent registration and command dispatch.

The design is intended to support enterprise-style governance while remaining practical for lab and pilot deployments.

## Performance and scalability

The architecture is intentionally decomposed so that each layer can evolve independently:

- agents remain lightweight and focused on registration and telemetry participation,
- the control plane can be scaled or hardened independently,
- analytics and detection services can be expanded as ingestion volume grows,
- and the containerized deployment model provides a flexible path for scaling across environments.

This structure is well suited for staged adoption and iterative expansion.

## Roadmap

The current repository provides a strong foundation for a reference security operations platform. Planned evolution areas include:

- stronger production hardening and operational documentation,
- broader connector coverage and richer detection integrations,
- more advanced automation and investigation workflows,
- improved deployment automation for larger environments,
- and a more polished administrative experience for operators.

## Screenshots

The public documentation should include product-style screenshots of:

- the deployment overview,
- the session and agent management experience,
- the detection and investigation workflow,
- and the dashboards used by analysts and operators.

## Architecture overview

Sentrix is best understood as a distributed control-and-detection platform for security operations. Its architecture combines an agent runtime, centralized orchestration, and a threat-processing layer into a cohesive whole that can be deployed in a containerized environment for research, pilot, or operational use.

## Contributing

Contributions are welcome. Please open an issue or submit a pull request with clear context, proposed changes, and any supporting validation steps.

## License

This repository does not currently declare a license file. Before public distribution or external reuse, a formal license such as MIT or Apache-2.0 should be added.

## Acknowledgements

This project builds on the broader ecosystem of open-source security tooling and modern software platforms, including Docker, FastAPI, Redis, Elasticsearch, Keycloak, Suricata, Wazuh, and TheHive.
