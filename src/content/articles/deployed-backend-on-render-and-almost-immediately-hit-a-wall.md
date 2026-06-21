---
title: "I deployed my backend on Render and almost immediately hit a wall."
date: 2026-06-20
---

I deployed my backend on Render and almost immediately hit a wall.

The CNJ — Brazil's national justice council — has a public API I was using to pull data for Bidtrack, a side project for tracking legal bids. The issue: Render provisions servers in North America by default. The CNJ API blocks requests from outside Brazil. Every call returned 403.

Two options: add a Brazilian proxy (expensive) or host the API myself on AWS in sa-east-1 (São Paulo). I went with AWS — not just to save money, but because I'd been abstracting infrastructure for years and this felt like a good excuse to stop.

For infra management I picked Pulumi over Terraform. I'd never touched it before and that was partly the point. The TypeScript semantics felt immediately familiar — defining AWS resources in the same language I write my services in made the whole thing easier to reason about.

The setup I landed on: a VPC with public subnets and no NAT gateways (no point paying for what I wasn't using), an ECS Fargate service running the API in a Docker container pulled from ECR, an Application Load Balancer handling HTTPS termination with an ACM certificate, and SSM Parameter Store for secrets — injected directly into the container via the ECS task execution role at startup. No .env files floating around anywhere.

I also had a daily sync job that needed to run at 6am Brazil time. Instead of adding a cron runner inside the API container, I created a small Lambda that hits the `/cron/sync-all-stages` endpoint, reads the secret key from SSM at runtime, and gets triggered by EventBridge Scheduler. It felt cleaner to keep the scheduling logic out of the app itself.

Autoscaling is configured between 1 and 5 tasks, tracking CPU at 50%. Probably overkill for where traffic is today, but it costs nothing to have in place and I didn't want to think about it later.

Honestly, the hardest part wasn't picking the right services. It was accepting that I was going to spend a weekend on infrastructure instead of shipping features. But I came out of it with a setup I actually understand end-to-end — and that feels better than a platform that hides everything from me.
