##PHP-FPM: Configuration the Listen Directive


PHP-FPM can listen on multiple sockets. I also listen on Unix sockets, or TCP sockets. See how this works and how to ensure Nginx is properly sending requests to PHP-FPM.
Command Rundown

###Default Configuration

Edit PHP-FPM configuration

    # Configure PHP-FPM default resource pool
    sudo vim /etc/php5/fpm/pool.d/www.conf

PHP-FPM Listen configuration:

    # Stuff omitted
    listen = /var/run/php5-fpm.sock
    listen.owner = www-data
    listen.group = www-data

Also edit Nginx and see where it's sending request to PHP-FPM:

    # Files: /etc/nginx/sites-available/default

    # ... stuff omitted

    server ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php5-fpm.sock;
    }

We can see above that Nginx is sending requests to PHP-FPM via a unix socket (faux file) at /var/run/php5-fpm.sock. This is also where the www.conf file is setting PHP-FPM to listen for connections.
###Unix Sockets

These are secure in that they are file-based and can't be read by remote servers. We can further use linux permission to set who can read and write to this socket file.

Nginx is run as user/group www-data. PHP-FPM's unix socket therefore needs to be readable/writable by this user.

If we change the Unix socket owner to user/group ubuntu, Nginx will then return a bad gateway error, as it can no longer communicate to the socket file. We would have to change Nginx to run as user "ubuntu" as well, or set the socket file to allow "other" (non user nor group) to be read/written to, which is insecure.

    # Stuff omitted
    listen = /var/run/php5-fpm.sock
    listen.owner = ubuntu
    listen.group = ubuntu

So, file permissions are the security mechanism for PHP-FPM when using a unix socket. The faux-file's user/group and it's user/group/other permissions determines what local users and processes and read and write to the PHP-FPM socket.
###TCP Sockets

Setting the Listen directive to a TCP socket (ip address and port) makes PHP-FPM listen over the network rather than as a unix socket. This makes PHP-FPM able to be listened to by remote servers (or still locally over the localhost network).

Change Listen to Listen 127.0.0.1:9000 to make PHP-FPM listen on the localhost network. For security, we can use the listen.allowed_clients rather than set the owner/group of the socket.

PHP-FPM:

    # Listen on localhost port 9000
    Listen 127.0.0.1:9000
    # Ensure only localhost can connect to PHP-FPM
    listen.allowed_clients = 127.0.0.1

Nginx:

# Files: /etc/nginx/sites-available/default

    # ... stuff omitted

    server ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass 127.0.0.1:9000;
    }

