#### joeserver docker-compose files

Repository for our joeserver docker-compose stacks. Each is added as a node on our tailscale VPN and available to machines on our tailnet via MagicDNS url `https://{hostname}.squeaker-boga.ts.net`.

~~We're using Portainer for managing our stacks (which is probably more trouble than it's worth but we're relying on its API for healthchecks at the moment).~~ The tyranny of Portainer is over we're just managing these compose stacks ourselves and all the happier for it.

Assumed file structure is:

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
