---
layout: post
title: "Connecting to SQL Server from PHP on Mac OS/X Lion+"
description: ""
category: 
tags: [PHP, SQL Server, OS/X, FreeTDS, ODBC]
---
{% include JB/setup %}
There are several tutorials on the web that you can follow in order to get PHP 5.4 on OS/X to connect to MS SQL Server.

While helpful, some of them contain *bad or incomplete information*.

I recommend using [FreeTDS](http://www.freetds.org). I recommend generally following this [tutorial](http://featurebug.blogspot.com/2011/01/mac-os-x-php-zend-server-ce-freetds-and.html), but with a few caveats:

* The tutorial uses Zend Server CE's web stack, which I no longer use. Instead, I used [Liip's](http://www.liip.ch/de) super-simple [PHP binary package](http://php-osx.liip.ch). This will save you from having to re-compile PHP with ODBC support, or hunt down a packaged version of PHP that comes with it (I couldn't find one that worked). This PHP package works with the OS/X bundled Apache. That means you need to be placing your web documents in /Library/WebServer/Documents in OS/X Lion or Mountain Lion.

* Several tutorials specify incomplete or incorrect odbc.ini files. A proper odbc.ini DSN entry for SQL Server 2005 and 2008 looks like this:
```
[YourDSN]
Driver = FreeTDS
TDS_Version = 7.2
Description = Your Database
Server = yourhost.yourdomain.com
Port = 1433
Database = DatabaseName
Trace = No
```

Obviously, fill in *YourDSN*, *Your Database*, *yourhost.yourdomain.com*, and *DatabaseName* with the appropriate values for the database you want to connect to.

* Once you get odbc.ini and odbcinst.ini created using the Featurebug tutorial above, you'll need to set two environment variables, which point PHP to the correct paths:
```php
<?php
putenv("ODBCINSTINI=/opt/local/etc/odbcinst.ini");
putenv("ODBCINI=/opt/local/etc/odbc.ini");
```
You can probably do this in httpd.conf or in php.ini instead.

You can test your work by turning error reporting on and using the [odbc_connect()](http://us1.php.net/odbc_connect) function.