# VS Code Server and Remote SSH on FreeBSD

## RECOMMENDED OPTION - Rocky Linux emulator (via pkg)

> Original how-to: [How to run Visual Studio (VS) Code Remote over SSH on FreeBSD 13 and 14](https://group.miletic.net/en/blog/2024-06-14-how-to-run-visual-studio-vs-code-remote-over-ssh-on-freebsd-13-and-14/).

> Important: If you previously used the Debian solution described below, you can remove it with `service linux disable && service debian disable`. Reboot, and after logging back in, make sure you have nothing mounted in /compat, and then manually remove /compat/linux and /compat/debian.

### Setting Up the FreeBSD Server

```shell
# pkg install linux-rl9-libsigsegv
# service linux enable
# service linux start
# cp .bash_linux ~
```

Allow sshd to accept BASH_ENV references during the connection:

```shell
# sysrc sshd_flags="-o AcceptEnv=BASH_ENV"
```

... and then restart sshd with `service sshd restart`.

It is also a good idea to reboot and check that everything starts up properly to avoid surprises later.

### Setting Up the VS Code Client

Edit your `~/ssh/config` file on your computer where the VS Code client is installed and add a special host entry just for VS Code connections:

```
Host vscode-server
	Hostname	my.vscode.server.com
	SetEnv BASH_ENV=".bash_linux"
```

Next, in VS Code, after installing the [Remote - SSH](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-ssh) extension, add a new remote SSH host via the command pallete by selecting `Remote-SSH: Add New SSH Host...` and following the prompts.  Then connect to the new host via the `Open a remote window` icon in the lower left or the command pallete by selecting `Remote-SSH: Connect to Host...`. The first time you make the connection, VS Code will ask you to specify your remote platform. Choose "Linux".

## Tips for Windows Users

A convenient way to open VS Code onto your remote SSH connection and start in a project workspace is to create a shortcut icon on the desktop and set the target property to something like this:

`"C:\Program Files\Microsoft VS Code\Code.exe" --folder-uri vscode-remote://ssh-remote+vscode-server/home/username/path/to/project`

... where `vscode-server` matches the Host identifier in the ssh config entry and everything else after that matches the path to your project.

## LEGACY OPTION - Using Debian (deprecated)

This legacy option is compatible with VS Code 1.88+ and FreeBSD 14. This is possible using `debootstrap` to install a stable release of [Debian](https://www.debian.org/News/), such as 12.5, codename "Bookworm".

> Inspiration for this repo originally came from the many good contributions in this discussion: [Configure FreeBSD to work with VScode's remote ssh extension](https://gist.github.com/mateuszkwiatkowski/ce486d692b4cb18afc2c8c68dcfe8602). Many thanks to all of those who participated in figuring this out.

### Setting Up the FreeBSD Server

>  Important: If you previously installed the `linux_base-c7` package for VS Code, it is no longer compatible with recent versions of VS Code Server and remote SSH connections will not work.  You can remove it with `service linux stop && service linux disable && pkg delete linux_base-c7`. Reboot and then manually remove /compat/linux.

```shell
# pkg install debootstrap
# service linux enable && service linux start
# debootstrap bookworm /compat/debian
# cp .bash_debian ~
# install -m 0755 -o root -g wheel debian /usr/local/etc/rc.d
# service debian enable && service debian start
```

Allow sshd to accept BASH_ENV references during the connection:

```shell
# sysrc sshd_flags="-o AcceptEnv=BASH_ENV"
```

... and then restart sshd with `service sshd restart`.

It is also a good idea to reboot and check that everything starts up properly to avoid surprises later.

### Setting Up the VS Code Client

Edit your `~/ssh/config` file on your computer where the VS Code client is installed and add a special host entry just for VS Code connections:

```
Host vscode-server
	Hostname	my.vscode.server.com
	SetEnv BASH_ENV=".bash_debian"
```

Next, in VS Code, after installing the [Remote - SSH](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-ssh) extension, add a new remote SSH host via the command pallete by selecting `Remote-SSH: Add New SSH Host...` and following the prompts.  Then connect to the new host via the `Open a remote window` icon in the lower left or the command pallete by selecting `Remote-SSH: Connect to Host...`.  The first time you do this you need to specify the host platform, and in this case will be Linux.

Note for Windows users: If you use the shortcut feature described above, you must add `/compat/debian` as a path component for it to work.  Example:

`"C:\Program Files\Microsoft VS Code\Code.exe" --folder-uri vscode-remote://ssh-remote+vscode-server/compat/debian/home/username/path/to/project`
