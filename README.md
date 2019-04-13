# Static Site Export/Import

### 1. Search & Replace

---

Copy this file down to project directory and call **export.md** for `Search & Replace.` Be sure to exclude the trailing slash(‘/’) from the urls when replacing. Remove this `Search & Replace` section when completed.

    [DOMAIN] = e.g. BusinessAnalytics.com
    [URL] = e.g. www.businessanalytics.com
    [PROTOCOL] = e.g. https://

### 2. Import Site

---

**Commands:**

    wget -m -p -E -nH -H -e robots=off --content-on-error -P '_import' -x "[URL]/sitemap.xml" -x "[URL]/404/" -D[DOMAIN] -X 'comments, /feed, **/feed, **/**/feed, **/**/**/feed, **/**/**/**/feed, wp-json, cdn-cgi' --reject-regex "(\{|(.html|.php|.htm|\/)\?).*?" [URL];
    cd _import;

**Combined Commands:**

    wget -m -p -E -nH -H -e robots=off --content-on-error -P '_import' -x "[URL]/sitemap.xml" -x "[URL]/404/" -D[DOMAIN] -X 'comments, /feed, **/feed, **/**/feed, **/**/**/feed, **/**/**/**/feed, wp-json, cdn-cgi' --reject-regex "(\{|(.html|.php|.htm|\/)\?).*?" [URL]; cd _import;


### Scan Sitemap.xml (Optional)

---

This will scan the sitemap.xml and compare to existing import, if there are additional pages/resources this will capture them. Some pages may not be imported the first time due to no links pointing to them. If the same map is a parent of child sitemap.xml files, you’ll need to run this command twice. If Sitemap is named differently and wasn’t downloaded using the above command, re-download the html to `_imports` directory.

**Commands:**

    rm -rf links.txt;
    for file in **/*.xml; do
       perl -lne "print for /<loc>(.*?)<\/loc>/g" < $file >> links.txt;
    done;
    for dir in $(find * -maxdepth 10 -type d); do
      dir_escaped=$(echo $dir | sed -e 's/[\/&]/\\&/g');
      perl -pi -w -e "s/^http(.*?)$dir_escaped\/?\n$//g" links.txt;
    done;
    wget -m -c -p -E -nH -e robots=off --content-on-error -X 'comments, /feed, **/feed, **/**/feed, **/**/**/feed, **/**/**/**/feed, wp-json, cdn-cgi' --reject-regex "(\{|(.html|.php|.htm|\/)\?).*?" -i links.txt;

**Combined Commands:**

    rm -rf links.txt; for file in **/*.xml; do perl -lne "print for /<loc>(.*?)<\/loc>/g" < $file >> links.txt; done; for dir in $(find * -maxdepth 10 -type d); do dir_escaped=$(echo $dir | sed -e 's/[\/&]/\\&/g'); perl -pi -w -e "s/^http(.*?)$dir_escaped\/?\n$//gi" links.txt; done; wget -m -c -p -E -nH -e robots=off --content-on-error -X 'comments, /feed, **/feed, **/**/feed, **/**/**/feed, **/**/**/**/feed, wp-json, cdn-cgi' --reject-regex "(\{|(.html|.php|.htm|\/)\?).*?" -i links.txt;

### 3. Directory Structure

---

*This transform the directory structure into the preferred file index for our Jekyll set up. For example, a set obtained from WordPress would be imported as:*

    /folder
      index.html
    /folder
      /folder
        index.html
      /folder
        index.html
    /folder
      index.html

... and then will be converted to:

    folderindex.html /folder file.html file.html folderindex.html

**Commands:**

    for file in **/*.htm; do
      new_file=$(echo $file | sed 's/\.htm/\.html/g');
      mv $file $new_file;
    done;
    for dir in $(find * -maxdepth 10 -type d); do
      if [ -f $dir/index.html ]; then
        mv $dir/index.html $dir.html;
      fi;
    done;
    find . -type d -empty -delete;
    for file in **/*.html; do
      new_file=$(echo $file | sed 's/@.*//g' | sed 's/\.php//g');
      mv $file $new_file;
    done;

