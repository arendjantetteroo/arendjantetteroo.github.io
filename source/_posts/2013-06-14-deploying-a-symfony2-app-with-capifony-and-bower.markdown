---
layout: post
title: "Deploying a Symfony2 app with Capifony and Bower"
date: 2013-06-14 18:21
comments: true
categories: [Symfony2, Bower, SPBowerBundle, Capifony]
---

UPDATE: See [Switching from bower to composer with components](/blog/2013/11/06/switching-from-bower-to-composer-with-components/) for a better solution.

<hr>


Today I wanted to deploy a new Symfony2 app I'm working on. 

In this app I'm using the [SPBowerBundle](https://github.com/Spea/SpBowerBundle) to use the [bower](https://github.com/bower/bower) package manager for all the 3rd party frontend code, like [Twitter Bootstrap](http://twitter.github.io/bootstrap/), [Jquery](http://jquery.org) and [Jqplot](http://www.jqplot.com). 

[Capifony](http://capifony.org) normally works great for deploying symfony2 apps (if you're not using it, you should check it out!), but this time it didn't.

The [setup of the SPBowerBundle](https://github.com/Spea/SpBowerBundle/blob/master/Resources/doc/index.md) is very straightforward, but if you want to deploy it with capifony you need to setup a few things that aren't really obvious. 

1) As Capifony by default does [not run the scripts](http://capifony.org/cookbook/composer-vendors.html) included by Composer, [step 6 of the SPBowerBundle](https://github.com/Spea/SpBowerBundle/blob/master/Resources/doc/index.md#step-6-add-composer-scripts-for-automatic-update-of-dependencies) of the installation manual doesn't work. 
Instead you need to add the following to your deploy.rb:
``` ruby /app/config/deploy.rb 
before 'symfony:cache:warmup', 'symfony:sp:bower:install'

namespace :symfony do
  namespace :sp do
  	namespace :bower do
    	desc '[internal] Run the bower install'
    	task :install do
    		invoke_command "cd #{latest_release} && app/console sp:bower:install --env=prod", :via => run_method
    	end
    end
  end
end
```
Before the cache is warmed up, the SPBowerBundle installs the specified assets, so they are available for the specified assetic filters. 

2) Make sure you setup the cache warmup of the SPBowerBundle, [step 7](https://github.com/Spea/SpBowerBundle/blob/master/Resources/doc/index.md#step-7-installing-dependencies-on-every-cache-warmup) 
``` yml app/config/config.yml 
# app/config/config.yml
sp_bower:
    install_on_warmup: true
```    
as otherwise you get the following error: 

``` php
 [Sp\BowerBundle\Bower\Exception\MappingException]
Bower returned an invalid dependency mapping. This mostly happens when the dependencies are not yet installed or if you are using an old version of bower.
```

If you did that, and you still get the above mentioned exception, copy your components.json to your server and try a bower install on it. If that works, it's a configuration issue in capifony, if not, then you should check your bower install. 
