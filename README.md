# kolla-ansible-CMT371S

## Introduction

Kolla-Ansible is a tool for deploying OpenStack services in Docker containers, using Ansible for automation. It simplifies the complexity of OpenStack deployments, making upgrades and management easier, and is widely used in production environments.

---

## Guide: Installing Kolla-Ansible on AWS EC2 (t3.large)

### Prerequisites

- AWS account with permissions to launch EC2 instances.
- EC2 t3.large instance (2 vCPUs, 8GB RAM) running Ubuntu 22.04 LTS.
- Security group allowing SSH (port 22) and required OpenStack ports.
- SSH key pair for instance access.

### 1. Launch EC2 Instance

- Go to AWS Console → EC2 → Launch Instance.
- Choose Ubuntu 22.04 LTS AMI.
- Select t3.large instance type.
- Configure storage (minimum 40GB recommended).
- Set up security group:
  - Allow SSH (port 22) from your IP.
  - Allow necessary ports for OpenStack (e.g., 80, 443, 5000, 8774, etc.).
- Launch the instance using a key pair.

### 2. Initial Server Setup

SSH into your instance:
```bash
ssh -i "your-key.pem" ubuntu@<EC2-PUBLIC-IP>
```

Update and install dependencies:
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y python3-pip python3-dev libffi-dev gcc libssl-dev python3-venv git
sudo apt install -y docker.io
sudo systemctl enable --now docker
sudo usermod -aG docker $USER
```
**Logout and login again to apply Docker permissions.**

### 3. Install Ansible and Kolla-Ansible

```bash
pip3 install --user ansible
pip3 install --user 'kolla-ansible==18.1.0'
export PATH=$PATH:~/.local/bin
```

### 4. Prepare Kolla-Ansible Directories

```bash
sudo mkdir -p /etc/kolla
sudo chown $USER:$USER /etc/kolla
cp -r ~/.local/share/kolla-ansible/etc_examples/kolla/* /etc/kolla
cp ~/.local/share/kolla-ansible/ansible/inventory/all-in-one ./inventory
```

### 5. Configure Kolla-Ansible

Generate passwords:
```bash
kolla-genpwd
```

Edit `/etc/kolla/globals.yml`:
```yaml
kolla_base_distro: "ubuntu"
kolla_install_type: "source"
openstack_release: "zed" # Or your desired version
network_interface: "ens5" # Check your network interface with `ip addr`
neutron_external_interface: "eth1" # Attach and configure a second ENI in AWS
```

**Note:** You may need to attach a second network interface in AWS and configure it on the instance for external networking.

### 6. Bootstrap Servers

```bash
kolla-ansible -i ./inventory bootstrap-servers
```

### 7. Prechecks

```bash
kolla-ansible -i ./inventory prechecks
```

### 8. Deploy OpenStack

```bash
kolla-ansible -i ./inventory deploy
```

### 9. Post-Deployment

Initialize OpenStack:
```bash
kolla-ansible post-deploy
```

Install OpenStack CLI:
```bash
pip3 install python-openstackclient
```

Source the admin credentials:
```bash
source /etc/kolla/admin-openrc.sh
```

Test with:
```bash
openstack service list
```

---

## Project Rubric

| Criteria                      | Excellent (5)         | Good (4)          | Satisfactory (3)   | Needs Improvement (1-2) |
|-------------------------------|-----------------------|-------------------|--------------------|-------------------------|
| **Documentation**             | Clear, comprehensive, step-by-step; all commands and configs provided | Minor omissions, mostly clear | Some steps unclear or missing | Unclear, incomplete, confusing |
| **Configuration**             | Correctly configures network, security, and OpenStack options | Minor configuration errors | Major config errors but works | Fails to configure correctly |
| **Deployment Success**        | Fully working OpenStack, all services running | Most services running, minor issues | Some major services missing | Deployment fails or unusable |
| **Testing & Validation**      | Successfully runs OpenStack CLI commands, demonstrates access | Partial testing, some issues | Minimal testing or unclear | No testing or access shown |
| **Security Best Practices**   | Proper SSH keys, secure group, least privilege | Minor security gaps | Some security issues | Unsafe defaults, no security |
| **Presentation & Clarity**    | Well organized, easy to follow, professional | Mostly organized | Some structure, minor clarity issues | Disorganized, hard to follow |

---

## References

- [Kolla-Ansible Docs](https://docs.openstack.org/kolla-ansible/latest/user/quickstart.html)
- [OpenStack Docs](https://docs.openstack.org/)
- [AWS EC2 User Guide](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/)
