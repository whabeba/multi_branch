# Multi-branch container app

This repository contains a small Node.js service. The application and its CI checks run in Docker; Jenkins builds and publishes one image for each supported branch.

## Branch workflow

| Branch | Purpose | Image tag example |
| --- | --- | --- |
| `dev` | Development integration | `dev-42` |
| `stg` | Staging candidate | `stg-43` |
| `main` | Production candidate | `main-44` |

Only these three branch names are accepted by the Jenkinsfile.

## Run locally

```sh
docker build -t multi-branch-app:local .
docker run --rm -p 3000:3000 multi-branch-app:local
```

Open `http://localhost:3000/` and check `http://localhost:3000/health`.

## GitHub setup

1. Push this repository to GitHub and create `dev`, `stg`, and `main` branches.
2. Protect `stg` and `main`; require pull requests and passing Jenkins checks.
3. Use `dev` for development, promote by pull request from `dev` to `stg`, then from `stg` to `main`.
4. Add a webhook under **Settings > Webhooks** pointing to `https://JENKINS_URL/github-webhook/` with push events enabled.

## Jenkins setup

1. Install the Pipeline, GitHub Branch Source, and Docker Pipeline plugins.
2. Ensure the Jenkins agent has Docker installed and permission to run it.
3. Add a Jenkins credential with ID `docker` of type **Username with password** for the container registry.
4. Create **New Item > Multibranch Pipeline**, configure the GitHub repository source and GitHub credentials, and set the script path to `Jenkinsfile`.
5. Run **Scan Multibranch Pipeline Now**. Jenkins will discover `dev`, `stg`, and `main`, then build each branch from its own Jenkinsfile.
6. The pipeline checks JavaScript syntax, builds the image, and pushes `${DOCKER_USERNAME}/multi-branch-app:<branch>-<build-number>`.

For a private GitHub repository, add a GitHub token credential and select it in the branch source configuration. Deployment is intentionally left as the `main` stage: connect that stage to your production platform after the image push succeeds.