**Combined Commands:**

    for file in **/*.htm; do new_file=$(echo $file | sed 's/\.htm/\.html/g'); mv $file $new_file; done; for dir in $(find * -maxdepth 10 -type d); do if [ -f $dir/index.html ]; then mv $dir/index.html $dir.html; fi; done; find . -type d -empty -delete; for file in **/*.html; do new_file=$(echo $file | sed 's/@.*//g' | sed 's/\.php//g'); mv $file $new_file; done;

### 4. Front Matter

---

The commands below will cherry pick important variables to include in various spots of our pages, include `<meta>`, `<title>`, and even URL structures.

**Commands:**

    for file in **/*.html; do
        title=$(perl -l -0777 -ne 'print $1 if /<title.*?>\s*(.*?)\s*<\/title/si' $file);
        featured_image=$(perl -l -0777 -ne 'print $1 if /<meta property="og:image" content="(.*?)"/si' $file);
        description=$(perl -l -0777 -ne 'print $1 if /<meta property="og:description" content="(.*?)"/si' $file);
        robots=$(perl -l -0777 -ne 'print $1 if /<meta name="robots" content="(.*?)"/si' $file);
        if [ -z "${description}" ]; then
            description=$(perl -l -0777 -ne 'print $1 if /<meta name="description" content="(.*?)"/si' $file);
        fi;
        permalink=$(echo $file | sed 's/\.html//g' | sed 's/\.php//g' | sed 's/@.*//g');
        if [ "$permalink" != "index" ]; then
            permalink_escaped=$(echo $permalink | sed -e 's/[\/&]/\\&/g');
            permalink="/$permalink/";
        else
            permalink=$(echo $permalink | sed 's/index/:path/g');
        fi;
        title=$(echo $title | sed -e "s/'/\&#039;/g" );
        description=$(echo $description | sed -e "s/'/\&#039;/g");
        perl -lpe "BEGIN{
            print '---';
            print 'title: \"$title\"';
            print 'permalink: $permalink';
            print 'description: \"$description\"';
            print 'robots: \"$robots\"';
            print 'featured_image: \"$featured_image\"';
            print '---';
            print '';
            print '';
        }" "$file" > foo && mv foo "$file";
    done;

**Combined Commands:**

    for file in **/*.html; do title=$(perl -l -0777 -ne 'print $1 if /<title.*?>\s*(.*?)\s*<\/title/si' $file); featured_image=$(perl -l -0777 -ne 'print $1 if /<meta property="og:image" content="(.*?)"/si' $file); description=$(perl -l -0777 -ne 'print $1 if /<meta property="og:description" content="(.*?)"/si' $file); robots=$(perl -l -0777 -ne 'print $1 if /<meta name="robots" content="(.*?)"/si' $file); if [ -z "${description}" ]; then description=$(perl -l -0777 -ne 'print $1 if /<meta name="description" content="(.*?)"/si' $file); fi; permalink=$(echo $file | sed 's/\.html//g' | sed 's/\.php//g' | sed 's/@.*//g'); if [ "$permalink" != "index" ]; then permalink_escaped=$(echo $permalink | sed -e 's/[\/&]/\\&/g'); permalink="/$permalink/"; else permalink=$(echo $permalink | sed 's/index/:path/g'); fi; title=$(echo $title | sed -e "s/'/\&#039;/g" ); description=$(echo $description | sed -e "s/'/\&#039;/g"); perl -lpe "BEGIN{ print '---'; print 'title: \"$title\"'; print 'permalink: $permalink'; print 'description: \"$description\"'; print 'robots: \"$robots\"'; print 'featured_image: \"$featured_image\"'; print '---'; print ''; print ''; }" "$file" > foo && mv foo "$file"; done;

*Note: If you see Front Matter in only the child directories, enable Globstar so Bash Globs work properly.*

### 5. SEO Setup/Cleanup

