---
layout: post
title: "Create a Local Composer Repo with Satis"
description: ""
category: 
tags: [Composer, Satis]
---
{% include JB/setup %}
It's no secret that [Composer](http://getcomposer.org) is changing the way PHP developers handle dependencies. I've been using it for about two months now, and I've mentioned my preferred setup in a [previous post](http://www.edsmith.org/content/ultimate-php-dev-environment-lamp-composer-and-jenkins-ci).

When I was explaining my excitement to a fellow developer, he seemed perplexed, saying "Well I guess that's great if you're using a lot of external libraries. But I work in a shop where almost everything we use is written in-house."

That got me to thinking: wouldn't it be great if Composer could handle dependencies from your own SVN or GIT repositories, as if your source control management system were your very own [packagist.org](http://packagist.org)?

Enter [**Satis**](http://getcomposer.org/doc/articles/handling-private-packages-with-satis.md).

[Detailed installation instructions](http://getcomposer.org/doc/articles/handling-private-packages-with-satis.md) can be found in the Composer documentation, but there are a few minor gotchas you should be aware of.

* You need to have a machine (I've done it with Windows and Linux) with a recent version of PHP >= 5.3 installed for this to work. This machine will serve your local Composer repository. It must also be running a web server such as Apache.
* *Any repository you serve through Satis must have a properly-formatted `composer.json` file.*
* Once you have Satis installed, you need to run this command to update your Satis repository:  
```bash
    sudo /path/to/satis/bin/satis build config.json /path/to/your/satis/repo/in/web/root -v
```

The `-v` switch triggers verbose mode, which will indicate if a repository was skipped due to a missing `composer.json` file or other problem. Once you get this stabilized, put this command in a cron job on the server you're running Satis from, and set it to run every 15 minutes or so. Keep in mind that when you run `composer update` on one of your packages, the latest code won't be available until the above command has been run and pulled the latest revision into the Satis repository.

* You must list all of the repositories you wish to serve in the `config.json` file in your Satis directory. An example `config.json` would be:

```json
    {
        "name": "Acme",
        "homepage": "http://linuxbox.acme.com/repo",
        "repositories": [
            { "type": "vcs", "url": "https://svn.acme.com/Webservice_Api" },
            { "type": "vcs", "url": "https://svn.acme.com/Scheduled_Jobs" },
            { "type": "vcs", "url": "https://svn.acme.com/Anvil" },
            { "type": "vcs", "url": "https://svn.acme.com/MouseTrap" },
        ],
        "require-all": true
    }
```

*After you run the `build` command above, you'll know you've done this correctly when you visit your repo's URL in a web browser and see something like this that lists all of your repositories:

![Satis repository screenshot](http://www.edsmith.org/sites/default/files/field/image/satisrepo_0.png "Private Composer repository")

* In order to take advantage of Composer's autoloading, you may need to reorganize your existing code according to the [PSR-0 standard](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-0.md). This isn't required, as you can manually list files in your `composer.json` file using the `files` directive, but it will make your life easier in the long run.  
If you use PSR-0 autoloading, put your source files in the 'src' directory and put this in your `composer.json` file:

```json
    {
        "autoload": {
            "psr-0": {"Vendorname": "src/"}
        }
    }
```

If you use manual autoloading, list the files you want autoloaded like this in you `composer.json` file:

```json
    {
        "autoload": {
    		"files": ["src/vendorname/packagename/class1.php", "src/vendorname/packagename/class2.php"]
    	}
    }
```

or, if you have a combination of both:

```json
    {
        "autoload": {
            "psr-0": {"Vendorname": "src/"},
            "files": ["src/vendorname/packagename/class1.php", "src/vendorname/packagename/class2.php"]
        }
    }
```

* Most importantly: don't forget to include your vendor/autoload.php file any time you're testing your code!

* If you're still having difficulty, head over to [#composer at Freenode IRC](http://irc.netsplit.de/channels/details.php?room=%23composer&net=freenode). They're a friendly bunch, and quite helpful.

**Update:** I just tried getting Satis going on a Windows machine and had no trouble at all.