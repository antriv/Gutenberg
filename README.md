# gutenberg

_So you want to use project gutenberg text?_

This package doesn't currently work (install problems with the BSD database that
happens even on python2) and doesn't provide the functionality I wanted. I used
code from this repo to clean up the headers from Gutenberg files (step 2) I
scraped using some bash commands (step 1).

## step 1: scrape

```bash
# 1. Scrape all 'en' books from a gutenberg mirror into a single directory. Took
#    3h 25m; total size was 11G. Run on Nov 17, 2016. All downloads are ".zip".
 wget -m -H -nd "http://www.gutenberg.org/robot/harvest?filetypes[]=txt&langs[]=en"

# 2. Remove extra crap we get from using wget.
rm robots.txt
rm harvest*

# 3. Remove all duplicate encodings in ISO-<something> and UTF-8. Based on a few
#    random samplings it seemed like there were ASCII versions of all of these.
#    (NOTE: after trying to process the remainders, it seems like many aren't
#    actually ASCII after all. Oh well.)
ls | grep "-8.zip" | xargs rm
ls | grep "-0.zip" | xargs rm

# 4. We were then left with a handful of other files with '-' characters in them
#    (which seemed to be an indication of non-standard formatting).
#     - 89-AnnexI.zip          - trade agreement doc; weird text
#     - 89-Descriptions.zip    - ""
#     - 89-Contents.zip        - ""
#     - 3290-u.zip             - unicode but with "u" instead of "0" suffix
#     - 5192-tex.zip           - tex formatted book
#     - 10681-index.zip        - thesaurus index
#     - 10681-body.zip         - thesaurus body
#     - 13526-page-inames.zip  - (I forget)
#     - 15824-h.zip            - windows-encoded file (I think)
#     - 18251-mac.zip          - mac-encoded file (I think)
#    I removed all of them
rm 89-AnnexI.zip
rm 89-Descriptions.zip
rm 89-Contents.zip
rm 3290-u.zip
rm 5192-tex.zip
rm 10681-index.zip
rm 10681-body.zip
rm 13526-page-inames.zip
rm 15824-h.zip
rm 18251-mac.zip

# 5. unzip all of the files and remove all of the zips.
sudo apt install unzip
unzip "*.zip"
rm "*.zip"

# 6. From foo.zip, some files extract directly into foo.txt, while other extract
#    into foo/foo.txt. Move all into this directory.
mv */*.txt ./

# 7. There will be empty directories left (what we want), and some non-empty
#    ones. The non-empty directories include other formats (e.g. PDF), sneaky
#    nested zips, sneaky nested zips with other formats in them, at least one
#    typo (.TXT), and possibly other things. There are only 20 such directories
#    (including several pdf/mid pairings of the same serial number), so we just
#    remove them along with the empty directories.
ls | grep -v "\.txt" | xargs rm -rf


# The final size of original/ is 37,229 .txt files totaling 14G.
```

## step 2: clean using this repo

I'm using `gutenberg/cleanup/strip_headers.py` to take off the inconsistent
mountain of crap above and below the texts.

```bash
# setup pip crap if you don't normally use python 3
pip install --upgrade pip
pip install virtualenv
virtualenv -p python3 venv
source venv/bin/activate
pip3 install six
pip3 install tqdm

# run. <indir> contains all of your downloaded .txt files. <outdir> is where the
# script dumps the (relatively) cleaned versions.
python3 clean.py <indir> <outdir>

# When I ran it, the above has encoding problems opening many of the files. I
# probably could have tried harder to fix these but believe it or not getting
# this text data has been about as fun as pulling my own teeth out so I decided
# to just let it go.
#
# What remained are 36,154 of the original 37,229 files, so about 97% of them.
```
