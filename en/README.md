# Documentation - Index

Welcome to the Remote Server project documentation.

## Navigation

| # | Section | Description |
|---|---------|-------------|
| 01 | [Introduction](./01-introduction.md) | Overview, architecture and goal |
| 02 | [Prerequisites](./02-prerequisites.md) | Hardware, software and required accounts |
| 03 | [Virtual Machine (VM)](./03-vm/README.md) | Creating the VM in VirtualBox with Ubuntu Server |
| 04 | [Domain and DNS](./04-domain.md) | Domain registration and Cloudflare DNS configuration |
| 05 | [Public Exposure](./05-public-exposure/README.md) | Methods to expose the VM on the internet |
| 06 | [Coolify](./06-coolify/README.md) | Installation and application management |
| 07 | [Security](./07-security.md) | Security best practices |
| 08 | [Maintenance](./08-maintenance.md) | Backup, updates and monitoring |
| 09 | [Applications](./09-applications/README.md) | Odysseus, n8n, Evolution Go and more |
| 10 | [Multichannel Chatbot](./10-multichannel-chatbot/README.md) | Bot for WhatsApp, Telegram, Facebook, Web, Email |
| 11 | [Next Suggestions](./11-suggestions.md) | Recommended applications (monitoring, CI/CD, backup) |
| -- | [Dev Tools](./dev-tools.md) | Webhook inspectors, API clients, tunnels, mocks |

## Conventions used in this documentation

- **`vm.doc.local`**: example domain. Replace with your real domain.
- **`ubuntu-vm`**: VM hostname. Can be changed during installation.
- Command blocks: assume the host operating system has already been identified.

## About

This project documents step by step how to turn a local computer (or VM) into a publicly accessible server, managed by Coolify, with no monthly cost beyond the domain.
