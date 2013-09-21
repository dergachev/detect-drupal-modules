detect-drupal-modules
=====================

Detect what modules a Drupal site might have installed.

## How it works

The following snippets will sniff out which modules a Drupal site has downloaded, 
by repeatedly making requests to `http://DRUPALSITE/sites/all/modules/MODULENAME/` 
and seeing which come back 403 (Access Denied) rather than 404 (Not found).

## Usage

Copy and paste the following snippet into the terminal, adjusting:

* `URL_ROOT` is the URL to the modules/* folder, eg `https://drupal.org/sites/all/modules` or `http://yourdrupalsite.com/sites/all/modules/contrib`
* `LIMIT` is number of modules to check, out of the top 500
* `PARALLEL` is the number of requests to make at a time

To run the checks:

```bash
export URL_ROOT=https://drupal.org/sites/all/modules LIMIT=100 PARALLEL=20 ; \
  curl -s https://raw.github.com/dergachev/detect-drupal-modules/master/top_500_drupal_modules.txt \
  | head -n $LIMIT \
  | xargs -I '{}' -n1 -P $PARALLEL curl -sL -w "{}--%{http_code}\\n" $URL_ROOT/{} -o /dev/null \
  | sed -n -e '/403/s/--.*//p'
```

Note that because these run in parallel, they're printed in no particular order.

The output will be:

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
