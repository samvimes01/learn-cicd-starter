# learn-cicd-starter (Notely)

This repo contains the starter code for the "Notely" application for the "Learn CICD" course on [Boot.dev](https://boot.dev).

![code coverage badge](https://github.com/samvimes01/learn-cicd-starter/actions/workflows/ci.yml/badge.svg)

## Local Development

Make sure you're on Go version 1.20+.

Create a `.env` file in the root of the project with the following contents:

```bash
PORT="8000"
```

Run the server:

```bash
go build -o notely && ./notely
```

*This starts the server in non-database mode.* It will serve a simple webpage at `http://localhost:8000`.

You do *not* need to set up a database or any interactivity on the webpage yet. Instructions for that will come later in the course!

## Gloud deployment

Within Artifact Registry in the GCP console, enable the Artifact Registry API and create a new repository:

Name: notely-ar-repo
Format: Docker
Mode: Standard
Location type: Region
Region for deployment: us-central1
Leave "Google-managed encryption key" selected

CREATING A SERVICE ACCOUNT
Go to the IAM & Admin Service Accounts section of the GCP console.
Create a service account (I named mine "Cloud Run Deployer") with these permissions:
Cloud Build Editor
Cloud Build Service Account
Cloud Run Admin
Service Account User
Viewer
Create a JSON key for that service account and download it to your computer.

ADD THE KEY AS A SECRET IN GITHUB ACTIONS
Go to your GitHub Repo > Repository Settings > Secrets and variables > Actions > New repository secret
Name: GCP_CREDENTIALS
Secret: Paste the entire JSON key from the file you downloaded from GCP
Save the secret

### Deploy image to artifact registry

```sh
# gcloud builds submit --tag REGION-docker.pkg.dev/PROJECT_ID/REPOSITORY/IMAGE:TAG .
gcloud builds submit --tag us-central1-docker.pkg.dev/notely-414013/notely-ar-repo/im:tag .
```

GOOGLE CLOUD RUN
Cloud Run is a serverless container hosting service. It's a great fit for Notely because it has a generous free tier, scales automatically, and automagically configures pesky infrastructure like:

Load balancing
DNS
HTTPS
In a nutshell, we give Cloud Run a Docker image, and it runs it for us.

CLOUD RUN SERVICES
Cloud run has 2 types of applications:

Services
Jobs
A "service" is a Cloud Run application that listens and responds to web requests, as opposed to a "job" which is simply a task that runs to completion.

### Deploy to cloud run after image deployed to artifact registry

```sh
# gcloud run deploy notely --image REGION-docker.pkg.dev/PROJECT_ID/REPO_NAME/IMAGE:TAG --region REGION --allow-unauthenticated --project PROJECT_ID --max-instances=4
gcloud run deploy notely --image us-central1-docker.pkg.dev/notely-414013/notely-ar-repo/notely:latest --region us-central1 --allow-unauthenticated --project notely-414013 --max-instances=4
```