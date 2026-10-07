# AGENTS.md

## What this project is
Static marketing site (`index.html`, `about.html`, `contact.html`, `privacy.html`, `terms.html`,
`assets.css`, `logo.png`) plus one PHP endpoint, `send.php`, which sends the contact-form message
through PHPMailer (vendored copy in `phpmailer/`, required by relative path — no Composer install
step, no `vendor/` directory needed).

## Running it
```
docker compose -f docker-compose.base44.yml up -d --build
```
One service, `web`, runs `php -S 0.0.0.0:3000` from the repo root with the working tree
bind-mounted, so HTML/CSS/JS/PHP edits are picked up on the next request — no rebuild and no
restart required for code changes.

Verify: `curl -sS -o /dev/null -w '%{http_code}\n' http://localhost:3000/` returns `200`, and
`curl -sS -X POST -d 'name=Test&email=t@example.com&message=hi' http://localhost:3000/send.php`
returns JSON (`{"status":"error"...}` with an SMTP message when no credentials are configured,
`success` when they are).

## Configuration
`send.php` reads the mailbox password from the environment variable `SMTP_PASSWORD`, delivered by
the platform to `/run/base44/app.env` and loaded via the service's `env_file` in the compose file.
Host (`smtp.titan.email`), port (465, SSL), username and recipient are still literals in `send.php`.
Without `SMTP_PASSWORD` the site renders normally and only the contact form fails, returning
`{"status":"error","message":"Mailer Error: ..."}` as JSON.

## Notes
- No database, no build step, no migrations.
- PHP is only needed for `send.php`; the HTML pages are plain static files.
