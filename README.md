# Git Lab 1

---

# Lab 2 — Docker Compose, KVM and Ansible

## Application

The application consists of two services:

- frontend — Nginx
- backend — Python Flask

The frontend sends an HTTP request to `/api/message`.
Nginx proxies this request to the backend container, which returns:

```text
Hello from backend!
```

## Local Docker Compose

Start the application locally:

```bash
docker compose up --build -d
```

Check the containers:

```bash
docker compose ps
```

The local application is available at:

```text
http://localhost:8080
```

## Architecture

```text
Laptop (Windows)
        |
        | http://localhost:8081
        v
WSL 2
        |
        | QEMU + KVM
        v
Ubuntu VM
        |
        | Docker Compose
        |
        +-- frontend (Nginx)
        |       |
        |       +-- /api/message
        |               |
        |               v
        +-- backend (Flask)
```

KVM stands for Kernel-based Virtual Machine.

WSL stands for Windows Subsystem for Linux.

VM stands for Virtual Machine.

## KVM verification

KVM availability inside WSL can be checked with:

```bash
ls -l /dev/kvm
```

CPU virtualization support can be checked with:

```bash
grep -E -c '(vmx|svm)' /proc/cpuinfo
```

Inside the virtual machine, the virtualization technology can be checked with:

```bash
systemd-detect-virt
```

Expected result:

```text
kvm
```

## Starting the KVM virtual machine

The VM was created manually inside WSL using QEMU and KVM.

Start it in the background:

```bash
cd ~/lab2-vm

qemu-system-x86_64 \
  -enable-kvm \
  -cpu host \
  -m 2048 \
  -smp 2 \
  -drive file=lab2.qcow2,format=qcow2 \
  -drive file=seed.img,format=raw \
  -nic user,hostfwd=tcp::2222-:22,hostfwd=tcp::8081-:8080 \
  -display none \
  -daemonize
```

Port forwarding:

```text
localhost:2222 -> VM:22
localhost:8081 -> VM:8080
```

SSH connection to the VM:

```bash
ssh -i ~/.ssh/lab2_ed25519 -p 2222 lab@localhost
```

## Ansible

Ansible connects to the VM using SSH.

Check the connection:

```bash
ansible -i ansible/inventory.ini lab_vm -m ping
```

Deploy the application:

```bash
ansible-playbook -i ansible/inventory.ini ansible/playbook.yml
```

The playbook:

- installs Docker and Docker Compose;
- starts and enables Docker;
- creates the application directory;
- copies the frontend, backend and Compose configuration;
- starts the application using Docker Compose.

The playbook can be executed repeatedly without manual correction of the VM state.

After deployment, the application is available from the laptop at:

```text
http://localhost:8081
```

## Frontend and backend communication

To verify communication between the frontend and backend, open:

```text
http://localhost:8081
```

The page displays:

```text
Hello from backend!
```

This text is not stored in the frontend page. It is returned by the Flask backend through the `/api/message` HTTP endpoint.

## Trivy security scan

GitHub Actions builds the Docker images and scans them with Trivy.

Trivy scans the images for known vulnerabilities.

The workflow is located at:

```text
.github/workflows/trivy.yml
```

The scan results are visible in the GitHub Actions workflow logs.
