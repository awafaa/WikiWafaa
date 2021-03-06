#systemd-nspawn#

##Bootstrap the Directory##
First create the target directory and add the base repositories. You can rename the target directory to whatever you feel fits. Obviously change the target repo depending on your desired distro.

`sudo zypper --root /srv/containers/tumbleweed/ addrepo http://download.opensuse.org/distribution/tumbleweed/repo/oss/ repo-oss`
`sudo zypper --root /srv/containers/tumbleweed/ addrepo http://download.opensuse.org/distribution/tumbleweed/repo/non-oss/ repo-non-oss`

Once the repos are added we need to install some base packages. These will pull in the required dependencies.

`zypper --root /srv/containers/tumbleweed/ refresh`
`zypper --root /srv/containers/tumbleweed/ install openSUSE-release bash procps coreutils vim zypper`

##Start the container##
To boot the container into a specific application - in this example the shell:
`sudo systemd-nspawn -jD /srv/containers/tumbleweed -M tumbleweedc0 /bin/bash`

`-j` links the container's journal to the hosts journal
`-D` Specifies the container base directory to use
`-M` Sets the machine name to use

To boot the full container:
`sudo systemd-nspawn -bDn /srv/containers/tumbleweed -M tumbleweedc0`

`-b` Boots the full container
`-D` Specifies the container base directory to use
`-n` Sets up a private network between the host and container
`-M` Sets the machine name to use

##Enabling the container on host boot##
systemd expects nspawn containers to reside in `/var/lib/machines`. You can either move your container to that directory or you can symlink them there:
`mv /srv/containers/tumbleweed /var/lib/machines/tumbleweed`
or
`ln -s /srv/containers/tumbleweed /var/lib/machines/tumbleweed`

Once the location is set, enable the systemd service:
`sudo systemctl enable systemd-nspawn@tumbleweed.service`
`sudo systemctl start systemd-nspawn@tumbleweed.service`

##Managing containers##
`machinectl` is the overseeing command. Using it on its own will list all running containers
`machinectl login $CONTAINER` - Starts a new shell inside the container named _$CONTAINER_
`machinectl status $CONTAIER` - Show detailed information about the container named _$CONTAINER_
`machinectl reboot $CONTAINER` - Reboots the container named _$CONTAINER_
`machinectl poweroff $CONTAINER` - Shutsdown the container named _$CONTAINER_

You can poweroff and reboot containers within a container session by using `systemctl poweroff` or `systemctl reboot` as well as using _machinectl_

Using `ctrl+]]]` from within the container will kill it immediately.

##Tooling##
Most of the core systemd tools work with containers. To utilise them pass the `-M` or `--machine` option.
`journalctl -M $CONTAINER` - view the journal logs for the container named _$CONTAINER_
`systemd-cgls -m $CONTAINER` - show the control group contents of the container named _$CONTAINER_
`systemd-analyze -M $CONTAINER` - view the startup time for the container named _$CONTAINER_
