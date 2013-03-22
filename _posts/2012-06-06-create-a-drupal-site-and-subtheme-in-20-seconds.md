---
layout: post
title: "Create a Drupal Site and Subtheme in 20 Seconds"
description: ""
category: 
tags: [OS/X, Zend Server, Drupal, Drush, OM]
---
{% include JB/setup %}
When I first started developing Drupal sites, I created each new site manually. I would download Drupal 7, download my preferred base theme, manually subtheme it, and then install all of my favorite modules. This could easily turn into hours worth of tedious work.

I now use a [Drush][1] makefile and a shell script to generate a new site and subtheme in seconds.

When [someone][2] finally convinced me to take the plunge and try Drush, I spent half an hour learning it, and another half hour creating my own personal Drush makefile. It was time well spent. If you've never used this feature in Drush, a makefile tells Drush things like: 

* Which version of Drupal to install
* Which themes you want installed
* Which modules to download

Using a makefile cut my site creation time down to about 5 minutes, since I still had to create the database and manually sub theme [OM][3] (my preferred base theme). Hence, I created a shell script to handle this for me. Note: you can easily modify both scripts to work with the Zen base theme if you prefer.
    
Put both of these in the web root of your **secured development machine.** I use [Zend Server CE][4] as my web stack over OS/X, so I can only guarantee it works in this environment (though I see no reason why it wouldn't work elsewhere).
    
Once they are in place and (assuming you've previously installed Drush), you should be able to create a site as easily as:<bash> yourname@yourmac/usr/local/zend/apache2/htdocs$ drush make eds\_drush\_makefile.make newsitename yourname@yourmac/usr/local/zend/apache2/htdocs$./setup\_new\_site.sh newsitename </bash> 
    
**Update:** I forgot to mention that although the above steps create the database, you will still have to create the database user. This can be done from the command line in less than a minute. Then, don't forget to add the connection info to settings.php.

*eds_drush_makefile.make*

```ini
; $Id$;
; Ed Smith's Makefile
; ----------------
; This is an example makefile to introduce new users of drush_make to the
; syntax and options available to drush_make. For a full description of all
; options available, see README.txt.

; This make file is a working makefile - try it! Any line starting with a `;`
; is a comment.

; Core version
; ------------
; Each makefile should begin by declaring the core version of Drupal that all
; projects should be compatible with.

core = 7.x

; API version
; ------------
; Every makefile needs to declare it's Drush Make API version. This version of
; drush make uses API version `2`.

api = 2

; Core project
; ------------
; In order for your makefile to generate a full Drupal site, you must include
; a core project. This is usually Drupal core, but you can also specify
; alternative core projects like Pressflow. Note that makefiles included with
; install profiles *should not* include a core project.

; Use pressflow instead of Drupal core:
; projects[pressflow][type] = "core"
; projects[pressflow][download][type] = "file"
; projects[pressflow][download][url] = "http://launchpad.net/pressflow/6.x/6.15.73/+download/pressflow-6.15.73.tar.gz"

; CVS checkout of Drupal 6.x core:
; projects[drupal][type] = "core"
; projects[drupal][download][type] = "cvs"
; projects[drupal][download][root] = ":pserver:anonymous:anonymous@cvs.drupal.org:/cvs/drupal"
; projects[drupal][download][revision] = "DRUPAL-6"
; projects[drupal][download][module] = "drupal"

; CVS checkout of Drupal 7.x. Requires the `core` property to be set to 7.x.
; projects[drupal][type] = "core"
; projects[drupal][download][type] = "cvs"
; projects[drupal][download][root] = ":pserver:anonymous:anonymous@cvs.drupal.org:/cvs/drupal"
; projects[drupal][download][revision] = "HEAD"
; projects[drupal][download][module] = "drupal"

projects[] = drupal

; Projects
; --------
; Each project that you would like to include in the makefile should be
; declared under the `projects` key. The simplest declaration of a project
; looks like this:

; To include the most recent views module:

projects[] = views
projects[] = views_slideshow
;projects[] = views_nivo_slider
projects[] = ds
projects[] = devel
projects[] = devel_themer
projects[] = libraries
projects[] = token
projects[] = pathauto
projects[] = ctools
projects[] = email
projects[] = om
projects[] = image_url_formatter
projects[] = fontyourface
projects[] = quicktabs
projects[] = date
;projects[] = date_popup
projects[] = link
projects[] = colorbox
projects[] = page_title

; This will, by default, retrieve the latest recommended version of the project
; using its update XML feed on Drupal.org. If any of those defaults are not
; desirable for a project, you will want to use the keyed syntax combined with
; some options.

; If you want to retrieve a specific version of a project:

;projects[cck] = 2.6

; Or an alternative, extended syntax:

;projects[ctools][version] = 1.3

; Check out the latest version of a project from CVS. Note that when using a
; repository as your project source, you must explictly declare the project
; type so that drush_make knows where to put your project.

;projects[data][type] = module
;projects[data][download][type] = cvs
;projects[data][download][module] = contributions/modules/data
;projects[data][download][revision] = DRUPAL-6--1

; Clone a project from github.

;projects[tao][type] = theme
;projects[tao][download][type] = git
;projects[tao][download][url] = git://github.com/developmentseed/tao.git

; If you want to install a module into a sub-directory, you can use the
; `subdir` attribute.

;projects[admin_menu][subdir] = custom

; To apply a patch to a project, use the `patch` attribute and pass in the URL
; of the patch.

;projects[admin_menu][patch][] = "http://drupal.org/files/issues/admin_menu.long_.31.patch"
```

*setup_new_site.sh*

```bash
#the following script can be run from htdocs to create a database with the same name as your site folder, and make a new OM sub theme bearing the same name
#Ideally, this gets run after a Drush Make install using my favorite eds_drush_makefile.make
#------------
#!/bin/sh
cd $1/sites/default;
cp default.settings.php settings.php;
chmod 777 settings.php;
mkdir files;
chmod 777 files;
cd ../all/themes;
mv om/om_html5_subtheme $1;
mv $1/om_html5_subtheme.info $1/$1.info;
#the next line creates the database
/usr/local/zend/mysql/bin/mysqladmin -S /usr/local/zend/mysql/tmp/mysql.sock -uroot create $1;
```

**Update:** If you use a password for your development database's root account, add "-pyourpassword" to the mysqladmin command in setup_new_site.sh.

[1]: http://www.drush.org
[2]: http://www.nickvahalik.com
[3]: http://drupal.org/project/om
[4]: http://www.zend.com/en/products/server-ce/