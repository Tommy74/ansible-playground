## Ansible Playground

This project demonstrates:

1. Creating some Podman images to be used as Ansible hosts: this image is pre-configured in order to SSH into it using a local Private Key
2. Running an Ansible Playbook that creates a new SSH Private Key on one Ansible host and distributes the corresponding SSH Public Key to the other Ansible hosts

## Generate SSH key if you don't have one
```shell
mkdir .ssh
ssh-keygen -t ed25519 -f ./.ssh/ci_test -N ""
```
## Build the image
```shell
podman build --no-cache -t ansible-fedora -f Containerfile.SSH .
```
## Run multiple containers as test hosts
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

## test
```shell
ansible all -i inventory.ini -m ping
```

## ssh
```shell
ssh -i ./.ssh/ci_test -p 2221 -o StrictHostKeyChecking=no -o IdentitiesOnly=yes ci@localhost
ssh -i ./.ssh/ci_test -p 2222 -o StrictHostKeyChecking=no -o IdentitiesOnly=yes ci@localhost
ssh -i ./.ssh/ci_test -p 2222 -o StrictHostKeyChecking=no -o IdentitiesOnly=yes ci@localhost
```

## run playbook

```shell
ansible-galaxy collection install community.crypto ansible.posix
```

```shell
ansible-playbook -i inventory.ini playbook.yml
```

after `playbook.yml` has completed successfully, you can ssh from machine to test_machine:
```shell
$ ssh -i ./.ssh/ci_test -p 2221 -o StrictHostKeyChecking=no -o IdentitiesOnly=yes ci@localhost
[ci@8819c6809523 ~]$ cat test-machines.txt 
CLUSTERING_BAREMETAL_HOST1=10.89.0.6
CLUSTERING_BAREMETAL_HOST2=10.89.0.7
[ci@8819c6809523 ~]$ . test-machines.txt
[ci@8819c6809523 ~]$ ssh $CLUSTERING_BAREMETAL_HOST1
[ci@409ad936178f ~]$ ip a | grep '10.89.0.6'
    inet 10.89.0.6/24 brd 10.89.0.255 scope global eth0
```

## [OPTIONAL] Copy any extra public key to each container (password is "ci")
```shell
ssh-copy-id -i ./.ssh/ci_test.pub -p 2221 -o StrictHostKeyChecking=no -o IdentitiesOnly=yes ci@localhost
ssh-copy-id -i ./.ssh/ci_test.pub -p 2222 -o StrictHostKeyChecking=no -o IdentitiesOnly=yes ci@localhost
ssh-copy-id -i ./.ssh/ci_test.pub -p 2223 -o StrictHostKeyChecking=no -o IdentitiesOnly=yes ci@localhost
```

## tear down
```shell
podman stop fedora1 fedora2 fedora3
```
