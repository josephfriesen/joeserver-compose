## joeserver docker-compose files

### DEPRECATED, don't need this anymore!

Hooray, with release of Unraid v7.0.0, docker containers on unraid have built-in support for Tailscale, installing an app via docker includes "Use Tailscale" option, which will alter the container entrypoint to first download Tailscale within the container, spin up Tailscale, then prompt for authentication (no more API keys to auth, must be done manually).

For now, just keeping services Dozzle and Searxng as docker-compose stacks rather than an Unraid docker container. Dozzle for some reason is the only service that gave a command not found when trying to install Tailscale on start, and Searxng requires a separate Redis instance so keeping it as a compose stack makes sense. These will continue to be compose stacks with a Tailscale sidecar container. The rest of the compose files we'll keep here, if we need to delete/create/edit a service config within Unraid we have these configs as a reference.

### README (for historical purposes, I suppose?)

Repository for our joeserver docker-compose stacks. Each is added as a node on our tailscale VPN and available to machines on our tailnet via MagicDNS url `https://{hostname}.squeaker-boga.ts.net`.

~~We're using Portainer for managing our stacks (which is probably more trouble than it's worth but we're relying on its API for healthchecks at the moment).~~ The tyranny of Portainer is over we're just managing these compose stacks ourselves and all the happier for it.

#### File structure

- Docker compose files live at `joeserver-compose/{service}/docker-compose.yml`. When making a change to a compose file, can make changes to these files, spin up `docker compose` make sure the changes are working then check changes in.
- Tailscale `ts` service config (`./ts-compose.yml`) lives at the project root `joeserver-compose/ts-compose.yml`.
  - Each compose stack extends this sidecar service as `{service}-ts` in its compose file.
  - Machine's Tailscale hostname should be stated in the .yml file as `services.{service}-ts.hostname`.
  - Requires .`.env` file, which defines our `TS_AUTHKEY`, which is a short-lived key for authenticating the `ts` sidecar service with Tailscale. `.env` lives on the server in the Tailscale config directory. The key is set to not be a one time use key, it should work for multiple services, but should have a very short expiry no more than one day.
  - Each service is non-ephemeral and should only need to authenticate with Tailscale once. When adding a new service, or if re-auth is necessary, generate a new key from Tailscale admin console, update the `.env` file on server, and `docker compose up`. Should then see the service show up in Tailscale machines under the `hostname`, and see details of authentication in logs.
  - We shouldn't need to edit this file often (TODO?: move to `tsdproxy` and just have a single tailscale node rather than one sidecar container for each stack?), but if we do:
    - Make changes to `./ts-compose.yml`
    - Stop and redeploy all stacks using the tailscale sidecar container (run appropriate `joeserver-tools` cmd).
    - If all is well, check changes into git.
- Each ts service needs a `serve.json` configuration file, essentially to tell the ts sidecar service which container port to forward traffic to.
  - For each service, we have a subfolder in the `appdata/tailscale` folder that will contain the `serve.json` file and the Tailscale state. Set up a volume mapping to this directory on the server to the container folder `/config` in each tailscale sidecar service.
  - The `serve.json` files live here in the repo, for each service set up a hard symlink in `appdata/tailscale/{service}` subfolder to link to this file.

#### Adding a new compose stack
- Create `/{service}` subfolder, copy `docker-compose.yml.template`, `serve.json.template` to `/{service}/docker-compose.yml`, `/{service}/serve.json.template`.
- Replace the placeholder values in `docker-compose.yml` and `serve.json`. Set the compose configuration as needed.
- Make new subdirectory `/appdata/tailscale/{service}`. In subdirectory, `ln joeserver-compose/{service}/serve.json`.
- In Unraid Compose Manager, add new compose stack with name `{service}`, stack directory `joeserver-compose/service`. In the stack that is created, confirm that the compose file there matches the contents of the `docker-compose.yml` file (no reason it shouldn't).
- Click `Compose Up`, wait for it to spin up, check logs. Once `{service}-ts` has authenticated with Tailscale, check Tailscale admin console for service to be listed. Then hit the tailnet URL `https://{service}.squeaker-boga.ts.net`. If it's working, should take a few seconds to load while generating cert.
