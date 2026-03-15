# DevOps Demo: GitHub Actions & EC2 Deployment

This repository contains a simple HTML demo page that displays "Hello from GitHub Actions & EC2!". Follow these steps to deploy it to an AWS EC2 instance.

## Prerequisites

- AWS account with EC2 access
- Local SSH key pair (or ability to create one)
- Basic knowledge of AWS console and command line
- GitHub repository with this code (for automated deployment via GitHub Actions)

## Step 1: Create an EC2 Instance

1. Log in to the [AWS Management Console](https://console.aws.amazon.com/).
2. Navigate to **EC2** > **Instances** > **Launch instances**.
3. Select an Amazon Machine Image (AMI):
   - Choose **Amazon Linux 2** or **Ubuntu Server 22.04 LTS** for simplicity.
4. Pick an instance type:
   - `t3.micro` is suitable for this demo and qualifies for free tier.
5. Configure instance details:
   - Leave defaults unless you have specific requirements.
6. Add storage:
   - Default 8GB is sufficient for this static HTML demo.
7. Add tags:
   - Key: `Name`, Value: `DevOps-Demo-EC2`
8. Configure security group:
   - Create a new security group or use existing.
   - Add rules:
     - **SSH** (port 22) - Source: Your IP address (use "My IP" for security)
     - **HTTP** (port 80) - Source: `0.0.0.0/0` (for web access)
9. Review and launch the instance.
10. Create or select a key pair:
    - Download the `.pem` file to your local machine (e.g., `~/Downloads/devops-demo-key.pem`)
    - Keep this file secure!

11. Wait for the instance to be in "running" state and note the **Public IPv4 address**.

## Step 2: Add Your Local SSH Public Key to the EC2 Instance

To use your own SSH key instead of the AWS-generated one:

### 2.1 Generate a local SSH key pair (if you don't have one)

On Windows (using PowerShell or Git Bash):

```bash
ssh-keygen -t rsa -b 4096 -C "your-email@example.com"
```

This creates:
- Public key: `~/.ssh/id_rsa.pub`
- Private key: `~/.ssh/id_rsa`

### 2.2 Connect to EC2 using the AWS key pair

```bash
ssh -i ~/Downloads/devops-demo-key.pem ec2-user@<EC2_PUBLIC_IP>
```

> Note: Use `ubuntu@` if using Ubuntu AMI instead of `ec2-user@`.

### 2.3 Add your local public key to the instance

On the EC2 instance:

```bash
# Create .ssh directory if it doesn't exist
mkdir -p ~/.ssh
chmod 700 ~/.ssh

# Add your public key to authorized_keys
echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC..." >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

Replace the example key with the actual content from your `~/.ssh/id_rsa.pub` file.

### 2.4 Test connection with your local key

From your local machine:

```bash
ssh -i ~/.ssh/id_rsa ec2-user@<EC2_PUBLIC_IP>
```

## Step 3: Deploy the Code

### 3.1 Set up GitHub Actions for automated deployment

This repository includes a GitHub Actions workflow (`.github/workflows/main.yml`) that automatically deploys your code to EC2 on every push to the `main` branch.

#### Configure GitHub Secrets

In your GitHub repository, go to **Settings** > **Secrets and variables** > **Actions** and add the following secrets:

- `EC2_HOST`: Your EC2 instance's public IP address or DNS name
- `EC2_USER`: The SSH username (e.g., `ec2-user` for Amazon Linux, `ubuntu` for Ubuntu)
- `EC2_SSH_KEY`: Your private SSH key content (the entire content of `~/.ssh/id_rsa`)

#### Deploy via Git

Instead of manual file copying, push your changes to the `main` branch:

```bash
git add .
git commit -m "Update HTML content"
git push origin main
```

The GitHub Actions workflow will automatically:
1. Copy all files to `/home/{EC2_USER}/app` on your EC2 instance
2. Install and configure Nginx web server
3. Copy `index.html` to `/var/www/html/`
4. Restart Nginx to serve the updated content

### 3.2 Set up the web server (done automatically by GitHub Actions)

The GitHub Actions workflow handles web server setup automatically by installing Nginx and copying files to the web root.

### 3.3 Verify the deployment

Open your web browser and navigate to:
```
http://<EC2_PUBLIC_IP>/
```

You should see the page displaying "Hello from GitHub Actions & EC2!".

## Next Steps

- **Monitor deployments**: Check the "Actions" tab in your GitHub repository to see deployment status
- Set up a custom domain name and SSL certificate
- Add monitoring and logging for the EC2 instance
- Implement additional CI/CD features (testing, staging environments)

## Troubleshooting

- **SSH connection issues**: Check security group rules and ensure the key file permissions are correct (`chmod 400 key.pem`)
- **Web server not accessible**: Verify port 80 is open in security group and the service is running
- **Permission denied**: Ensure you're using the correct username (`ec2-user` for Amazon Linux, `ubuntu` for Ubuntu)

## Cleanup

When done with the demo:
1. Stop the EC2 instance to avoid charges
2. Terminate the instance if no longer needed
3. Delete the key pair from AWS if created just for this demo