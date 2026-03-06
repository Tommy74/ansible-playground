# Generate SSH key if you don't have one
```shell
mkdir .ssh
ssh-keygen -t ed25519 -f ./.ssh/ansible_test -N ""
```
# Build the image
```shell
podman build --no-cache -t ansible-fedora -f Containerfile.SSH .
```
# Run multiple containers as test hosts
```shell
podman network create ansible-fedora-network
```
```shell
podman run --rm -it --name fedora1 -p 2221:22 --network ansible-fedora-network ansible-fedora
```
```shell
podman run --rm -it --name fedora2 -p 2222:22 --network ansible-fedora-network ansible-fedora
```
```shell
podman run --rm -it --name fedora3 -p 2223:22 --network ansible-fedora-network ansible-fedora
```
# [OPTIONAL] Copy any extra public key to each container (password is "ansible")
```shell
ssh-copy-id -i ./.ssh/ansible_test.pub -p 2221 -o StrictHostKeyChecking=no -o IdentitiesOnly=yes ansible@localhost
ssh-copy-id -i ./.ssh/ansible_test.pub -p 2222 -o StrictHostKeyChecking=no -o IdentitiesOnly=yes ansible@localhost
ssh-copy-id -i ./.ssh/ansible_test.pub -p 2223 -o StrictHostKeyChecking=no -o IdentitiesOnly=yes ansible@localhost
```
# test
```shell
ansible all -i inventory.ini -m ping
```

# ssh
```shell
ssh -i ./.ssh/ansible_test -p 2221 -o StrictHostKeyChecking=no -o IdentitiesOnly=yes ansible@localhost
ssh -i ./.ssh/ansible_test -p 2222 -o StrictHostKeyChecking=no -o IdentitiesOnly=yes ansible@localhost
ssh -i ./.ssh/ansible_test -p 2222 -o StrictHostKeyChecking=no -o IdentitiesOnly=yes ansible@localhost
```

# run playbook
```shell
ansible-galaxy collection install community.crypto ansible.posix
ansible-playbook -i inventory.ini playbook.yml
```
after `playbook.yml` has completed successfully, you can ssh from machine to test-machine:
```shell
$ ssh -i ./.ssh/ansible_test -p 2221 -o StrictHostKeyChecking=no -o IdentitiesOnly=yes ansible@localhost
[ansible@8e6c3def7990 ~]$ cat /tmp/test-machines.txt 
CLUSTERING_BAREMETAL_HOST1=10.89.0.18
CLUSTERING_BAREMETAL_HOST2=10.89.0.19
[ansible@8e6c3def7990 ~]$ su - pippo
Password: pippo
[pippo@8e6c3def7990 ~]$ ssh pippo@10.89.0.18
The authenticity of host '10.89.0.18 (10.89.0.18)' can't be established.
ED25519 key fingerprint is SHA256:aqtK02RNWfYU5u3hOs560dRRSf98xoMgDFTrZRl6m5Q.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.89.0.18' (ED25519) to the list of known hosts.
[pippo@20c4d47a4a61 ~]$
```

# tear down
```shell
podman stop fedora1 fedora2 fedora3
```
