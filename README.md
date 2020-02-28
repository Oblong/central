
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

* Linux operating system
* [Docker CE (Community Edition)](https://docs.docker.com/install/) > v19.03.3
* [Docker Compose](https://docs.docker.com/compose/install/) > v1.24.1
* wget
* Access to the Internet to download a Docker Compose file and pull container
  images



# Getting Started

Oblong provides a convenient command line tool to make managing your Mezzanine
Central instance easy and robust. [Grab it here](https://raw.githubusercontent.com/Oblong/central/master/central).

1. Initialize your configuration.

        $ ./central init --hostname <hostname> --port <port>

2. Start Mezzanine Central.

        $ ./central up

3. Go to https://[hostname]:[port] where you'll be prompted to create an
   account and upload a license file.

Run `central --help` or `central <command> --help` to learn more about the
available commands.



# Updating

To update Mezzanine Central to the latest version:

    $ ./central upgrade

Running this command automatically creates a backup which will be restored
should you experience trouble following an upgrade:

    $ ./central revert

You can also create and restore named backups at any time with the `backup`
and `restore` commands. Backups encapsulate both the central software
version as well as the database containing devices and settings, and the
environment defined with `init`.



# Troubleshooting

Check the current status of Mezzanine Central:

    $ ./central status

Check the logs for the various containers:

    $ ./central logs

If you encounter trouble, you can bring down the containers and destroy
their volumes to ensure a clean start the next time you bring them up.
This is recommended when encountering difficulty, or after changing the
hostname and port using `init`.

    $ ./central destroy

If you forgot the admin account credentials, you can initiate a reset
using the following command. After running the command, you can create a
new default account login using the Central web portal.

    $ ./central reset-admin

Check the SSL connection:

    $ openssl s_client -showcerts -connect <CENTRAL_HOST>:<CENTRAL_PORT>

This command uses a generic SSL/TLS client that establishes a connection to a
server speaking SSL/TLS and returns all the certificates. The certificate chain
is listed under the "Certificate chain" label. Use Ctrl+D to close the session
to which the server should respond "DONE". This assumes OpenSSL is already
installed.

The output from the previous command will also display the raw certificate data.
To view it in a human readable format:

    $ openssl s_client -showcerts -connect <CENTRAL_HOST>:<CENTRAL_PORT> | openssl x509 -text -noout
