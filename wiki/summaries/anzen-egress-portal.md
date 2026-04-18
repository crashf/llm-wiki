---
title: Anzen Egress Portal
type: summary
tags: [project, wireguard, mikrotik, pund-it, automation, portal]
sources: 2
created: 2026-04-07
updated: 2026-04-07
---

## Summary

The Anzen Egress Portal is a Next.js-based management interface that automates WireGuard tunnel deployment for Pund-IT's Anzen Egress service. It eliminates manual MikroTik CLI configuration by providing a wizard-driven interface for pre-staging CPEs, deploying tunnels, and monitoring usage. The portal integrates with ConnectWise Manage for customer lookup, agreement mapping, and billing.

## Key Points

- **Zero-Touch Deployment**: Three-phase model — Pre-Stage (bootstrap via USB) → Deploy (wizard) → Day-2 Operations
- **Wizard-Driven**: 10-step deployment wizard handles WAN/LAN/firewall configuration without CLI access
- **ConnectWise Integration**: Company search, agreement mapping, billing sync with CW Manage
- **Full IPAM**: Manages public IP pools (/30 allocations from /28 blocks) and transit networks
- **Real-Time Monitoring**: Tunnel status polling every 5 minutes, bandwidth tracking, Prometheus export
- **Tech Stack**: Next.js 15, shadcn/ui, Drizzle ORM, PostgreSQL, NextAuth.js + Azure AD SSO

## Architecture

- **Frontend**: Next.js 15 App Router with shadcn/ui + Tailwind
- **Backend**: Next.js API Routes + Server Actions
- **Database**: PostgreSQL with Drizzle ORM (full schema: users, gateways, cpe_devices, deployments, ip_pools, tunnel_metrics, alerts)
- **Integrations**: MikroTik REST API (RouterOS 7+), ConnectWise Manage API, Azure AD SSO, Prometheus/Grafana
- **Hosting**: Docker on punsvnfdocker01/02

## Connections

- Related to [[anzen-egress]] concept — the underlying service this portal manages
- Related to [[mikrotik-edge-router]] — the edge routers the portal configures
- See also [[connectwise-integration]] — CW Manage integration patterns
