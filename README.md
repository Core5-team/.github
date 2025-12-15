# Core5 Organization Repositories

**Purpose of this document**
This document provides a narrative overview of the Core5 repository ecosystem: what each repository is responsible for, how they conceptually fit together and when an engineer is expected to interact with them. 

## What is Core5?

Core5 is a **student DevOps team** formed as part of a **SoftServe Academy internship program**. The team works on a shared multi-project ecosystem over a fixed period (approximately 3 months) with the primary goal of gaining hands-on experience in DevOps, cloud infrastructure and real-world system design.

The work is not strictly bound to a single permanent team structure. Team members rotate between modules and responsibilities to gain exposure to different parts of the system. As a result, some repositories are forked or adapted rather than built entirely from scratch, reflecting a learning-oriented and experimental environment rather than a closed production organization.

Core5 currently supports three main products:
- Birdwatching – a user-facing web application.
- Illuminati – a security-focused internal platform with complex business logic.
- Portal – a minimal entry point linking both platforms.

## Architectural Overview

The Core5 repository ecosystem is intentionally split into infrastructure, applications and operations/deployment layers. This separation exists to keep responsibilities clear and to make it possible to evolve or replace individual platforms without redesigning the entire system. Infrastructure is defined entirely as code and organized into reusable modules. Application repositories focus on business logic and service boundaries.

## Products Overview

### Birdwatching

Birdwatching is a Flask-based web application that provides a comprehensive platform for bird enthusiasts.

The application allows users to:
- Share bird photographs enriched with location data
- Browse a community-driven gallery
- Manage personal photo collections
- Report malicious users (e.g. hunters)

The system also includes an administrative interface for moderation and content management.

**User features** include registration and authentication, photo uploads, personal dashboards and reporting tools. **Administrative features** include full access to all uploaded content, author attribution and moderation actions.

Architecturally, Birdwatching represents a relatively standard web application and serves as an entry point into infrastructure automation and CI/CD workflows.

### Illuminati 

Illuminati is a microservices-based application designed around the idea of a secret society that manages highly sensitive information.

This module was intentionally treated differently from Birdwatching. The focus was not only on infrastructure and automation, but also on application design quality. The goal was to build software that is ready for automation, scaling and cloud-native operation.

The system is built around complex domain rules such as:
- Strict role-based access control
- Internal voting mechanisms
- Invitation-based onboarding
- Scheduled background processes
- Emergency actions, including full data erasure in case of compromise

These requirements introduce real technical challenges related to authorization, data consistency, background jobs and inter-service communication. As a result, Illuminati goes far beyond a simple CRUD application.

The platform uses multiple technologies:
- A Django (Python) backend for business logic, authentication and database access
- A React frontend for user interaction
- Independent Go services for email processing and scheduling

### Portal

Portal is a deliberately minimal application. It serves as a simple entry page that contains links to the Birdwatching and Illuminati platforms.

Its primary purpose is structural rather than functional: providing a single visible access point and allowing the team to experiment with deploying a lightweight application using a different cloud provider.

## Infrastructure Repositories

These repositories define all cloud infrastructure used by the Core5 ecosystem. They are modular by design and are composed together by a central orchestration layer.

### [iac_terragrunt](https://github.com/Core5-team/iac-terragrunt)

This repository is the core of the entire infrastructure setup. It ties together all Terraform modules and is responsible for executing them in the correct order across environments. In practice, this is the only repository that is directly applied when provisioning or updating infrastructure.

### [iac_core](https://github.com/Core5-team/iac_core)

This repository defines environment-level, generic infrastructure that should exist in every environment regardless of the applications deployed. Examples include networking components (VPC, internet gateways), shared buckets, and baseline resources.

It is intentionally application-agnostic: platforms such as Birdwatching are added as modular components and could theoretically be removed from an environment without affecting this core layer.

### [iac_birdwatching](https://github.com/Core5-team/iac_birdwatching)

This repository contains a modular set of Terraform resources specific to the Birdwatching platform. It acts as a "lego block" that can be plugged into an environment when Birdwatching is required.

