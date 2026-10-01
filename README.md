# Node.js CI/CD Demo

A small dependency-free Node.js web app that demonstrates a complete **test → build → push** CI/CD workflow with GitHub Actions and Docker Hub.

## What is included

- Node.js HTTP server and responsive landing page
- `/health` and `/api/info` JSON endpoints
- Automated tests using the built-in Node.js test runner
- Reproducible build output in `dist/`
- Multi-stage, non-root Docker image with a health check
- `.github/workflows/main.yml` for CI on pull requests and Docker Hub deployment on pushes to `main`

## Run locally

Requirements: Node.js 24+ and npm.

```bash
npm ci
npm test
npm run build
npm start
```

Open <http://localhost:3000>. Check health at <http://localhost:3000/health>.

## Run with Docker

```bash
docker build -t nodejs-demo-app .
docker run --rm -p 3000:3000 nodejs-demo-app
```

## Configure GitHub Actions

1. Create a Docker Hub access token with read/write permission.
2. In the GitHub repository, open **Settings → Secrets and variables → Actions**.
3. Add the repository variable `DOCKERHUB_USERNAME` with your Docker Hub username.
4. Add the repository secret `DOCKERHUB_TOKEN` with the access token.
5. Push this project to the `main` branch.

Pull requests run the test/build job without publishing. A push to `main` runs tests first, then builds and publishes these Docker Hub tags:

- `<username>/nodejs-demo-app:latest`
- `<username>/nodejs-demo-app:sha-<commit>`

The deploy job uses `needs: test`, so a failed test prevents an image from being published.

## Pipeline flow

```text
push or pull request
        |
        v
 npm ci → npm test → npm run build
        |
        v (main push only)
 Docker build → Docker Hub push
```

## Interview questions

### 1. What is CI/CD?

Continuous Integration (CI) automatically integrates code changes and validates them with builds and tests. Continuous Delivery or Deployment (CD) takes validated changes and prepares or releases them to an environment. Delivery keeps a manual production approval; deployment can release automatically.

### 2. How do GitHub Actions work?

A YAML workflow in `.github/workflows/` listens for repository events such as pushes and pull requests. An event starts a workflow run containing one or more jobs. Runners execute each job's ordered steps, which can run shell commands or reusable actions.

### 3. What are runners?

Runners are machines that execute jobs. GitHub-hosted runners are short-lived virtual machines managed by GitHub. Self-hosted runners are machines managed by the repository owner and are useful when custom hardware, software, or network access is required.

### 4. What is the difference between jobs and steps?

A job is an isolated unit assigned to one runner. Jobs run in parallel by default and can depend on each other with `needs`. Steps are the ordered commands or actions inside one job; they share that job's workspace and environment.

### 5. How are secrets secured in GitHub Actions?

Store credentials as GitHub Actions secrets, reference them with the `secrets` context, grant the minimum necessary permissions, and never print them. Prefer short-lived OIDC credentials where supported. Protect environments and require reviews for sensitive deployments.

### 6. How should deployment errors be handled?

Fail fast on tests and builds, make deployments observable with logs and health checks, keep immutable versioned artifacts, and use automated rollback or redeploy a previously known-good image. Production environments can add approvals and concurrency controls to avoid overlapping releases.

### 7. Explain the Docker build-push workflow.

The workflow logs into a registry, sets up Buildx, generates consistent tags and labels, then builds the Dockerfile and pushes the resulting image. Build cache reduces later build times. In this project, deployment only starts after the test job succeeds.

### 8. How can a CI/CD pipeline be tested locally?

Run the same commands used by CI (`npm ci`, `npm test`, `npm run build`, and `docker build`). A local Actions emulator such as `act` can provide an additional check, but it may differ from GitHub-hosted runners and does not replace a real workflow run.

## Troubleshooting

- **Docker login fails:** confirm the variable and secret names and regenerate the Docker Hub token.
- **Image name is invalid:** make sure `DOCKERHUB_USERNAME` is set and uses lowercase Docker Hub naming rules.
- **Tests fail:** run `npm test` locally and inspect the failing assertion before pushing again.
- **Container is unhealthy:** inspect `docker logs <container>` and verify port 3000 is available.
