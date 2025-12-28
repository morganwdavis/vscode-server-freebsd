# VS Code Server and Remote SSH on FreeBSD

### Setting Up the FreeBSD Server

```shell
service linux enable
service linux start
cp .bash_linux ~
pkg install linux_base-rl9
```

Allow sshd to accept BASH_ENV references during the connection:

```shell
sysrc sshd_flags="-o AcceptEnv=BASH_ENV"
```

... and then restart sshd with `service sshd restart`.

It is also a good idea to reboot and check that everything starts up properly to avoid surprises later.

## Setting Up the VS Code Client

Edit your `~/ssh/config` file on your computer where the VS Code client is installed and add a special host entry just for VS Code connections:

```text
Host vscode-server
	Hostname	my.vscode.server.com
	SetEnv BASH_ENV=".bash_linux"
```

Next, in VS Code, after installing the [Remote - SSH](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-ssh) extension, add a new remote SSH host via the command pallete by selecting `Remote-SSH: Add New SSH Host...` and following the prompts.  Then connect to the new host via the `Open a remote window` icon in the lower left or the command pallete by selecting `Remote-SSH: Connect to Host...`. The first time you make the connection, VS Code will ask you to specify your remote platform. Choose "Linux".

## Tips for Windows Users

A convenient way to open VS Code onto your remote SSH connection and start in a project workspace is to create a shortcut icon on the desktop and set the target property to something like this:

`"C:\Program Files\Microsoft VS Code\Code.exe" --folder-uri vscode-remote://ssh-remote+vscode-server/home/username/path/to/project`

... where `vscode-server` matches the Host identifier in the ssh config entry and everything else after that matches the path to your project.
