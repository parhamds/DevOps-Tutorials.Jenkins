# Jenkins Guide

This guide provides step-by-step instructions for setting up and managing various DevOps tasks using Jenkins and Docker. Follow the sections below to efficiently configure your environment.

## Java Installation

To ensure compatibility with Jenkins and other tools, install Java 17 by running the following commands:

```bash
apt install openjdk-17-jdk openjdk-17-jre
java --version
```

## Jenkins Setup

Follow these steps to set up Jenkins on Ubuntu:

1. Check Jenkins status:
   ```bash
   systemctl status jenkins
   ```

2. Enable Jenkins on system boot:
   ```bash
   systemctl enable jenkins
   ```

3. Access the Jenkins web portal on port 8080 and complete the initial setup. Retrieve the initial admin password using:
   ```bash
   cat /var/lib/jenkins/secrets/initialAdminPassword
   ```

4. Select suggested plugins and finish the setup process.

## Building and Pushing Docker Images

Refer to page 6 of Master DevOps for detailed instructions on building and pushing Docker images.

## New Node Setup

For adding a new node, follow these steps:

1. SSH to the node and add the Jenkins user to sudoers:
   ```bash
   vim /etc/sudoers
   ```
   Add the following line:
   ```
   jenkins ALL=(ALL) NOPASSWD: ALL
   ```

2. Create a pipeline job and modify the script as per instructions in "71. Add New Node" in the Master DevOps document.

## Pipeline Examples

### Input Example

```groovy
stage('Deploy') {
    input {
        message 'Deploy?'
        ok 'Do it!'
        parameters {
            string(name: 'TARGET_ENVIRONMENT', defaultValue: 'PROD', description: 'Target deployment environment')
        }
    }
    steps {
        echo "Deploying release ${RELEASE} to environment ${TARGET_ENVIRONMENT}"
    }
}
```

### Parameter Example

```groovy
pipeline {
    agent any
    parameters {
        booleanParam(name: 'RELEASE', defaultValue: false, description: 'Target deployment environment')
    }
    stages {
        // Define stages here
    }
}
```

## Connecting Slave to Master

Follow these steps to connect a slave to the master:

1. On the master:
   ```bash
   sudo -iu jenkins
   ssh-keygen -t rsa
   cat /var/lib/jenkins/.ssh/ip_rsa # Copy the private key
   cat /var/lib/jenkins/.ssh/ip_rsa.pub # Copy the public key
   ```

2. On the slave:
   ```bash
   ssh ubuntu@<ip_of_slave>
   sudo su
   mkdir /root/.ssh
   vim /root/.ssh/authorized_keys # Add the public key of the master here
   ```

3. Test SSH connection from master to slave.

## Jenkins Agents on Docker

To configure Jenkins agents on Docker, follow these steps:

1. Disable firewall:
   ```bash
   sudo systemctl disable firewalld
   ```

2. Enable Docker:
   ```bash
   sudo systemctl enable --now docker
   sudo usermod -aG docker jenkins
   sudo chmod 666 /var/run/docker.sock
   ```

3. Execute Jenkins master Docker:
   ```bash
   docker run -d -u root --privileged=true --volume /root/jenkins_home:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock -p 8080:8080 -p 50000:50000 --name jenkins-centos jenkins/jenkins:lts
   ```

## Additional Configurations

### Jenkins Integration with GitHub via SSH

For Jenkins integration with GitHub using SSH:

1. Generate an SSH key pair and add the public key to your GitHub account.

2. Enter the Git repo URL in the Jenkins job configuration.

3. Add credentials (SSH username with private key) in Jenkins.

4. SSH to Jenkins and verify the connection:
   ```bash
   sudo su
   sudo - jenkins
   git ls-remote -h git@<git_repo_clone_url> HEAD
   ```

### Auto-Start Job with GitHub Webhooks

1. Allow port 8080 from anywhere on Jenkins security group.

2. On GitHub:
   - Go to repository settings -> Webhooks -> Add webhook.
   - Set URL as `http://<public_ip_of_jenkins>:8080/github-webhook/`, content type as JSON, and select events to trigger webhooks (e.g., push).

3. Enable "GitHub Hook trigger for GITScm polling" in Jenkins job settings.

Enjoy streamlined DevOps workflows with Jenkins and Docker!