# Watchtower ECR Helper with Mattermost Notification

![diagram](img\diagram.png)

This repository contains a setup for Watchtower running in both HTTP-API and polling modes, integrated with AWS ECR Credentials Helper and Mattermost Channel notifications.

## Watchtower (v1.7.1)

Watchtower is a tool that automates the process of updating Docker containers. When a new image is pushed to a Docker Hub or private registry, Watchtower pulls the updated image, stops the currently running container, and redeploys it with the same configuration.

## AWS ECR Credentials Helper (v0.8.0)

The Amazon ECR Docker Credential Helper simplifies authentication with the Amazon Elastic Container Registry (ECR) by automating the process of managing credentials for Docker.

## Prerequisites

Before you start, ensure that you have the following:

1. Docker Engine installed on the host machine.
2. AWS CLI installed and configured on the host machine.
3. Mattermost Channel Webhook and a Mattermost Bot Account (optional, for notifications).

## Setup Guide

### 1. Clone the Repository

To get started, clone this repository to your local machine:

```
git clone <repository-url>
cd <repository-directory>
```

### 2. AWS IAM Policy

Create an AWS IAM role with the following policy to grant access to Amazon ECR:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "ECRAccess",
            "Effect": "Allow",
            "Action": [
                "ecr:CompleteLayerUpload",
                "ecr:UploadLayerPart",
                "ecr:InitiateLayerUpload",
                "ecr:BatchCheckLayerAvailability",
                "ecr:PutImage",
                "ecr:BatchGetImage",
                "ecr:GetDownloadUrlForLayer"
            ],
            "Resource": "arn:aws:ecr:<YOUR_AWS_REGION>:<YOUR_AWS_ACCOUNT_ID>:repository/*"
        },
        {
            "Sid": "GetAuthToken",
            "Effect": "Allow",
            "Action": "ecr:GetAuthorizationToken",
            "Resource": "*"
        }
    ]
}
```

Replace `<YOUR_AWS_REGION>` and `<YOUR_AWS_ACCOUNT_ID>` with your actual AWS region and account ID.

### 3. Configure AWS CLI

Install and configure the AWS CLI by following the [AWS CLI installation guide](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html). After installation, run:

```
aws configure
```

You'll need your AWS Access Key ID, Secret Access Key, region, and output format.

### 4. Mattermost Setup
![Mattermost Notification](img\mattermost-notification.png)

To enable notifications in Mattermost, create a channel, an incoming webhook, and (optionally) a bot account. Follow the [Mattermost webhook setup guide](https://docs.mattermost.com/developer/webhooks-incoming.html).

### 5. Configure Environment Variables

Copy the example `.env` file to `.env`, then customize the configuration:

```
cp .env.example .env
nano .env
```

Make sure to update the environment variables with your specific settings, including AWS credentials, Mattermost webhook URL, and any additional Watchtower configurations.

### 6. Start the Docker Container

Once everything is configured, start the Docker container using `docker-compose`:

```
docker compose up -d --remove-orphans --force-recreate
```

This will start Watchtower, which will begin monitoring your Docker containers for updates.

## Configuration Options

### Running in HTTP API Mode

If you want Watchtower to operate in HTTP API mode (manual updates via API), you can use the following configuration in your `.env` file:

```env
WATCHTOWER_HTTP_API=true
WATCHTOWER_HTTP_API_TOKEN=<YOUR_API_TOKEN>
```

You can generate a secure token using the command below:

```
tr -dc A-Za-z0-9 </dev/urandom | head -c 64; echo
```

To disable polling mode in this case, make sure the following environment variables are **not** included in your `.env` file:

```env
WATCHTOWER_HTTP_API_PERIODIC_POLLS=true
WATCHTOWER_POLL_INTERVAL=<YOUR_INTERVAL>
```

### Running in Polling Mode

If you'd prefer Watchtower to run in polling mode, periodically checking for new image versions, use the following configuration in your `.env` file:

```env
WATCHTOWER_POLL_INTERVAL=1800  # Poll every 30 minutes
```

To disable HTTP API mode, ensure the following is not present in your `.env`:

```env
WATCHTOWER_HTTP_API=true
WATCHTOWER_HTTP_API_UPDATE=true
```

## Additional Notes

1. This setup uses both HTTP API and polling mode. You can choose one by adjusting the configuration in your `.env` file as shown above.
2. Notifications to Mattermost are optional but can be set up for alerting when containers are updated.

## References

- [AWS ECR Docker Credential Helper](https://github.com/awslabs/amazon-ecr-credential-helper)
- [Watchtower Documentation](https://containrrr.dev/watchtower/)
- [Shoutrrr Notification System](https://containrrr.dev/shoutrrr/)

---

*Maintained by: Taufiq, DevOps Engineer*