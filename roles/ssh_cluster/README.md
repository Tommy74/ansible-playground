# ssh_cluster

Configures SSH key-based authentication from the controller node (`machine` group) to the worker nodes (`test_machine` group) so that the controller can access them without password prompts.

It also creates a `test-machines.txt` file on the controller node listing the IP addresses of all worker nodes and the path to the SSH private key used to connect to them, formatted as environment variable assignments (e.g. `CLUSTERING_BAREMETAL_HOST1=10.0.0.1`, `CLUSTERING_BAREMETAL_SSH_PRIVATE_KEY=/home/user/.ssh/id_internal_ed25519`).

## What it does

1. **Generates an SSH key pair** on the controller node (ed25519 by default).
2. **Adds the public key** to `authorized_keys` on each worker node.
3. **Configures `~/.ssh/config`** on the controller so it uses the generated key when connecting to worker nodes.
4. **Creates `~/test-machines.txt`** on the controller with one line per worker node and the path to the SSH private key.

## Default variables

| Variable | Default | Description |
|---|---|---|
| `controller_nodes_group_name` | `machine` | Inventory group for the controller node |
| `worker_nodes_group_name` | `test_machine` | Inventory group for the worker nodes |
| `ssh_key_user` | `{{ ansible_env.USER }}` | SSH key owner |
| `ssh_key_type` | `ed25519` | Key type |
| `worker_nodes_list_file_path` | `~/test-machines.txt` | Path to the generated worker list file |
| `worker_nodes_list_file_env_variable_prefix` | `CLUSTERING_BAREMETAL_HOST` | Prefix for each worker node line in the worker list file |
| `worker_nodes_list_file_ssh_key_variable` | `CLUSTERING_BAREMETAL_SSH_PRIVATE_KEY` | Variable name for the SSH private key path in the worker list file |
