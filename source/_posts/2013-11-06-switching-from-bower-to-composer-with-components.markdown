---
layout: post
title: "Switching from Bower to composer with components"
date: 2013-11-06 09:34
comments: true
categories: 
---

Last time I talked about [Deploying a symfony2 app with capifony and bower](/blog/2013/06/14/deploying-a-symfony2-app-with-capifony-and-bower/). 

Working with this setup for a few months I found a few problems:

 * Deploys get a lot slower
 * Strange errors with cache clear and cache warmup
 * Why not use composer as we already use it for other dependencies
 
Looking for a better solution I found out most of my dependencies were already on packagist, so I could just include them in my composer.json

``` json /composer.json
{
	"require": {
		"components/jquery": "1.10.*"
	}
}
```

Now while running composer.phar install you'll get the following output:

``` sh php composer.phar install
Loading composer repositories with package information
Installing dependencies (including require-dev)
  - Installing symfony/process (v2.3.6)
    Loading from cache

  - Installing kriswallsmith/assetic (v1.1.2)
    Loading from cache

  - Installing robloach/component-installer (0.0.12)
    Loading from cache

  - Installing components/jquery (1.10.2)
    Loading from cache

kriswallsmith/assetic suggests installing twig/twig (Assetic provides the integration with the Twig templating engine)
kriswallsmith/assetic suggests installing leafo/lessphp (Assetic provides the integration with the lessphp LESS compiler)
kriswallsmith/assetic suggests installing leafo/scssphp (Assetic provides the integration with the scssphp SCSS compiler)
kriswallsmith/assetic suggests installing ptachoire/cssembed (Assetic provides the integration with phpcssembed to embed data uris)
kriswallsmith/assetic suggests installing leafo/scssphp-compass (Assetic provides the integration with the SCSS compass plugin)
Writing lock file
Generating autoload files
Compiling component files
```

Composer uses the [robloach/component-installer](https://github.com/RobLoach/component-installer) to install the components in /vendor/components along with assetic and symfony process which it uses to generate a require.js file you can directly use in your project. This require.js (and require.css) can be found in /components/require.js

If you're using Symfony2 and want to use assetic instead of the require.js file, you can configure that in your config.yml like so:

``` yml /app/config.yml
# Assetic Configuration
assetic:
    ...
    assets:
        jquery_js:
                    inputs:
                        - %kernel.root_dir%/../web/components/jquery/jquery.js
```

Usage is then just using this asset in your template files:

{% raw %}
``` html /app/Resources/views/base.html.twig
{% javascripts
            "@jquery_js"
            output='compiled/main.js'
%}
<script src="{{ asset_url }}"></script>
{% endjavascripts %}
```        
{% endraw %}

Ok, so what packages are available on packagist?
------------------------------------------------

Just fill in components on [packagist](https://packagist.org/search/?q=components) which already has 44 pages. Ok, so not all of those are real components, there are some other php packages as well, but most popular libs are already available this way.

I need component X and it's not on packagist, what to do?
---------------------------------------------------------

Just add your own repository to composer.json and figure out which specific files you need in your project. 
The following is an example with jqplot

``` json /composer.json
{ 
    "repositories": [
        {
            "type": "package",
            "package": {
                "name": "components/jqplot",
                "type": "component",
                "version": "1.0.8",
                "dist": {
                    "url": "https://bitbucket.org/cleonello/jqplot/downloads/jquery.jqplot.1.0.8r1250.zip",
                    "type": "zip"
                },
                "extra": {
                    "component": {
                        "scripts": [
                            "excanvas.js",
                            "jquery.jqplot.js",
                            "plugins/*.js"
                        ],
                        "styles": [
                            "jquery.jqplot.css"
                        ]
                    }
                },
                "require": {
                    "robloach/component-installer": "*"
                }
            }
        }
    ],
    "require": {
    	"components/jqplot": "1.0.8",
	}
}
```

 * L6: Add a name for the package, which you use later on in the require
 * L9 Add a repository with a dist to a zipfile (jqplot is on bitbucket but works as well for github or normal http urls)
 * l7 specify type: 'component'
 * L25 require the robloach/component-installer so it is handled by the component installer by composer
 * L32 require your package with the name you've given in L6 and the version in L8

 * Lines 13 till 24 are important as they define which files will be copied to the /components/jqplot directory from the /vendords/components/jqplot directory. In this case we'll need excanvas, the jquery.jqplot.js file and all plugins. And we include the jquery.jqplot.css style as well. 

 Check out the documentation on [robloach/component-installer](https://github.com/RobLoach/component-installer#packages-without-composer-support) for more information. 

Conclusion
----------

As most js and css packages are on packagist anyway and composer is really good integrated into capifony, this solution works a lot better than depending on bower. Adding a package not on packagist is a bit more work, but not that difficult. Try it out yourself!