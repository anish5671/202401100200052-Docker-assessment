# FestPass container security assessment

## Personal details

- Name: `YOUR NAME`
- REGNO: `YOUR REGISTRATION NUMBER`
- PORT: `80 + the last two digits of your REGNO`
- Docker Hub username: `YOUR DOCKER HUB USERNAME`
- Docker Hub repository: `https://hub.docker.com/r/YOUR_DOCKERHUB_USER/fest-backend`
- AI tools: GitHub Copilot, used to scaffold the app and explain command errors.

Replace every placeholder and `.env.example` value with your own details. Keep `.env`, registry credentials, and `cosign.key` out of Git. Run each command yourself and capture your own unedited screenshots in `screenshots/`.

## Run the stack

Copy `.env.example` to `.env`, set your actual REGNO and the required PORT (`80` plus your final two registration-number digits), and set a private DB password. Then run:

```sh
docker compose up -d --build
docker compose ps
curl "http://localhost:${PORT}/api/health"
```

Open `http://localhost:PORT` (replace `PORT` with your number) and submit two registrations. Verify persistence with `docker compose down` followed by `docker compose up -d`.

## Stage 1 image

```sh
docker build -t fest-backend:0.1 ./backend
docker image ls fest-backend
docker run -d --name backend -p "${PORT}:5000" -e REGNO="$REGNO" fest-backend:0.1
docker ps
curl "http://localhost:${PORT}/api/health"
docker logs backend
docker rm -f backend
```

Set `REGNO` and `PORT` from your `.env` first (`set -a; . ./.env; set +a` in bash). Stage 1's standalone backend is expected to report `database: false`.

## Stage 3: scan and harden

Save your own scanner output; do not edit or fabricate the reports:

```sh
mkdir -p reports
trivy image fest-backend:0.1 | tee reports/trivy-before.txt
```

The backend now uses the version-pinned `python:3.14.2-slim-bookworm` base, runs as UID 10001, and pins updated Flask, Werkzeug, and psycopg2-binary dependencies. Build `fest-backend:0.2`, save its actual scan as `reports/trivy-after.txt`, and run the assessment's `--severity CRITICAL --ignore-unfixed --exit-code 1` check. Fill in the measured 0.2 counts and size below.

| Image | CRITICAL | HIGH | Size |
| --- | ---: | ---: | ---: |
| `fest-backend:0.1` | 234 | 2249 | 412MB |
| `fest-backend:0.2` | 9 | 88 | 59.2MB |

## Stage 4: publish

Build the hardened image with `APP_VERSION=1.0.0`, tag and push both `1.0.0` and `1.0` to your public Docker Hub repository. Record the real pushed digest here: `sha256:REPLACE_WITH_DIGEST`.

Update the backend service in `compose.yaml` to use your own `YOUR_DOCKERHUB_USER/fest-backend:1.0.0` image instead of `build: ./backend`, then verify Compose pulls and runs that published image.

## Stage 5: sign and verify

Generate a password-protected cosign key pair locally. Commit only `cosign.pub`. Sign the `1.0.0` image by its registry digest, verify it with `cosign.pub`, then build and push `1.0.1` without signing it and show that verification rejects it. Never add the private key or its password to this repository.

## Evidence and completion

Put genuine screenshots named with the assessment evidence codes in `screenshots/`. Include the real vulnerability results, digest, Docker Hub tags, and any incomplete step here. This starter does not contain generated scan reports, signatures, screenshots, or claims that registry operations succeeded; produce those on your own machine.