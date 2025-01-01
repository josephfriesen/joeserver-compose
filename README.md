#### joeserver docker-compose files

Repository for our joeserver docker-compose stacks. Each is added as a node on our tailscale VPN and available to machines on our tailnet via MagicDNS url `https://{hostname}.squeaker-boga.ts.net`.

We're using Portainer for managing our stacks (which is probably more trouble than it's worth but we're relying on its API for healthchecks at the moment).

Assumed file structure is:

- Docker compose files live at `/mnt/user/appdata/portainer/compose/{id}/{service}/docker-compose.yml`. These are updated automatically when we update the appropriate file here in the repo
- Tailscale `ts` service config (`./ts-compose.yml`) lives at `/mnt/user/appdata/portainer/compose/{id}/ts-compose.yml`. This file is _not_ managed by Portainer nor are updates to this repo automatically reflected in Portainer stacks. We shouldn't need to edit this file often (TODO?: move to `tsdproxy` and just have a single tailscale node rather than one sidecar container for each stack?), but if we do:
  - Make changes to `./ts-compose.yml`
  - Stop and redeploy all stacks using the tailscale sidecar container (run appropriate `joeserver-tools` cmd).
- Tailscale auth key lives on server. Key is persistent (each container will use the one key to authenticate to tailscale) but short-lived (expires in one day). When adding a new container to authenticate with tailscale, get new short-lived key from Tailscale, update file on server.
