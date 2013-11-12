---
layout: post
title: "Running Symfony2 on the php 5.4 internal webserver and assets 404"
date: 2013-11-12 08:58
comments: true
categories: 
---

TL;DR
-----
Use the app/console server:run method to start a correct symfony2 server site instead of just using php -S

Long version
------------

I'm working on a docker container for a client, so the working environment is almost the same as the production environment and doesn't interfere with other projects that use other dependencies. More on [Docker](docker.io) in a later post. 

Working on this Docker container I tried to get [Symfony2](http://www.symfony.com) running with the easiest option, so not configuring apache-modphp or nginx-phpfpm, but just using the [internal webserver](http://www.php.net/manual/en/features.commandline.webserver.php) that php offers since 5.4. As this is just a development envirement, this should be fine. 

I started with this command, which seemed to work:

``` php /root/of/your/project
php -S 0.0.0.0:8000 -t web web/app_dev.php
```

However, when visiting the site in question on http://0.0.0.0:8000 all assets error out with a 404. 

Not sure what was happening, I debugged it and tried it on other sites where it worked fine. 

For Symfony 2 there is a console command you can run to get the webserver running, which adds a small router file to let the internal webserver know where certain files are. 

``` php /root/of/your/project
php app/console server:run  -vvv 0.0.0.0:8000
```

The -vvv option defines the verbosity, normally you can skip it, but as I wanted to see what was happening I added it. 
Visiting my site now at http://0.0.0.0:8000 gave a normal functioning site as I expected. 

So what does the server:run command do?
---------------------------------------
The ps fax command shows what is happening when the server:run command is run:

``` 
#ps fax
 9 ?        S+     0:00 php app/console server:run -vvv 0.0.0.0:8005
29 ?        S+     0:00  \_ sh -c '/usr/bin/php5' '-S' '0.0.0.0:8005' '/server/vendor/symfony/symfony/src/Symfony/Bundle/FrameworkBundle/Resources/config/router_dev.php'
30 ?        S+     0:05      \_ /usr/bin/php5 -S 0.0.0.0:8005 /server/vendor/symfony/symfony/src/Symfony/Bundle/FrameworkBundle/Resources/config/router_dev.php
```

The server:run command starts a php webserver with a special router_dev.php file, that looks like this:


``` php /vendor/symfony/symfony/src/Symfony/Bundle/FrameworkBundle/Resources/config/router_dev.php
if (is_file($_SERVER['DOCUMENT_ROOT'].DIRECTORY_SEPARATOR.$_SERVER['SCRIPT_NAME'])) {
    return false;
}

$_SERVER['SCRIPT_FILENAME'] = $_SERVER['DOCUMENT_ROOT'].DIRECTORY_SEPARATOR.'app_dev.php';

require 'app_dev.php';
```

So basically what happens is that a check is done on the filesystem to see if the file exists. If so, a false is returned to the webserver so it knows that it needs to deliver it directly to the browser instead of forwarding the request to the app_dev.php file into our symfony2 application. 

Hope this helps someone avoid a couple of hours searching for why it doesn't work. 

Next post will be a dockerfile for the symfony2 standard edition so you can easily code away on a new project without needing some hours to set it up with docker. 