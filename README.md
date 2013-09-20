detect-drupal-modules
=====================

Detect what modules a Drupal site might have installed.

## How it works

The following snippets will sniff out which modules a Drupal site has downloaded, 
by repeatedly making requests to `http://DRUPALSITE/sites/all/modules/MODULENAME/` 
and seeing which come back 403 (Access Denied) rather than 404 (Not found).

## Usage

Copy and paste either of the following snippets into the terminal, adjusting:

* `LIMIT` is number of modules to check
* `URL_ROOT` is the URL to the modules/* folder, eg `https://drupal.org/sites/all/modules` or `http://yourdrupalsite.com/sites/all/modules/contrib`

To run the checks serially (can take a while!):

```
export LIMIT=100 URL_ROOT=https://drupal.org/sites/all/modules/ ; \
  curl -s https://raw.github.com/dergachev/detect-drupal-modules/master/top_500_drupal_modules.txt \
  | head -n $LIMIT \
  | while read line; do echo $line "--" $(curl -s -I $URL_ROOT/$line/ | head -n 1) ; done \
  | sed -n -e '/403/s/ .*//p'
```

To run the checks in parallel (much faster; requires installing GNU Parallel):

```
export LIMIT=100 URL_ROOT=https://drupal.org/sites/all/modules/ ; \
  curl -s https://raw.github.com/dergachev/detect-drupal-modules/master/top_500_drupal_modules.txt \
  | head -n $LIMIT \
  | parallel --keep-order 'echo {}--$(curl -s -I $URL_ROOT/\{}/ | head -n 1)' \
  | sed -n -e '/403/s/--.*//p'
```

In both cases, the output will be:

```
views
ctools
google_analytics
link
cck
devel
imageapi
filefield
imagefield
features
imagecache
views_bulk_operations
jquery_ui
image
jcarousel
diff
```

## Warnings 

* BE CAREFUL NOT TO PUT UNDUE STRESS ON SERVERS YOU DONT OWN
* BE WARY OF THROTTLING/GETTING BLOCKED (might give false positives or negatives)

## Generating top_500_drupal_modules.txt

Visit https://drupal.org/project/usage and run the following in the JS console:

```
jQuery('#project-usage-all-projects tbody a').each(function(i) {
  var name = this.href.replace(/^.*usage./, ""); 
  if(i<500) { console.log(name);}
});
``` 

## Installing GNU Parallel

### On Mac OS X

```
brew install parallel
```

### On Ubuntu 12.04

From http://askubuntu.com/a/227788/194314 and http://askubuntu.com/a/298598/194314

```
echo "deb http://archive.ubuntu.com/ubuntu precise-backports main restricted universe multiverse" >> /etc/apt/sources.list
apt-get update

sudo apt-get install parallel
sudo rm /etc/parallel/config
```

See http://static.usenix.org/publications/login/2011-02/pdfs/Tange.pdf for GNU Parallel examples.
