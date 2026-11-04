# Mautic service for Kubernetes on Wodby

Run Mautic marketing automation on Kubernetes with Wodby.

This repository defines the Wodby service manifests and operational
configuration for Mautic.

- [Browse Wodby services](https://wodby.com/services)
- [Wodby service documentation](https://wodby.com/docs/2.0/services/)
- [Service manifest reference](https://wodby.com/docs/2.0/services/template/)

## Wodby stacks using this service

- [Mautic application stack](https://github.com/wodby/stack-mautic)

## Service overview

| Property | Manifest configuration |
| --- | --- |
| Service name | `mautic` |
| Type | Application service |
| Image | [`wodby/mautic`](https://github.com/wodby/mautic): Mautic, Apache and PHP-FPM, prebuilt |
| Versions | `7` |
| Endpoint | HTTP on port 80 |
| Service links | Database (MariaDB or MySQL), required; Files storage, required; SMTP server |
| Settings | Admin username, Admin password (required), Admin email (required) |
| Volumes | `files`, shared |
| Derivatives | `mautic-cron`, `mautic-worker` |
| Application build | None: the service runs the prebuilt image |
| Operations | 1 action |

## Use this service

Use this service through [Mautic application stack](https://github.com/wodby/stack-mautic), or reference `mautic` from a
custom Wodby stack.

A service is a reusable component and does not deploy by itself. The stack
defines its links, settings, versions, resources, and relationship to the rest
of the application.

## Maintain a custom version

1. Fork this repository.
2. Edit the service manifest and referenced files.
3. Import the repository as a [Git-backed service](https://wodby.com/docs/2.0/services/create/#create-a-git-backed-service).
4. Reference the service from a stack manifest.

Keep service, workload, container, endpoint, link, volume, config, and
derivative names stable unless dependent stacks and app-level overrides are
updated at the same time.

Validate the manifests with:

```bash
wodby service validate-manifest service.yml --org <org-id>
```

See the [service manifest reference](https://wodby.com/docs/2.0/services/template/) and the [managed services index](https://github.com/wodby/services).
