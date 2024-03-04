# How to download a site from the Archive.org Wayback Machine

~~~ shell
WEB_SITE='https://blog.boochtek.com/'
TIME='20230101000000'

git clone git@github.com:hartator/wayback-machine-downloader.git
cd wayback-machine-downloader
cd ..

brew install waybackpy
waybackpy --url $WEB_SITE --from 2023-01-01 --known_urls > urls.txt

# Add Wayback Machine URL prefix to the URLs.
awk -v prefix="https://web.archive.org/web/$TIME/" '{print prefix $0}' urls.txt > wayback-urls.txt

# Figured out --adjust-extension via https://lists.gnu.org/archive/html/bug-wget/2015-06/msg00021.html.
# NOTE: We might find it useful to add `--directory-prefix=$TARGET_DIR`.
wget --input-file=wayback-urls.txt --continue --wait=4 --retry-connrefused \
    --force-directories --no-host-directories --cut-dirs=5 --adjust-extension
  # --directory-prefix=.

# Convert URLs in all files to remove the Wayback Machine URL prefix.
sed -i '' -e "s|https://web.archive.org/web/.*/||g" $(find . -type f)

rm -rf 201* index.html

# Convert all HTML files into MarkDown, recursively.
if ! brew list | grep pandoc ; then
    uninstall_pandoc=1
    brew install pandoc
fi
find . -type f -name '*.html' -exec pandoc -f html -t markdown -o {}.md {} \;

wayback_machine_downloader $WEB_SITE -l -a --from 2023-01-01 > wayback-urls.txt
wget --input-file=wayback-urls.txt --continue --wait=4 --retry-connrefused

brew uninstall waybackpy
if [ -n "$uninstall_pandoc" ]; then
    brew uninstall pandoc
fi
rm -rf wayback-machine-downloader
rm urls.txt wayback-urls.txt
~~~
