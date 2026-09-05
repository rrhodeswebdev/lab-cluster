# Twenty CRM

Plain manifests modelled on Twenty's supported `docker-compose.yml`
(`packages/twenty-docker/docker-compose.yml`). Twenty's core team supports only
the Docker deployment - both the community Helm chart in their repo and the
truehostcloud repackaging of it are unmaintained and were broken in several
ways, so this repo drives the four containers directly.

## Secret (created out of band)

The manifests expect a Secret named `twenty` in the `twenty` namespace. It is
deliberately not in git. Create it once per cluster:

```sh
# hex for the DB password: it is embedded in PG_DATABASE_URL, and base64
# can emit / and + which would need percent-encoding inside a URL.
kubectl -n twenty create secret generic twenty \
  --from-literal=POSTGRES_PASSWORD="$(openssl rand -hex 32)" \
  --from-literal=APP_SECRET="$(openssl rand -base64 32)" \
  --from-literal=ENCRYPTION_KEY="$(openssl rand -base64 32)"
```

`ENCRYPTION_KEY` encrypts secrets at rest (OAuth tokens, TOTP secrets, config
values). **Losing it means losing access to every secret in the database**, and
rotating it requires `FALLBACK_ENCRYPTION_KEY` - see Twenty's key rotation
guide. Back this Secret up somewhere outside the cluster.

Replace this with SOPS or sealed-secrets when you want the cluster to be
rebuildable from git alone.

## Per-environment values

`SERVER_URL` and the ingress host are set in `apps/<env>/twenty`, so this base
is reusable by the production cluster.
