.. _server-adhoc:

Configure your computer to be a server (advanced)
=================================================

.. note::

    This page assumes you have already installed the oTree server,
    as described on the server setup pages for Windows/Mac
    (see :ref:`here <server>`).

If you will be running your study with devices on the local network
(e.g. your university network) and don't need access from the internet,
you can follow the below steps to use your own computer as a server.

Windows
-------

Create a firewall rule (Windows only)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You need to allow other computers to connect to oTree through your firewall.

-   Open the Windows Firewall
-   Go to "Inbound Rules"
-   Click "New Rule"
-   Select "Port" to make a port rule
-   Under "Specific local ports", enter 80 and 8000
-   Select "Allow the connection"
-   Click "next" then choose a name for your rule (e.g. "oTree").

Find your IP address (Windows)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Open PowerShell or Command Prompt and enter ``ipconfig``.
Look for the entry ``IPv4 Address``.
Maybe it will look something like ``10.0.1.3``, or could also start with 172 or 192.

MacOS
-----

Find your IP address (MacOS)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Go to "System Preferences" > "Network", and select the network you're using.
Depending on your version of macOS, you may have to click "Advanced" > "TCP/IP"
to view your IP address.
Maybe it will look something like ``10.0.1.3``, or could also start with 172 or 192.

Run the server
--------------

Start the server with your IP address and port 8000, e.g.
``otree devserver 10.0.1.3:8000``.

.. note::

    We are just using ``devserver`` as a temporary step.
    Once you have these steps working, you should switch to using the
    production server ``otree runprodserver``.

On the client device's browser, connect to the IP address followed by ``:8000``,
e.g. ``10.0.1.3:8000`` and you should be able to load the oTree demo page.

Make sure your IP address doesn't change
----------------------------------------

In most university networks, your internal IP address will generally stay the same,
as long as you stay connected to the same network. If it changes unpredictably,
you can ask your IT department to add a rule on their DHCP server to always
assign the same IP to your computer.


Try using port 80 (optional)
----------------------------

If it works, you can try using port 80, which is the conventional port for web servers,
and allows you to omit the ``:8000`` from your URLs.

(If you use Skype) fix Skype issue
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Skype sometimes uses port 80, which can conflict with oTree.
go in Skype to Tools > Options > Advanced > Connections
and unselect "Use port 80 and 443 for additional incoming connections".
Or just quit Skype.

Run the server on port 80
~~~~~~~~~~~~~~~~~~~~~~~~~

Replace "8000" with "80".
And actually, instead of typing in your IP address,
you can just use the "wildcard address" ``0.0.0.0``, and oTree will
automatically start your server on whatever your computer's IP address is.

On Windows::

    otree devserver 0.0.0.0:80

On macOS, you need superuser privilege to use port 80::

    sudo otree devserver 0.0.0.0:80

