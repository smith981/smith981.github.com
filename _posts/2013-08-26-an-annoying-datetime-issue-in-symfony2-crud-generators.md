---
layout: post
title: "An Annoying DateTime Issue in Symfony2 CRUD Generators"
description: "Or \"Why You Should Always Set the Default Timezone in PHP\""
category: 
tags: [PHP, Symfony2, CRUD, DateTime]
---
{% include JB/setup %}
While building a Symfony 2.3 bundle using [the excellent Crud Generator from MWSimple](https://packagist.org/packages/mwsimple/crud-generator), I kept getting this fatal error when trying to create a new object with a DateTime field:

    DateTime::__construct(): Failed to parse time string (2008-10-10 0:0:0 CST6CDT) at position 20 (6): Unexpected character 

After a lot of googling, I found that the timezone stamp was not being recognized by Symfony2. While "CST6CDT" is technically a valid timezone, [it is listed under "Others" in the PHP manual, and we are discouraged from using it](http://php.net/manual/en/timezones.others.php). As you probably know, when you create a new DateTime object, the current timezone is used.

After even more googling, it occurred to me that I had not set the default timezone anywhere in my code. I never found out why it defaults to "CST6CDT" or even what this timezone actually means. But adding this to my AppKernel.php solved my problem:

```php
<?php

class AppKernel extends Kernel
{
	/* Add this method to set the default timezone */
    public function init()
    {
        date_default_timezone_set( 'America/Chicago' );
        parent::init();
    }

    public function registerBundles()
    [...]
```