### [illuminati_eks](https://github.com/Core5-team/illuminati_eks)

This repository serves the same role as [iac_birdwatching](https://github.com/Core5-team/iac_birdwatching), but for the Illuminati platform. It focuses on Kubernetes-based infrastructure and provisions all resources required to run Illuminati inside an EKS cluster.

### [iac_gcp_portal](https://github.com/Core5-team/iac_gcp_portal)

Unlike the other platforms, Portal is deployed on GCP. This repository contains Terraform configuration for GCP services such as Cloud Run and Cloude build resources used to run the Portal application.

### [iac_account_setup](https://github.com/Core5-team/iac_account_setup)

This repository defines infrastructure that should be created only once at the account level. This includes shared services such as Jenkins, SonarQube, and container registries (ECR). These resources are not environment-specific and should not be duplicated.

### [iac_packer](https://github.com/Core5-team/iac_packer)

This repository contains Packer templates used to build reusable AMIs. Prebuilt images reduce provisioning time and external traffic when launching new instances.

### [github_terraform_setup](https://github.com/Core5-team/github_terraform_setup)

This repository provides Terraform templates for automating GitHub repository creation. It ensures consistent configuration (settings, permissions, integrations) across repositories.

## Application Repositories

These repositories contain the actual business logic and user-facing code for each platform.

### Birdwatching

#### [birdwatching_app](https://github.com/Core5-team/birdwatching_app)
  - Technology: Flask
  - Responsibility: Core backend application handling user interactions, uploads and data management.

#### [birdwatching_mcp](https://github.com/Core5-team/birdwatching_mcp)
  - Role: MCP configuration and support tooling.
  - Responsibility: This repository contains configuration and supporting components for the MCP integration used by Birdwatching. MCP enables an AI-powered chat feature that allows users to ask contextual questions, such as inquiries about birds in a specific location (e.g. birds in Lviv). The LLM processes user queries using application-provided context and returns domain-aware responses. This repository defines MCP client and server configuration, integration glue code and related setup required to connect the Birdwatching application with the language model.

### Illuminati

#### [illuminati_frontend](https://github.com/Core5-team/illuminati_frontend)
  - Technology: React
  - Responsibility: User interface for all Illuminati workflows.

#### [illuminati_backend](https://github.com/Core5-team/illuminati_backend)
  - Technology: Django
  - Responsibility: API and business logic layer for Illuminati.

#### [illuminati_email_service](https://github.com/Core5-team/illuminati_email_service)
  - Technology: Go
  - Responsibility: Handles email-related workflows such as invitations, notifications and messages.

#### [illuminati_scheduler_service](https://github.com/Core5-team/illuminati_scheduler_service)
  - Role: Background processing.
  - Responsibility: This service is responsible for triggering time-based actions in the Illuminati system. Rather than executing business logic directly, it instructs the backend to perform specific operations when scheduled conditions are met.

### Portal

#### [portal_frontend](https://github.com/Core5-team/portal_frontend)
  - Technology: HTML, CSS
  - Responsibility: User-facing entry point linking Birdwatching and Illuminati.

## Deployment & Automation

These repositories describe how applications are deployed, configured and observed once infrastructure is in place.

#### [illuminati_helm_charts](https://github.com/Core5-team/illuminati_helm_charts)

This repository contains Helm charts used to deploy Illuminati services into the Kubernetes cluster. ArgoCD consumes these charts automatically to keep the cluster state in sync with Git.

#### [illuminati_gitops](https://github.com/Core5-team/illuminati_gitops)

This repository stores GitOps configuration and environment-specific values. Jenkins pipelines update these values automatically, triggering ArgoCD reconciliations.

#### [illuminati_monitoring](https://github.com/Core5-team/illuminati_monitoring)

This repository contains Ansible playbooks used to configure monitoring and observability tooling. It provisions and configures Grafana Alloy, sets up scraping targets and configures the main node responsible for metrics collection.
