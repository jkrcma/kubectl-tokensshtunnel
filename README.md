# kubectl-tokensshtunnel

## Installation

To install it you just need to copy `kubectl-tokensshtunnel` anywhere within your `PATH` with execution permissions.

## Configuration

The script supports the following command-line options:

- `-c <ssh command>`: Set the SSH command to use. This is the command used to establish an SSH connection to the remote server.
- `-k <kube config>` (optional): Set the remote kube config file location. Default: `/etc/rancher/k3s/k3s.yaml`.
- `-L <ssh tunnel>` (optional): Set the SSH tunnel configuration. The format is `[<local_bind>:]<local_port>:<remote_host>:<remote_port>`. Use this option to forward a local port to the remote Kubernetes API server.
- `-n` (optional): Hop to `<remote_host>` to fetch the kube config file. Useful when you connect from your bastion to a remote Kubernetes API server.
- `-s` (optional): Add `sudo` to the SSH command. Use this option if `sudo` access is required to read the kube config file.
- `-t <tmp dir>` (optional): Set the location to store the cached credentials. This is the location where the generated Kubernetes config files will be stored.
- `-d <seconds>` (optional): Set the duration for which the SSH tunnel will be available. Default: 3600.

## Kubeconfig

To configure your kubeconfig file, add the following configuration to the users section, update accordingly to your needs:

```yaml
- name: sshtunnel
  user:
    exec:
      apiVersion: client.authentication.k8s.io/v1beta1
      args:
      - tokensshtunnel
      - -c
      - ssh bastion
      - -s
      - -L
      - 127.0.0.1:7443:127.0.0.1:6443
      - -k
      - /etc/rancher/k3s/k3s.yaml
      command: kubectl
      env: null
      interactiveMode: IfAvailable
      provideClusterInfo: false
```

## Cluster config extension

Hardcoding tunnel ports with `-L` requires to define a user per each cluster. If you only use administrator accounts all users are nearly identical (user `kubernetes-admin`), you can move the SSH tunnel host and port definition into the config extension:

```yaml
- cluster:
    certificate-authority-data: <ohlongjohnson>
    extensions:
    - name: client.authentication.k8s.io/exec
      extension:
        ssh-tunnel-to: kubemaster1.example.com:6443
    server: https://127.0.0.1:55443
  name: kubemaster1
```

In the user definition, skip the entire `-L <value>` part.
