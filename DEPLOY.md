# Vaultwarden Dokploy Deployment

This repo is prepared for deploying Vaultwarden as a separate Dokploy Docker Compose app on `vault.ravitejaknts.com`.

## Safety Model

- Do not run Caddy for this app on the VPS.
- Do not publish host ports from this compose file.
- Let Dokploy/Traefik handle public `80`/`443` and Let's Encrypt.
- Keep this as a separate Dokploy project from the existing production web app.
- Use the named Docker volume `vaultwarden_data` for `/data` so Dokploy volume backups can be enabled.

## Dokploy Setup

1. Point DNS:
   - `vault.ravitejaknts.com` -> VPS IP

2. Create a new Dokploy Docker Compose app from this private GitHub repo.

3. Use `compose.yaml` as the compose file.

4. Add environment variables in Dokploy from `.env.production.example`.
   Dokploy stores these in its generated `.env` file, and `compose.yaml` reads them with `${VARIABLE}` substitution.

5. In the Dokploy Domains tab, add:
   - Host: `vault.ravitejaknts.com`
   - Container port: `80`
   - HTTPS enabled through Dokploy/Traefik

6. Deploy.

7. Create the first Vaultwarden account.

8. Set `SIGNUPS_ALLOWED=false` in Dokploy and redeploy.

9. Configure SMTP before inviting real users.

## Admin Token

Generate a hashed admin token instead of storing a plain token:

```sh
docker run --rm -it vaultwarden/server:1.35.8 /vaultwarden hash
```

Use the generated Argon2 PHC string as `ADMIN_TOKEN`. The password you type into the hash command is what you use to log in at `/admin`.

## Backups

Enable Dokploy volume backups for `vaultwarden_data` before storing real passwords. The important Vaultwarden data lives under `/data`, including the SQLite database, keys, attachments, and sends.
