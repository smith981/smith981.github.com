---
layout: post
title: "Zend Certified Engineer"
description: ""
category: 
tags: [Zend, PHP, certifications]
---
{% include JB/setup %}
After much preparation, I earned my Zend Certified Engineer designation in PHP 5.3 yesterday. The test was fairly difficult, but to Zend's credit it did follow the manual and their training materials. That said, I was struck by how much the test questions were geared towards interpreting confusing code by hand-- code which would never have been written had best practices been followed. Nevertheless, if you are interested in this certification, I highly recommend Zend's recorded training, as there is no better preparation (unless of course you can memorize the PHP manual).

You will certainly learn a lot about the language if you pursue this designation. One of my favorite discoveries is that PHP interprets integers with a leading zero as octal numbers, and failing that, will return 0. For instance:

```php
<?php

$a = 05; // 5
$b = 06; // 6
$c = 07; // 7
$d = 08; // 0, because there is no '8' in octal notation
```

Fun stuff which you'd never use unless you were preparing for a certification.