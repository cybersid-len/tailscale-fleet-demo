# Tailscale Fleet Operations Demo

## Overview

This project demonstrates secure, identity-aware access to private fleet operations services using Tailscale.

The scenario simulates a shipping/logistics environment where remote vessel operations users and fleet engineering users need access to private internal services across distributed or NATed environments such as LTE, 5G, or satellite connectivity.

The environment uses:

- Tailscale tailnet connectivity
- MagicDNS
- ACL-based access control
- Docker-hosted internal services
- Role-based validation testing

## Business Problem

Distributed fleet environments often include remote vessels, field systems, or operational sites connected through networks such as LTE, 5G, Starlink, or other NATed internet links.

Traditional remote access can require VPN concentrators, firewall rule management, public IP coordination, bastion hosts, or broad network-level access. This can increase operational overhead and make least-privilege access harder to manage.

This demo shows how Tailscale can provide secure access to private internal services while keeping those services off the public internet.

## Demo Scenario

A fleet organization has two user roles:

| Role | Description |
|---|---|
| Vessel Operations User | Requires access only to operational services |
| Fleet Engineering User | Requires access to engineering and telemetry services |

The infrastructure services are hosted on an Ubuntu VM named:

```text
fleet-services
```


The test workstation joins the same tailnet and validates access based on the authenticated user identity.

```text
Environment Components
Component	                   Purpose
fleet-services	                   Azure Ubuntu VM running Tailscale and Docker services
fleet-demo-ws	                   Test workstation used to validate access as different users
Operations Portal	           Internal operational web service
Engineering Portal	           Restricted engineering web service
IoT Telemetry	                   Simulated onboard telemetry/sensor service 
```



```text
Services
The Ubuntu service node runs three lightweight Docker/Nginx web services:

Service	Port	Purpose
Operations Portal	8081	Operational access for vessel users
Engineering Portal	8082	Restricted engineering access
IoT Telemetry	        8090	Simulated onboard telemetry/sensor service
```




## Architecture


```text
Remote / NATed User Workstation
        |
        | Tailscale encrypted overlay
        |
Tailscale Tailnet
        |
        | MagicDNS / Tailscale IP
        |
fleet-services Ubuntu VM
        |
        | Docker services
        |
+-----------------------------+
| Operations Portal  :8081    |
| Engineering Portal :8082    |
| IoT Telemetry      :8090    |
+-----------------------------+

```


#Access Model


```text
User Role	                        Operations :8081	Engineering :8082	IoT Telemetry :8090	SSH :22
Vessel Operations	                         Allowed	           Denied	             Denied	Denied
Fleet Engineering	                         Allowed	          Allowed	            Allowed	Allowed
Fleet Admin	                                 Allowed	          Allowed	            Allowed	Allowed
```




## Tailscale Features Demonstrated

- Tailnet-based secure connectivity
- MagicDNS service discovery
- ACL-based access control
- Identity-aware access behavior
- Private service access without public application exposure
- Dockerized internal service deployment

## Repository Structure

```text
fleet-demo-lab/
├── README.md
├── docker/
│   ├── docker-compose.yml
│   ├── engineering-portal/
│   │   └── index.html
│   ├── iot-telemetry/
│   │   └── index.html
│   └── operations-portal/
│       └── index.html
├── docs/
├── scripts/
└── tailscale/
    └── acl-policy-working.json
```

## Deployment Steps

### Install Docker

```bash
sudo apt update
sudo apt install -y docker.io docker-compose-v2
sudo systemctl enable --now docker
sudo usermod -aG docker $USER
newgrp docker
```

### Install Tailscale

```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
```

### Start Demo Services

```bash
cd docker
docker compose up -d
docker ps
```

### Local Validation

```bash
curl localhost:8081
curl localhost:8082
curl localhost:8090
```

## Docker Compose

The Docker Compose file is located at:

```text
docker/docker-compose.yml
```

It deploys three Nginx-based services:

- `operations-portal`
- `engineering-portal`
- `iot-telemetry`

## Tailscale ACL Policy

The working ACL policy is stored in:

```text
tailscale/acl-policy-working.json
```

The policy defines:

- `group:admins`
- `group:ops`
- `group:engineering`

For this lab, the working ACL model grants access to services based on the owner group of the service node.

## Validation Results

### Vessel Operations User

| Test | Result |
|---|---|
| Access Operations Portal :8081 | Allowed |
| Access Engineering Portal :8082 | Denied / timeout |
| Access IoT Telemetry :8090 | Denied / timeout |
| SSH :22 | Denied |

### Fleet Engineering User

| Test | Result |
|---|---|
| Access Operations Portal :8081 | Allowed |
| Access Engineering Portal :8082 | Allowed |
| Access IoT Telemetry :8090 | Allowed |
| SSH :22 | Allowed |

## Screenshots

Screenshots collected during validation include:

```text
docker-services-deployment-validation.png
user-onboarding-validation.png
acl-role-based-access-policy-working.png
ops-user-operations-portal-allowed.png
ops-user-engineering-portal-denied.png
ops-user-iot-telemetry-denied.png
engineering-user-operations-portal-allowed.png
engineering-user-engineering-portal-allowed.png
engineering-user-iot-telemetry-allowed.png
```

## Traditional Remote Access Comparison

Traditional VPN and firewall-based access models are valid and widely used. However, in distributed environments they can introduce operational overhead through firewall rules, route management, public IP coordination, bastion hosts, and broader network-level access.

This demo shows how Tailscale can simplify secure connectivity by shifting access control closer to identity and policy, while still supporting private infrastructure access.

## Why Tailscale Is Compelling Here

Tailscale is useful in this scenario because it allows secure connectivity across distributed or NATed environments without exposing the internal services directly to the public internet.

In environments such as vessels, remote offices, LTE/5G-connected locations, or satellite-connected sites, traditional inbound connectivity can become difficult due to NAT, changing IP addresses, or firewall complexity. Tailscale helps simplify this by allowing authenticated devices to join a private overlay network and access only the resources allowed by policy.

## What Worked Well

- Docker made it simple to deploy multiple lightweight internal services.
- Tailscale quickly connected the Azure-hosted service node and the test workstation.
- MagicDNS simplified access by avoiding reliance on memorized IP addresses.
- ACL testing clearly demonstrated different access outcomes by user role.
- The final setup stayed small and focused while still demonstrating the core access-control behavior.

## What Was Difficult or Surprising

The most time consuming part was identity onboarding with consumer email accounts. Some newly created accounts and aliases introduced friction due to identity-provider anti-abuse protections or incomplete onboarding flows.

In a production deployment, this would normally map to an existing organizational identity provider such as Google Workspace, Microsoft Entra ID, Okta, JumpCloud, or OIDC.

Another learning point was Tailscale ACL targeting. For this lab, targeting the owner group of the service node was the clearest working model for validating role-based access.

## Future Improvements

With more time, I would add:

- Subnet router simulation for private onboard IoT/OT networks
- Tailscale SSH policy refinement
- Device posture checks
- Integration with an organizational IdP
- Automated deployment with Terraform or cloud-init
- A cleaner final architecture diagram
- A recorded walkthrough demo

## AI Usage Disclosure

AI assistance was used for:

- Brainstorming the demo scenario
- Structuring the README
- Refining the architecture narrative
- Documentation wording

All implementation decisions, testing, validation, and final review were performed manually. AI suggestions were reviewed and adjusted based on actual lab behavior.
