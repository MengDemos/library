##Summary

Several days ago, I had to deal with a compromised web application: an attacker had somehow managed to upload PHP backdoor scripts onto the application’s server. Thanks to some log file sleuthing and Google searches, I was quickly able to identify what had allowed the attack: a misconfigured nginx server can allow non-PHP files to be executed as PHP. As I researched the vulnerability a bit more, however, I realized that many of the nginx / PHP setup tutorials found on the Internet suggest that people use vulnerable configurations.
###The misconfiguration

As I mentioned, the attack was made possible by a very simple misconfiguration between nginx and php-fastcgi. Consider the configuration block below, taken from a tutorial at http://library.linode.com/web-servers/nginx/php-fastcgi/fedora-14:

```
server {
    listen 80;
    server_name www.bambookites.com bambookites.com;
    access_log /srv/www/www.bambookites.com/logs/access.log;
    error_log /srv/www/www.bambookites.com/logs/error.log;
    root /srv/www/www.bambookites.com/public_html;

    location / {
        index  index.html index.htm index.php;
    }

    location ~ \.php$ {
        include /etc/nginx/fastcgi_params;
        fastcgi_pass  127.0.0.1:9000;
        fastcgi_index index.php;
        fastcgi_param  SCRIPT_FILENAME /srv/www/www.bambookites.com/public_html$fastcgi_script_name;
    }
}
```

It may not be immediately clear, but this configuration block allows for arbitrary code execution under certain circumstances (and I don’t just mean if an attacker can upload a file ending in .php: that kind of vulnerability is independent of the web server used).

Consider a situation where remote users can upload their own pictures to the site. Lets say that an attacker uploads an image to http://www.bambookites.com/uploads/random.gif. What happens, given the server block above, if the attacker then browses to http://www.bambookites.com/uploads/random.gif/somefilename.php?

> nginx will look at the URL, see that it ends in .php, and pass the path along to the PHP fastcgi handler.
> PHP will look at the path, find the .gif file in the filesystem, and store /somefilename.php in $_SERVER['PATH_INFO'], executing the contents of the GIF as PHP.

Since GIFs and other image types can contain arbitrary content within them, it’s possible to craft a malicious image that contains valid PHP. That is how the attacker was able to compromise the server: he or she uploaded a malicious image containing PHP code to the site, then browsed to the file in a way that caused it to be parsed as PHP.

This issue was first discovered almost a year ago. The original report can be found in Chinese at http://www.80sec.com/nginx-securit.html. There is also a discussion about it on the nginx forums.

This issue can be mitigated in a number of ways, but there are downsides associated with each of the possibilities:

Set cgi.fix_pathinfo to false in php.ini (it’s set to true by default). This change appears to break any software that relies on PATH_INFO being set properly (eg: Wordpress).
Add try_files $uri =404; to the location block in your nginx config. This only works when nginx and the php-fcgi workers are on the same physical server.
Add a new location block that tries to detect malicious URLs. Unfortunately, detecting based on the URL alone is impossible: files don’t necessarily need to have extensions (eg: README, INSTALL, etc).
Explicitly exclude upload directories using an if statement in your location block. The disadvantage here is the use of a blacklist: you have to keep updating your nginx configuration every time you install a new application that allows uploads.
Don’t store uploads on the same server as your PHP. The content is static anyway: serve it up from a separate (sub)domain. Of course, this is easier said than done: not all web applications make this easy to do.

> Note: If anyone is aware of other possible solutions (or workarounds to improve the effectiveness of these solutions), please let me know and I’ll add them here!]
Tutorials

Now, the configuration file for the compromised server wasn’t written by hand. When the server was set up, the configuration was created based on suggestions found on the Internet. I assume that other people use tutorials and walkthroughs for setting up their servers as well. Unfortunately, most of the documentation for configuring nginx and php-fastcgi still encourages people to set up their servers in a vulnerable way.

The default configuration file for nginx suggests the use of an insecure location block (source).
The nginx wiki supplies potentially dangerous examples as well. To be fair, some pages do encourage users to explicitly prevent PHP execution in upload directories [Edit: and in the Pitfalls document, which everyone should read before configuring nginx]. However, other pages ignore the issue entirely.
The Linode Library has an extensive collection of documents, including a number that talk about setting up nginx and PHP on various OSes. Unfortunately, all of those tutorials suggest using a vulnerable configuration for PHP. I’ve contacted the documentation team at Linode and I’m waiting to hear back from them. [Update: The Linode documentation team has updated the tutorials with more information and workarounds]
Howto Forge has several tutorials (1, 2) which show up when searching Google for “nginx php setup.” These tutorials also suggest the use of a vulnerable configuration.
People have written many tutorials on blogs and other sites (ie: 1, 2). A number of these tutorials encourage using the same vulnerable configuration.

In contrast, codex.wordpress.org provides an excellent configuration example that warns people about and mitigates the vulnerability. I’ve reproduced the relevant portion below:

```
# Pass all .php files onto a php-fpm/php-fcgi server.
location ~ \.php$ {
   # Zero-day exploit defense.
   # http://forum.nginx.org/read.php?2,88845,page=3
   # Won't work properly (404 error) if the file is not stored on this server, which is entirely possible with php-fpm/php-fcgi.
   # Comment the 'try_files' line out if you set up php-fpm/php-fcgi on another machine.  And then cross your fingers that you won't get hacked.
   try_files $uri =404;

   fastcgi_split_path_info ^(.+\.php)(/.+)$;
   include fastcgi_params;
   fastcgi_index index.php;
   fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
#    fastcgi_intercept_errors on;
   fastcgi_pass php;
}  
```
Conclusion

If you run PHP on an nginx web server, check your configuration and update if necessary.
If you’re doing a security audit on a PHP application running on an nginx web server, remember to test for this configuration.
If you run across a tutorial that is out of date, please point the author to this post.
If you know of a way to better secure nginx / php-fastcgi, let me know!
