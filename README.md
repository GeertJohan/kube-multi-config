# kube-multi-config

A simple, secure and effective method to manage kube config files. Without installing anything.

> Some backstory:
>
> My `.kube/config` file contained hundreds of lines of yaml. It became a pain to edit the file. I wanted to simplify this without installing any tools to manage the file. It should be possible to add/remove kubernetes contexts manually.

This walkthrough helps you setup a simple structure to manage large number of kubernetes contexts.

## How it works

The `KUBECONFIG` environment variable allows `kubectl` users to list individual kube config files, separated by a semicolon. This is the feature that you'll use to separate the single `.kube/config` file into multiple config files. Each config file can still contain more than one cluster/context/user, it's up to you how to organize these.

A script will automatically update the `KUBECONFIG` environment variable so that you don't have to synchronize the list of configs and the `KUBECONFIG` variable pointing to those configs.

## Setup

Lets start by creating some directories

```bash
mkdir ~/.kube-multi-config
# The config.d folder will contain the kubeconfig files.
mkdir ~/.kube-multi-config/config.d
```

Next, let's create a bash/zsh compatible script to compose the `KUBECONFIG` environment variable.

```bash
nano ~/.kube-multi-config/export-kubeconfig
```

(Or use your favorite editor to create this file.)

Add this code:

```bash
# List of files in config.d
kubeConfigFileList=$(find ${HOME}/.kube-multi-config/config.d -type f)

# Always include the original kubeconfig file for backwards compatibility. Also,
# some tools (for example `gcloud`) add kube contexts directly to
# `.kube/config`.
KUBECONFIG="${HOME}/.kube/config"

# Combine all file paths into the single `KUBECONFIG` variable.
while IFS= read -r kubeConfigFile; do
  KUBECONFIG+=":${kubeConfigFile}"
done <<< ${kubeConfigFileList}

# Export the `KUBECONFIG` variable so that it can be 'seen' by commands
# following this script.
export KUBECONFIG
```

Note that the file doesn't have to be executable.

To run this script every time you open a terminal/shell, add it to `~/.bashrc` or `~/.zshrc` (depending on which shell you use) using the `source` builtin.

```bash
source ${HOME}/.kube-multi-config/export-kubeconfig
```

Now add a config file to the `config.d` folder.

```bash
cd ~/.kube-multi-config/config.d
touch my-project
chmod 600 my-project
nano my-project
```

Add a config file, it could look something like this:

```yaml
apiVersion: v1
kind: Config

clusters:
- name: my-project_cluster
  cluster:
    certificate-authority-data: <ca blob>
    server: https://<server address>

users:
- name: my-project_admin
  user:
    token: <secret token>

contexts:
- name: my-project
  context:
    cluster: my-project_cluster
    user: my-project_admin
```

Feel free to copy groups of cluster/context/user enries from `~/.kube/config` files into separate files in `~/.kube-multi-config/config.d`.

Note that the `~/.kube/config` needs to remain. It's the only file that contains the `current-cluster:` line.

Restart your shell and `echo $KUBECONFIG` to see the new config files. Run `kubectl config get-contexts` to list all your contexts. You can still use `kubectl config use-context <your context name>` to switch to any context in any config file.

<hr/>

**License:** This walkthrough and the scripts are license under a [BSD license](./LICENSE)
