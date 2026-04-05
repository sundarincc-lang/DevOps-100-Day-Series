# Phase 1: Foundations (Days 1–20)

## 📚 Introduction
This phase establishes the core skills necessary for DevOps success. You'll build proficiency in Linux system administration, version control with Git, CI/CD pipeline basics, and cloud infrastructure fundamentals.

**Prerequisites:**
- Basic computer literacy
- Willingness to practice hands-on exercises daily
- Approximately 1-2 hours per day

**Phase Goals:**
By the end of this phase, you will:
✅ Navigate and manage Linux systems confidently  
✅ Master Git for version control and collaboration  
✅ Understand CI/CD pipelines and automation  
✅ Get started with cloud infrastructure (AWS)  
✅ Introduce Infrastructure as Code (IaC) concepts

---

## 📅 Day-by-Day Breakdown

### Day 1: Install Linux VM & Learn Basic Commands
**Goal:** Set up a Linux development environment and become comfortable with the command line.

**Activities:**
- Download and install VirtualBox or VMware (free options available)
- Install a Linux distribution (Ubuntu 22.04 LTS recommended for beginners)
- Learn essential commands: `ls`, `cd`, `pwd`, `mkdir`, `touch`, `rm`, `cp`, `mv`
- Practice navigating the file system and creating directories
- Understand the Linux directory structure (`/home`, `/etc`, `/var`, etc.)

