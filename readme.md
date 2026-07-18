# ssh-lfs

## 1. What It Does

`ssh-lfs` is a small shell-based file synchronization tool. It transfers selected
files and directories between a local working directory and a remote server over
SSH, using `rsync`. Run `ssh-lfs init <relative-path>` to create the `.ssh-lfs/`
configuration template for a project.

Sync lists are stored in `.ssh-lfs/lfs-config/`. Each `.config` file is a named
sync configuration; each line is one relative path. The same relative path is
used on both sides:

```text
local:  <current-working-directory>/<relative-path>
remote: <REMOTE_BASE>/<relative-path>
```

For example, if the list contains `example_data/`, a push run from the directory
containing `example_data/` copies it to `REMOTE_BASE/example_data/`. Use `push`
to send local files to the remote host and `pull` to retrieve them.

## 2. Quick Start

### 2.1 Install ssh-lfs

Clone or copy this repository, then install the `ssh-lfs` command into your
user-level bin directory. From the project root, run:

```sh
mkdir -p ~/.local/bin
cp ./ssh-lfs ~/.local/bin/ssh-lfs
chmod +x ~/.local/bin/ssh-lfs
```

Check whether the command is available:

```sh
which ssh-lfs
```

If there is no output, `~/.local/bin` is not in your `PATH`. Add the following
line to `~/.bashrc`:

```sh
export PATH="$HOME/.local/bin:$PATH"
```

Then reload your shell configuration and check again:

```sh
source ~/.bashrc
which ssh-lfs
```

The local machine must have `bash`, `ssh`, and `rsync` installed. Configure SSH
key authentication or another supported SSH authentication method before syncing.

### 2.2 Use ssh-lfs

**Important: add `.ssh-lfs/` to the target project's `.gitignore` before adding
your own host configuration. This directory can contain private SSH host names,
usernames, and remote paths, so do not upload it to a repository.**

1. Initialize the project configuration from the directory containing the project:

   ```sh
   ssh-lfs init .
   ```

   For a new project directory, provide its relative path instead:

   ```sh
   ssh-lfs init my-project
   cd my-project
   ```

   This creates `.ssh-lfs/hosts/ffu-ssh-example.host`,
   `.ssh-lfs/hosts/ssh-alias-example.host`, and a fully commented
   `.ssh-lfs/lfs-config/ssh-lfs.config` in the target directory. If `.ssh-lfs/`
   already exists, `init` does not overwrite any files and reports that the
   project has already been initialized.

2. Edit one of the generated `.host` files in `.ssh-lfs/hosts/`. Use
   `ffu-ssh-example.host` when the connection details belong in the host file:

   ```ini
   SSH_HOST=server.example.com
   SSH_USER=deploy
   SSH_PORT=22
   REMOTE_BASE=/srv/ssh-lfs
   ```

   Or edit `ssh-alias-example.host` to use an alias from `~/.ssh/config`. In
   that case, only these two values are required:

   ```ini
   SSH_HOST=my-ssh-alias
   REMOTE_BASE=/srv/ssh-lfs
   ```

3. Edit `.ssh-lfs/lfs-config/ssh-lfs.config` to list the relative files and
   directories to synchronize. Blank lines and lines beginning with `#` are
   ignored:

   ```text
   example_data/
   .env
   ```

4. Run the command from the initialized local project directory, which contains
   both the listed paths and `.ssh-lfs/`. The optional final argument selects a
   file in `.ssh-lfs/lfs-config/`; omit the `.config` extension if preferred:

   ```sh
   ssh-lfs push ffu-ssh-example
   ssh-lfs pull ffu-ssh-example
   ssh-lfs push ffu-ssh-example ssh-lfs-l
   ```

   The first two commands select the default `ssh-lfs.config`; the last one
   selects `ssh-lfs-l.config`. This lets separate file groups be synchronized
   independently.

   To synchronize a different local directory, first run `ssh-lfs init .` there.
   Once initialized, simply run the installed command from that directory:

   ```sh
   ssh-lfs push ffu-ssh-example
   ```

This repository includes `example_data/` as sample content. To synchronize a
directory with that name in your own project, uncomment or add `example_data/`
in `.ssh-lfs/lfs-config/ssh-lfs.config`, configure one of the generated host files, and run
the corresponding `ssh-lfs push` command.

### 2.3 Lightweight Project Configuration

This project also includes `.ssh-lfs/lfs-config/ssh-lfs-l.config` as a
lightweight transfer list. It deliberately contains only:

```text
example_data/example.txt
```

Run `ssh-lfs push <host-name> ssh-lfs-l` to use this configuration, rather than
the complete `example_data/` directory.
