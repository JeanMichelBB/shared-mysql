# shared-mysql

One MySQL 8 instance (`mysql` Deployment/Service/PVC in the `default` namespace) shared by multiple app backends, each with its own database inside it.

## Why a separate repo

Originally each app repo (`Apercu`, `x`, `BotWhy`) declared its own copy of the same `mysql`/`mysql-pvc`/`mysql-secret` resources — harmless under manual `kubectl apply`, but under ArgoCD, multiple `Application`s claiming the same resource name causes ownership conflicts (each `Application` tries to reconcile the shared resource against its own git source). Moving it here means exactly one `Application` (`shared-mysql`) owns it; every app connects to it as an existing external dependency.

## Connected apps and their databases

| App | Database | Connects via |
|---|---|---|
| Apercu | `apercu` | `mysql:3306` |
| x | `twitter_db` | `mysql:3306` |
| BotWhy | `chatbox_db` | `mysql:3306` |
| PopRoom | `poproom` | `mysql:3306` |

To add a new app: create its database and grant access manually (`CREATE DATABASE ...; GRANT ALL ON ....* TO 'user'@'%';`), then point its backend's connection string at `mysql:3306/<its db name>`. Don't add another `mysql.yaml` to the new app's own repo.

## Secrets

`k3s/secrets/mysql-secret.yml` is gitignored — apply it manually to the cluster, same pattern as every other app repo.

## Backup

A `mysql-backup` CronJob (defined in the cluster, not this repo) runs daily, dumping all databases to the TrueNAS NFS share.
