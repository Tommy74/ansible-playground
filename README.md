# Generate SSH key if you don't have one
```shell
mkdir .ssh
ssh-keygen -t ed25519 -f ./.ssh/ansible_test -N ""
```
# Build the image
```shell
podman build -t ansible-fedora -f Containerfile.SSH .
```
# Run multiple containers as test hosts
```shell
podman run --rm -it --name fedora1 -p 2221:22 ansible-fedora
```
```shell
podman run --rm -it --name fedora2 -p 2222:22 ansible-fedora
```
```shell
podman run --rm -it --name fedora3 -p 2223:22 ansible-fedora
```
# Copy any extra public key to each container (password is "ansible")
```shell
ssh-copy-id -i ./.ssh/ansible_test.pub -p 2221 -o StrictHostKeyChecking=no -o IdentitiesOnly=yes ansible@localhost
ssh-copy-id -i ./.ssh/ansible_test.pub -p 2222 -o StrictHostKeyChecking=no -o IdentitiesOnly=yes ansible@localhost
ssh-copy-id -i ./.ssh/ansible_test.pub -p 2223 -o StrictHostKeyChecking=no -o IdentitiesOnly=yes ansible@localhost
```
# test
```shell
ansible all -i inventory.ini -m ping
```
# run playbook
```shell
ansible-playbook -i inventory.ini playbook.yml
```
# tear down
```shell
podman stop fedora1 fedora2 fedora3
```
