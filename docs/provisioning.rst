============
Provisioning
============

To run the provisioning scripts, get a local copy of this repository on your
machine::

    $ git clone --recursive https://github.com/abersailbot/dewi

Now set up ssh configuration to connect to the target device (the pi on the
boat). If you don't already have one, create ``~/.ssh/config`` and add
something like the following, changing the hostname accordingly.

..code-block::

    Host dewi
      User pi
      HostName 192.168.0.10

Generate an ssh key if you do not already have one::

    $ ssh-keygen -f ~/.ssh/id_rsa -t rsa

Copy the ssh key to the pi::

    $ ssh-copy-id dewi

Then, run::

    $ ./deploy dewi

This should copy all the local data to the pi and install the configuration to
the appropriate locations.
