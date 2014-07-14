Crawling a Smithsonian book
---------------------------

This document describes how to download the individual pages of a Smithsonian online book and assemble them into a PDF.

Before beginning: 

* Pull up a terminal.  
* Make sure you are running `bash`.  If in doubt, run `echo $SHELL` and if you not see `/bin/bash` then run `bash`.
* Make sure you have Python, `wget`, ImageMagick (`convert`), and `pdftk`.  
* Get the XML file with links to the images to be downloaded.  In this case it's `viewer_Harrington_mf2_r2.xml`.

First, extract the links using a Python script.  Put the following in a file called `extract.py`.

```python
import re

xml = open('viewer_Harrington_mf2_r2.xml').read()
fo = open('images.txt', 'w')

link_pat = re.compile(r'<NAME>http://ids.si.edu/ids/deliveryService\?id=(.+?)</NAME>')

for matches in link_pat.findall(xml):
  fo.write(matches + '\n')

fo.close()
```

Run it:

```
python3 extract.py
```

This creates a file called `images.txt` containing just the filenames of the images to be downloaded.  Now create a directory for the downloaded images:

```
mkdir image
```

Extract the images using a `bash` script.  Put the following in a file called `get-images.sh`.

```bash
#!/bin/bash

while read i
do
  wget -O image/$i http://ids.si.edu/ids/deliveryService?id=$i
  sleep 10
done < images.txt
```

Then run:

```
./get-images.sh
```

This can take hours.  It pauses for 10 seconds between downloading each file, to avoid being blocked.  When this completes, convert the images to PDFs while reducing their quality.  First create a directory for the results:

```
mkdir q25
```

Then run the following from the command line:

```bash
for i in image/* ; do convert $i -quality 25 q25/$(basename $i).pdf ; done
```

Finally, stitch together the individual PDFs:

```
pdftk q25/*.pdf cat output harrington-papers-q25.pdf
```