---

This will remove any SEO meta tags (generated by WP's Yoast SEO plugin) and replace it with `{% include head-seo.html %}`. The `head-seo.html` will be your source for `meta` tags.

**Commands:**

    perl -i -p0e 's/<head(.*?)>/<head$1>\n\n{% include head-seo\.html %}\n/si;' **/*.html;
    perl -i -p0e 's/<title>.*?<\/title>//si;' **/*.html;
    perl -i -p0e 's/<!-- This site is optimized.*?SEO plugin\. -->\s*?\n//si;' **/*.html;
    perl -pi -w -e "s/<meta (name|itemprop|property)=(\"|\')(robots|googlebot|keywords|copyright|title|referrer|author|google-site-verification|msvalidate.*|twitter:.*|description|name|image|og:.*|article:.*|fb:.*)(\"|\').*?>\s*?\n//gi;" **/*.html;
    perl -pi -w -e "s/<link rel=(\"|\')(canonical|shortlink)(\"|\').*?>\s*?\n//i;" **/*.html;
    perl -pi -w -e "s/<script type=(\"|\')application\/ld\+json(\"|\')>.*?<\/script>\s*?\n//i;" **/*.html;

**Combined Commands:**

    perl -i -p0e 's/<head(.*?)>/<head$1>\n\n{% include head-seo\.html %}\n/si;' **/*.html; perl -i -p0e 's/<title>.*?<\/title>//si;' **/*.html; perl -i -p0e 's/<!-- This site is optimized.*?SEO plugin\. -->\s*?\n//si;' **/*.html; perl -pi -w -e "s/<meta (name|itemprop|property)=(\"|\')(robots|googlebot|keywords|copyright|title|referrer|author|google-site-verification|msvalidate.*|twitter:.*|description|name|image|og:.*|article:.*|fb:.*)(\"|\').*?>\s*?\n//gi;" **/*.html; perl -pi -w -e "s/<link rel=(\"|\')(canonical|shortlink)(\"|\').*?>\s*?\n//i;" **/*.html; perl -pi -w -e "s/<script type=(\"|\')application\/ld\+json(\"|\')>.*?<\/script>\s*?\n//i;" **/*.html;

### 6. Update URLs

---

This will scour your imported html files and replace any hard-coded links to the main domain URL with the Jekyll variable `{{ site.url }}` (then the same for rebranded form URLs).

**Commands:**

    perl -pi -w -e 's/https?:\/\/(www.)?[URL]/{{ site.url }}/gi;' **/*.html;
    perl -pi -w -e "s/href=(\"|\')(?!(http|www))(.*?)(\.html|\.htm|\.php)(.*?)(\"|\')/href=\$1\$3\/\$5\$6/gi;" **/*.html;

**Combined Commands:**

    perl -pi -w -e 's/https?:\/\/(www.)?[DOMAIN]/{{ site.url }}/gi;' **/*.html; perl -pi -w -e "s/href=(\"|\')(?!(http|www))(.*?)(\.html|\.htm|\.php)(.*?)(\"|\')/href=\$1\$3\/\$5\$6/gi;" **/*.html;

### 7. Code Cleanup

---

This will remove a few WordPress-related snippets of code like the **feeds**, and multiple references to `w.org`.

**Commands:**

    perl -pi -w -e "s///g;" **/*.html;
    perl -pi -w -e "s/featured_image: \"\{\{ site\.url \}\}/featured_image: \"/g;" **/*.html;
    perl -pi -w -e "s/src=(\"|\')http:\/\//src=\$1https:\/\//g;" **/*.html;
    perl -pi -w -e "s/<link rel=(\"|\')alternate(\"|\') type=(\"|\')(application\/rss\+xml|text\/xml\+oembed|application\/json\+oembed)(\"|\').*?>\n//g;" **/*.html;
    perl -pi -w -e "s/<link rel=(\"|\')https:\/\/api\.w\.org\/(\"|\').*?>\n//g;" **/*.html;
    perl -pi -w -e "s/<link rel=(\"|\')dns-prefetch(\"|\') href='\/\/s\.w\.org(\"|\').*?>\n//g;" **/*.html;
    perl -pi -w -e "s/<script type=(\"|\')text\/javascript(\"|\')>\(?window\.NREUM\|\|\(NREUM=\{\}\).*?<\/script>(\n)?//g;" **/*.html;
    perl -pi -w -e "s/<script type=(\"|\')text\/javascript(\"|\')>try\{ clicky\.init.*?<\/script>(\n)?//g;" **/*.html;
    perl -pi -w -e "s/<script src=(\"|\')(https:|http:)?\/\/pmetrics\.performancing\.com\/js.*?<\/script>(\n)?//g;" **/*.html;
    perl -pi -w -e "s/<noscript><p><img alt=(\"|\')Performancing Metrics.*?<\/noscript>(\n)?//g;" **/*.html;
    perl -i -p0e "s/<script type=(\"|\')text\/javascript(\"|\')>\s*?_stq = window\._stq \|\| \[\]\;.*?<\/script>(\n)?//s;" **/*.html;
    perl -i -p0e "s/<script>\s*?var _prum = \[\[\'id\'.*?<\/script>(\n)?//s;" **/*.html;
    perl -i -p0e "s/<script type=(\"|\')text\/javascript(\"|\')>\s*?window\._wpemojiSettings.*?<\/style>(\n)?//s;" **/*.html;

**Combined Commands:**

    perl -pi -w -e "s///g;" **/*.html; perl -pi -w -e "s/featured_image: \"\{\{ site\.url \}\}/featured_image: \"/g;" **/*.html; perl -pi -w -e "s/src=(\"|\')http:\/\//src=\$1https:\/\//g;" **/*.html; perl -pi -w -e "s/<link rel=(\"|\')alternate(\"|\') type=(\"|\')(application\/rss\+xml|text\/xml\+oembed|application\/json\+oembed)(\"|\').*?>\n//g;" **/*.html; perl -pi -w -e "s/<link rel=(\"|\')https:\/\/api\.w\.org\/(\"|\').*?>\n//g;" **/*.html; perl -pi -w -e "s/<link rel=(\"|\')dns-prefetch(\"|\') href='\/\/s\.w\.org(\"|\').*?>\n//g;" **/*.html; perl -pi -w -e "s/<script type=(\"|\')text\/javascript(\"|\')>\(?window\.NREUM\|\|\(NREUM=\{\}\).*?<\/script>(\n)?//g;" **/*.html; perl -pi -w -e "s/<script type=(\"|\')text\/javascript(\"|\')>try\{ clicky\.init.*?<\/script>(\n)?//g;" **/*.html; perl -pi -w -e "s/<script src=(\"|\')(https:|http:)?\/\/pmetrics\.performancing\.com\/js.*?<\/script>(\n)?//g;" **/*.html; perl -pi -w -e "s/<noscript><p><img alt=(\"|\')Performancing Metrics.*?<\/noscript>(\n)?//g;" **/*.html; perl -i -p0e "s/<script type=(\"|\')text\/javascript(\"|\')>\s*?_stq = window\._stq \|\| \[\]\;.*?<\/script>(\n)?//s;" **/*.html; perl -i -p0e "s/<script>\s*?var _prum = \[\[\'id\'.*?<\/script>(\n)?//s;" **/*.html; perl -i -p0e "s/<script type=(\"|\')text\/javascript(\"|\')>\s*?window\._wpemojiSettings.*?<\/style>(\n)?//s;" **/*.html;

### 8. Invalid UTF-8 Character

---

Check for invalid UTF-8 Characters, find and fix prior to adding the `_imports` into your `_collections` directory.

**Commands:**

    for file in **/*.html; do
      grep -axv '.*' $file;
    done;

**Combined Commands:**

    for file in **/*.html; do grep -axv '.*' $file; done;


### 9. Integrate Import with new stack. :)