**Resources:**
- [Linux Command Line Basics - Ubuntu Official Docs](https://ubuntu.com/tutorials/command-line-for-beginners)
- [Linux Filesystem Hierarchy](https://www.geeksforgeeks.org/linux-file-hierarchy-structure/)

---

### Day 2: Write Your First Shell Script
**Goal:** Automate routine tasks with basic shell scripting.

**Activities:**
- Install a text editor (`nano`, `vim`, or `gedit`)
- Learn shell script basics: variables, loops, conditionals
- Write a script to list files in a directory and output to a file
- Make the script executable with `chmod +x script.sh`
- Execute and debug your first script

**Resources:**
- [Shell Scripting Tutorial for Beginners](https://www.linuxconfig.org/bash-scripting-tutorial-for-beginners)
- [Bash Scripting Guide - GNU](https://www.gnu.org/software/bash/manual/bash.html)

---

### Day 3: Explore File Permissions & Processes
**Goal:** Understand Linux security and process management.

**Activities:**
- Learn the permission model: user (u), group (g), other (o), and read/write/execute (rwx)
- Use `chmod` to modify permissions; use `chown` to change ownership
- Explore the `/etc/passwd` and `/etc/group` files
- Learn process basics: `ps`, `top`, `kill`, `bg`, `fg`
- Monitor running processes and kill unnecessary ones

**Resources:**
- [Linux File Permissions Explained](https://www.linux.com/training-tutorials/understanding-linux-file-permissions/)
- [Linux Process Management](https://www.geeksforgeeks.org/process-management-in-linux/)

---

### Day 4: Networking Basics (IP, DNS, Ports)
**Goal:** Understand networking fundamentals essential for DevOps.

**Activities:**
- Learn IP addressing (IPv4, IPv6) and subnetting basics
- Configure network interfaces on your Linux VM
- Use `ping` to test connectivity
- Explore DNS with `nslookup`, `dig`, and `host` commands
- Understand ports and use `netstat` or `ss` to inspect listening ports
- Learn about common ports (80, 443, 22, 3306, 5432)

**Resources:**
- [Networking Basics - TryHackMe](https://tryhackme.com/room/networkfundamentals)
- [DNS and Domain Names Explained](https://www.cloudflare.com/learning/dns/what-is-dns/)
- [Understanding Ports and Networking](https://www.geeksforgeeks.org/ip-address-and-port-in-networking/)

---

### Day 5: Package Management (apt, yum)
**Goal:** Master package installation and system maintenance.

**Activities:**
- Understand package managers: `apt` (Debian/Ubuntu) and `yum` (RedHat/CentOS)
- Update package lists: `sudo apt update` / `sudo yum update`
- Install packages: `sudo apt install <package>` / `sudo yum install <package>`
- Search for packages: `apt search` / `yum search`
- Remove packages: `sudo apt remove` / `sudo yum remove`
- Practice installing and managing multiple tools

**Resources:**
- [Ubuntu Package Management Guide](https://ubuntu.com/server/docs/package-management)
- [RedHat Package Management](https://access.redhat.com/articles/11258)

---

### Day 6: Install Git & Configure Repositories
**Goal:** Set up Git and understand basic configuration.

**Activities:**
- Install Git: `sudo apt install git` or `sudo yum install git`
- Configure your identity: `git config --global user.name "Your Name"` and `git config --global user.email "your@email.com"`
- Create your first local repository: `git init my-project`
- Understand `.git` directory structure
- Create and stage files: `git add`, `git commit`
- Explore repository history: `git log`

**Resources:**
- [Official Git Documentation](https://git-scm.com/doc)
- [Pro Git Book - Free Online](https://git-scm.com/book/en/v2)

---

### Day 7: Practice Branching & Merging
**Goal:** Master Git workflows for collaborative development.

**Activities:**
- Create branches: `git branch feature-branch` and `git checkout feature-branch`
- Use `git switch` (modern alternative to checkout)
- Make changes on different branches
- Merge branches: `git merge feature-branch`
- Resolve merge conflicts (if any)
- Practice with at least 3-4 branch scenarios

**Resources:**
- [Git Branching Strategy](https://www.atlassian.com/git/tutorials/comparing-workflows)
- [Understanding Git Merge](https://www.atlassian.com/git/tutorials/using-branches/git-merge)

---

### Day 8: Learn Pull Requests & Code Review
**Goal:** Understand collaborative code review practices.

**Activities:**
- Create a GitHub account (if you haven't already)
- Fork a practice repository
- Create a feature branch and make commits
- Open a pull request with a descriptive title and description
- Request code review from peers or practice reviewers
- Address feedback and update your pull request
- Merge the pull request

**Resources:**
- [GitHub Pull Requests Guide](https://docs.github.com/en/pull-requests)
- [Code Review Best Practices](https://www.atlassian.com/blog/bitbucket/code-review-best-practices)

---

### Day 9: Explore Git Workflows (GitFlow)
**Goal:** Learn professional Git workflow patterns.

**Activities:**
- Study GitFlow: `main`, `develop`, `feature/*`, `release/*`, `hotfix/*` branches
- Compare GitFlow with trunk-based development
- Understand when to use GitFlow vs. simpler workflows
- Practice creating feature branches and release branches
- Learn about semantic versioning

**Resources:**
- [GitFlow Cheat Sheet](https://danielkummer.github.io/git-flow-cheatsheet/)
- [A Successful Git Branching Model](https://nvie.com/posts/a-successful-git-branching-model/)

---

### Day 10: Build a Small Project with GitHub
**Goal:** Apply Git knowledge to a real project.

**Activities:**
- Create a simple project (e.g., a bash utility, Python script, or documentation)
- Initialize a Git repository and push to GitHub
- Create multiple branches and pull requests
- Add a README with project description and setup instructions
- Add a `.gitignore` file
- Practice committing frequently with meaningful messages
- Invite a peer to review your code

**Resources:**
- [GitHub Getting Started](https://docs.github.com/en/get-started)
- [Creating a Good README](https://www.makeareadme.com/)

---

### Day 11: Introduction to CI/CD Concepts
**Goal:** Understand the principles behind continuous integration and deployment.

**Activities:**
- Learn CI/CD core concepts: automation, testing, deployment
- Understand the benefits: faster feedback, reduced errors, rapid deployment
- Explore the CI/CD pipeline stages: Build → Test → Deploy
- Study why CI/CD matters in DevOps
- Review real-world CI/CD examples and use cases

**Resources:**
- [CI/CD Explained - Red Hat](https://www.redhat.com/en/topics/devops/what-is-ci-cd)
- [Getting Started with CI/CD](https://www.atlassian.com/continuous-delivery/continuous-integration/continuous-integration-intro)

---

### Day 12: Create a Simple CI Pipeline with GitHub Actions
**Goal:** Build your first automated pipeline.

**Activities:**
- Explore GitHub Actions dashboard in your repository
- Create a workflow file (`.github/workflows/main.yml`)
- Write a simple workflow that runs on push/pull request
- Add a basic job: lint and build steps
- Test the workflow by pushing changes to GitHub
- View workflow execution logs and debug

**Resources:**
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [GitHub Actions Getting Started](https://docs.github.com/en/actions/quickstart)

---

### Day 13: Automate Builds with Jenkins
**Goal:** Learn Jenkins for build automation.

**Activities:**
- Install Jenkins locally or use a cloud instance
- Access the Jenkins web interface
- Create a freestyle job that clones a repository and builds it
- Configure build triggers (poll SCM or webhook)
- Add build steps (shell commands)
- View build logs and history

**Resources:**
- [Jenkins Official Documentation](https://www.jenkins.io/doc/)
- [Jenkins Getting Started Guide](https://www.jenkins.io/doc/book/getting-started/)

---

### Day 14: Learn Artifact Management Basics
**Goal:** Understand how to store and manage build artifacts.

**Activities:**
- Learn artifact concepts: binaries, packages, dependencies
- Explore artifact repositories: Artifactory, Nexus, AWS S3
- Practice uploading and retrieving artifacts
- Understand versioning and retention policies
- Learn about container registries (Docker Hub, ECR basics)

**Resources:**
- [Artifact Management Best Practices](https://www.sonatype.com/blog/what-is-artifact-management)
- [JFrog Artifactory Documentation](https://jfrog.com/artifactory/)

---

### Day 15: Explore Testing Integration in Pipelines
**Goal:** Integrate automated testing into your CI pipeline.

**Activities:**
- Write simple unit tests (using any language you prefer)
- Learn test types: unit, integration, smoke tests
- Add test execution to your GitHub Actions workflow
- Configure test reporting and coverage
- Practice test-driven development (TDD) basics
- Understand how tests prevent bugs in production

**Resources:**
- [Testing Best Practices](https://testingjavascript.com/)
- [Automated Testing in CI/CD Pipelines](https://www.atlassian.com/continuous-delivery/ci-cd-testing)

---

### Day 16: Introduction to AWS Free Tier
**Goal:** Get started with cloud infrastructure.

**Activities:**
- Create an AWS account and explore the Free Tier
- Set up billing alerts to avoid unexpected charges
- Explore the AWS Management Console
- Learn about key AWS services: EC2, S3, IAM, RDS
- Understand AWS regions and availability zones

**Resources:**
- [AWS Free Tier Overview](https://aws.amazon.com/free/)
- [AWS Getting Started](https://aws.amazon.com/getting-started/)

---

### Day 17: Launch EC2 Instance & Connect via SSH
**Goal:** Manage cloud compute resources.

**Activities:**
- Create an EC2 instance (t2.micro for free tier)
- Generate or import an SSH key pair
- Configure security groups for SSH access (port 22)
- Connect to the instance via SSH: `ssh -i key.pem ec2-user@<instance-ip>`
- Install software on the instance
- Understand lifecycle: launch, stop, terminate

**Resources:**
- [AWS EC2 Getting Started](https://docs.aws.amazon.com/ec2/index.html)
- [SSH Key Pair Guide](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html)

---

### Day 18: Explore IAM Roles & Policies
**Goal:** Understand cloud security and access management.

**Activities:**
- Create an IAM user and access keys
- Explore IAM policies and permissions
- Assign roles to EC2 instances
- Practice the principle of least privilege
- Create custom policies for specific use cases
- Understand identity-based vs. resource-based policies

**Resources:**
- [AWS IAM User Guide](https://docs.aws.amazon.com/iam/)
- [IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)

---

### Day 19: Learn S3 Basics (Storage)
**Goal:** Master cloud object storage.

**Activities:**
- Create an S3 bucket
- Upload and download objects
- Understand bucket policies and access control
- Explore versioning and lifecycle policies
- Learn about S3 storage classes
- Practice using AWS CLI to interact with S3: `aws s3 ls`, `aws s3 cp`

**Resources:**
- [AWS S3 User Guide](https://docs.aws.amazon.com/s3/)
- [S3 Getting Started](https://docs.aws.amazon.com/AmazonS3/latest/userguide/GetStartedWithS3.html)

---

### Day 20: Introduction to Infrastructure as Code (IaC)
**Goal:** Learn to manage infrastructure through code.

**Activities:**
- Understand IaC concepts and benefits
- Explore tools: Terraform, CloudFormation, Ansible
- Write a simple Terraform configuration to create an S3 bucket or EC2 instance
- Learn HCL (HashiCorp Configuration Language) basics
- Apply and destroy infrastructure with Terraform
- Understand version control for infrastructure code

**Resources:**
- [Infrastructure as Code Explained](https://www.hashicorp.com/resources/what-is-infrastructure-as-code)
- [Terraform Getting Started](https://developer.hashicorp.com/terraform/tutorials/aws-get-started)
- [Terraform AWS Provider Documentation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)

---

## 🎯 Phase 1 Summary

You've built a **solid foundation** in:
- ✅ Linux system administration and command-line proficiency
- ✅ Version control with Git and collaborative workflows
- ✅ CI/CD pipeline concepts and practical implementation
- ✅ Cloud infrastructure basics with AWS
- ✅ Introduction to Infrastructure as Code

**Next Steps:**
Ready to level up? Move to **Phase 2** to dive deeper into containerization (Docker), orchestration (Kubernetes), monitoring, logging, and advanced infrastructure practices.

**Tips for Success:**
- 🔁 **Practice consistently**: Dedicate 1-2 hours daily to hands-on exercises
- 📝 **Document your learning**: Create a personal wiki or blog post for each topic
- 🤝 **Join communities**: Participate in DevOps forums and communities
- 💾 **Keep your code**: Push all projects to GitHub for portfolio building
- 🔄 **Revisit basics**: Don't skip fundamentals—they're essential for advanced topics