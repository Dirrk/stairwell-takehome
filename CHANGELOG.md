# Setup entrypoint
* Copied some of my makefile over to build docker into my ecr repository
* In my separate terraform repo I created an ecr repository + route53 entry for stairwell that cnamed back to my home network
```
diff --git a/aws/dns/home.tf b/aws/dns/home.tf
index 21cbd05..8dd592d 100644
--- a/aws/dns/home.tf
+++ b/aws/dns/home.tf
@@ -13,6 +13,7 @@ locals {
     "valheim",
     "request",
     "auth",
+    "stairwell",
   ]
 }

diff --git a/aws/ecr/main.tf b/aws/ecr/main.tf
index ea6f6c4..e7b4c7d 100644
--- a/aws/ecr/main.tf
+++ b/aws/ecr/main.tf
@@ -32,6 +32,7 @@ output "repos" {
 locals {
   app_repos = toset([
     "app/tradier-collector",
+    "app/stairwell-entrypoint",
     "network/unifi-exporter",
     "media/plex",
     "media/plex-exporter",
```
* Modified the entrypoint yaml to use my repository and corrected the service type
* Added tls certificate to support stairwell.rada.world
* Added Traefik route to expose the service via the Traefik / Metal LB setup

# Setup generator
* Created systemd service, then enabled and started the service on worker-stairwell

# Setup proxmox vm for worker-stairwell
* Manually created 1 vm from my ubuntu20 cloudinit image in proxmox
* Ran the ansible playbook `qemu-setup-vm.yaml` from my main repo that installs the ensures qemu guest agent is installed/running/sets the hostname to the desired hostname, configures my user and ssh keys. Example output provided below

```
$ ansible-playbook -i hosts qemu-setup-vm.yml --limit worker-stairwell

PLAY [qemu-vms]

TASK [Gathering Facts] ok: [worker-stairwell]
TASK [start qemu agent] changed: [worker-stairwell]
TASK [set the hostname] changed: [worker-stairwell]
TASK [overwrite cloudcfg] changed: [worker-stairwell]
TASK [linux-server : include_vars] ok: [worker-stairwell]
TASK [linux-server : include_tasks] included: /mnt/c/Users/derek/code/terraform/ansible/roles/linux-server/tasks/ssh_keys.yml for worker-stairwell
TASK [linux-server : install ssh] ok: [worker-stairwell]
TASK [linux-server : enable ssh] ok: [worker-stairwell]
TASK [linux-server : Create user derek] ok: [worker-stairwell]
TASK [linux-server : Add me to the sudooers] ok: [worker-stairwell]
TASK [linux-server : Configure ssh keys] ok: [worker-stairwell]
TASK [linux-server : Configure ssh keys] ok: [worker-stairwell]
TASK [linux-server : install required utils] ok: [worker-stairwell]
TASK [linux-server : install optional utils] ok: [worker-stairwell]
TASK [linux-server : install qemu guest agent] ok: [worker-stairwell]
TASK [linux-server : update the kernel modules that load on start] ok: [worker-stairwell]
TASK [linux-server : update grub config] ok: [worker-stairwell]
TASK [linux-server : disable graphics drivers on vm hosts to allow pcie passthrough] skipping: [worker-stairwell]
TASK [linux-server : configure cloud-init for proxmox] ok: [worker-stairwell]
TASK [reboot] changed: [worker-stairwell]

PLAY RECAP worker-stairwell           : ok=14   changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
