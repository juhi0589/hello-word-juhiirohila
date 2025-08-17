Hello/World — Docker → GHCR → ECR → Elastic Beanstalk

A tiny site that shows “Hello” on the first visit and “World” on later visits using localStorage.
Containerized with Docker. Images are built by GitHub Actions → stored in GHCR (public) → mirrored to Amazon ECR → deployed to Elastic Beanstalk by uploading a Dockerrun.aws.json.

Live URL
http://Hello-word-juhiirohila-5.eba-uvnvup4p.eu-central-1.elasticbeanstalk.com

How it works (app)

On page load, JS checks localStorage.visited.

If missing → shows Hello, then sets visited=true.

If present → shows World.

Repository layout
.
├─ Dockerfile          # Nginx serves index.html on port 80
├─ index.html          # localStorage logic
├─ nginx.conf          # Nginx config (listen 80)
└─ .github/workflows/  # CI pipelines (build to GHCR, mirror to ECR)

Prerequisites

AWS account with permissions for ECR and Elastic Beanstalk.

Elastic Beanstalk Web Server environment on Docker running on 64-bit Amazon Linux 2.

EB EC2 instance profile has permission to pull from ECR: attach AmazonEC2ContainerRegistryReadOnly.

Region: eu-central-1

AWS Account ID: 693148422260

ECR repo: hello-word-juhiirohila

EB Application: Hello-word-juhiirohila

EB Environment: Hello-word-juhiirohila-5

Since GHCR is public, no GHCR token is required.

What the workflows do (high level)

CI — Build & Push Docker to GHCR: on push to main, builds the Docker image from this repo and publishes it to GHCR.

Mirror GHCR → ECR: pulls the GHCR image, logs into ECR, retags, and pushes to your ECR repo 693148422260.dkr.ecr.eu-central-1.amazonaws.com/hello-word-juhiirohila:latest.

Deploy to Elastic Beanstalk (manual upload, no S3)

Confirm the image exists in ECR
AWS Console → ECR → hello-word-juhiirohila → confirm the latest tag.

Create Dockerrun.aws.json (single-container v1)
Use this exact content:

{
  "AWSEBDockerrunVersion": 1,
  "Image": {
    "Name": "693148422260.dkr.ecr.eu-central-1.amazonaws.com/hello-word-juhiirohila:latest",
    "Update": "true"
  },
  "Ports": [
    { "ContainerPort": "80" }
  ]
}


Upload to EB and deploy

AWS Console → Elastic Beanstalk → Application: Hello-word-juhiirohila → Environment: Hello-word-juhiirohila-5

Click Upload and deploy

Select your Dockerrun.aws.json file → Deploy

Wait until health is Green.

Verify
Open the environment URL shown in EB.

First visit shows Hello.

Refresh shows World.

Updating the site

Push any change to main (e.g., index.html).

The workflows will:

rebuild the image → push to GHCR

mirror into ECR (:latest)

Then repeat the EB upload:

upload Dockerrun.aws.json again in Upload and deploy.

Troubleshooting
Symptom	Cause	Fix
EB deploy fails or remains grey	EB needs a valid Dockerrun.aws.json	Ensure the file name is exactly Dockerrun.aws.json and JSON is valid
EB stuck “Downloading Docker image”	EB instance can’t pull from ECR	Attach AmazonEC2ContainerRegistryReadOnly to EB instance profile

Clean-up

Terminate the EB environment and delete the application if not needed.

Delete ECR images/repository if not needed.

Remove GitHub Actions secrets if archiving the project.
