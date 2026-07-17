# ssh-lfs

## 1. What It Does

`ssh-lfs` is a small shell-based file synchronization tool. It transfers selected
files and directories between a local working directory and a remote server over
SSH, using `rsync`.

The sync list is stored in `.ssh-lfs/ssh-lfs.config`. Each line is one relative
path. The same relative path is used on both sides:

```text
local:  <current-working-directory>/<relative-path>
remote: <REMOTE_BASE>/<relative-path>
```

For example, if the list contains `example_data/`, a push from the project root
copies `./example_data/` to `REMOTE_BASE/example_data/`. Use `push` to send local
files to the remote host and `pull` to retrieve them.

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

1. Create a host configuration in `.ssh-lfs/hosts/`. Host files use the `.host`
   suffix. You can copy the complete connection example:

   ```sh
   cp .ssh-lfs/hosts/full-ssh.host .ssh-lfs/hosts/my-server.host
   ```

   Then edit `my-server.host`:

   ```ini
   SSH_HOST=server.example.com
   SSH_USER=deploy
   SSH_PORT=22
   REMOTE_BASE=/srv/ssh-lfs
   ```

   Alternatively, use an alias from `~/.ssh/config`. In that case, only these
   two values are required:

   ```ini
   SSH_HOST=my-ssh-alias
   REMOTE_BASE=/srv/ssh-lfs
   ```

2. Edit `.ssh-lfs/ssh-lfs.config` to list the relative files and directories to
   synchronize. Blank lines and lines beginning with `#` are ignored:

   ```text
   example_data/
   .env
   ```

3. Run the command from the local directory that contains the listed paths.
   From this repository root, use:

   ```sh
   ./ssh-lfs push my-server
   ./ssh-lfs pull my-server
   ```

   To synchronize a different local directory, call the script by its absolute
   path from that directory:

   ```sh
   /path/to/ssh-lfs/ssh-lfs push my-server
   ```

The repository includes `example_data/`, which is already enabled in the default
manifest. You can test the setup from the repository root with:

```sh
./ssh-lfs push full-ssh
```
