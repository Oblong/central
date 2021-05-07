# What is Mezzanine Central?

Mezzanine Central is a directory service for Mezzanines inside an organization
that also provides centralized configuration and monitoring. To learn more
about Mezzanine, visit [the Mezzanine website](https://oblong.com/mezzanine).

You'll need a license provided by Oblong to deploy Mezzanine Central on your
network and enable registration of Mezzanine systems, however you can set up
the tool itself in advance.

# Dependencies

Mezzanine Central is deployed using [Docker](https://docs.docker.com/). Docker
is a tool that packages an application and all its dependencies in an isolated
environment called a container. This container is downloaded as an image on the
host system and is used to run container instances. Each instance is isolated,
named, and utilizes the underlying kernel's functionality.

Mezzanine Central must be deployed on a Linux system with both Docker and
[Docker Compose](https://docs.docker.com/compose/) installed.

Minimal requirements:

- x86 CPU (we do not support ARM processors such as those used by Raspberry
  Pi)
- Single core
- 512MB of RAM
- Ubuntu 18.04 or higher is required as the Linux operating system
- [Docker CE (Community Edition)](https://docs.docker.com/install/) > v19.03.3
- [Docker Compose](https://docs.docker.com/compose/install/) > v1.24.1
- wget
- Access to the Internet to download a Docker Compose file and pull container
  images

# Getting Started

Oblong provides a convenient command line tool to make managing your Mezzanine
Central instance easy and robust. [Grab it here](https://raw.githubusercontent.com/Oblong/central/master/central).

1.  Initialize your configuration.

        $ ./central init --hostname <hostname> --port <port>

2.  Start Mezzanine Central.

        $ ./central up

3.  Go to https://[hostname]:[port] where you'll be prompted to create an
    account and upload a license file.

Run `central --help` or `central <command> --help` to learn more about the
available commands.

# Updating

To update Mezzanine Central to the latest version:

    $ ./central upgrade

Running this command automatically creates a backup which will be restored
should you experience trouble following an upgrade:

    $ ./central revert

# Backup / Restore

You can also create and restore named backups at any time with the `backup`
and `restore` commands. Backups encapsulate the Mezzanine Central software
version, the database containing devices and settings, and the environment
defined with `init`.

To backup, first make sure your Mezzanine Central server cluster is up,
then run the backup command:

    $ ./central backup <BACKUP_FILE>

To restore, the Mezzanine Central cluster must be down. With the cluster
down, run the following command:

    $ ./central restore <BACKUP_FILE>

# Troubleshooting

## Commands

The Mezzanine Central CLI has several other commands that are useful
for troubleshooting issues.

### Checking the Status

To check the status of your Mezzanine Central server:

    $ ./central status

### Checking the Logs

To check the Mezzanine Central server logs:

    $ ./central logs

### Restarting the Cluster:

To restart the cluster you need to bring down the containers and bring them
back up:

    $ ./central down
    $ ./central up

### Resetting Admin Credentials

If you forgot the admin account credentials, you can initiate a reset
like so:

    $ ./central reset-admin

After running the command, you can create a new default account login using
the Mezzanine Central web portal.

### Destroying the Cluster

If you need to regenerate the server certificates and remove all registered
Mezzanines you can bring the cluster down and blow everything away to ensure
a clean start the next time you run `central up`. This is recommended as a
last resort to problems or after changing the server's hostname or port using
`init`.

    $ ./central destroy

## Mezzanine Central Agent Logs

The central-agent service is what runs on Mezzanine to handle communication
with the Mezzanine Central server. These logs should be accessible via a
Mezzanine's admin web portal under the Logs tab as `central-agent.service`.

## Mezzanine Central Service Failures

The Mezzanine Central server and the central-agent service running on
Mezzanine should automatically restart if they encounter an unrecoverable
error. If the server goes down, all connected Mezzanines will attempt to
reconnect until it comes back online.

If for whatever reason it is not coming back up you can try the following
options:

- [restart the cluster](#restarting-the-cluster)
- [restore a backup](#backup-restore)
- [destroy and rebuild the cluster](#destroying-the-cluster)

## Connection Issues

The most common problems when setting up Mezzanine Central stem from
networking issues.

### FQDN, Hostname, and IP Address

When identifying machines in Mezzanine Central, be it the server during
`central init` or when adding rooms through the dashboard, prefer using
the fully qualified domain name (FQDN). IP addresses should work but
need to be static to avoid issues in the future.

### Certificates

Mezzanine Central uses a secure connection, so if certificates are wrong
on either the server or the associated Mezzanine appliances you'll run into
issues registering rooms and receiving updates.

#### Mezzanine Central => Mezzanine

To check the connection from the Mezzanine Central server to a Mezzanine
appliance and verify the certificate:

    $ openssl s_client -showcerts -connect <MEZZANINE_ADDRESS>:443 | openssl x509 -text -noout

The output should contain details about the Mezzanine under `Issuer`,
`Subject`, and `Subject Alternative Name`. The current date should also
fall within the `Validity` range.

#### Mezzanine => Mezzanine Central

To check the connection from a Mezzanine appliance to the Mezzanine Central
server and verify the certificate you'll need to SSH into the appliance and
run:

    $ openssl s_client -showcerts -connect <CENTRAL_HOST>:<CENTRAL_PORT> | openssl x509 -text -noout

The `Issuer` and `Subject` should have "Mezzanine Central Temporary
Certificate" as the subject org (O) and `<CENTRAL_HOST>` as the subject
common name (CN). The `Subject Altnernative Name` should contain the
`<CENTRAL_HOST>`. The current date should also fall within the `Validity`
range.

### Date and Time

If the date and time are out of sync on the Mezzanine Central server's host
machine or on the individual Mezzanines this could cause communication
issues.

#### Setting Mezzanine Date & Time

You should be able to set the Mezzanine date & time via the admin portal.
This is the ideal approach for our appliances.

The following methods should work via the terminal on Mezzanine appliances and
any Mezzanine Central server host machine running Ubuntu 18.04 or other Linux
distro using systemd (run `systemctl --version` to check).

#### Setting Automatic Date & Time

Check the time with the following command:

    $ timedatectl

Ideally the time is correct and the device is using NTP as represented by
the following fields of the output:

           System clock synchronized: yes
    systemd-timesyncd.service active: yes

If NTP is not enabled you can enable it by running the following:

    $ sudo timedatectl set-ntp true

This should automatically sync the system time to the NTP server.

If NTP is already set but the time is still off by several minutes you can
try forcing a sync, but this could be indicative of network problems:

    $ sudo systemctl restart systemd-timesyncd.service

You can confirm that the time was updated by checking the time as before
using `timedatectl` or by checking the service status:

    $ systemctl status systemd-timesyncd.service

#### Setting Manual Date & Time

If for some reason you can't use NTP such as on a private network with no
access to an NTP server, you can set the time manually like so:

    $ sudo timedatectl set-ntp false
    $ sudo timedatectl set-time '2020-07-20 18:23:49'

**Note**: After changing the system time you may need to reboot the device or
cluster and regenerate the certificates.
