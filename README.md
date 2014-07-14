Crawling a Smithsonian document
-------------------------------

This document describes how to download the individual pages of a Smithsonian online book and assemble them into a PDF.

Before beginning, make sure you have Python (either 2 or 3), `wget`, ImageMagick (`convert`), and PDFtk (`pdftk`).  Also you need the XML file with links to the images to be downloaded.  In this case it's `viewer_Harrington_mf2_r2.xml`.

First, extract the links using a Python script.  Put the following in a file called `extract.py`.

```python
import re

xml = open('viewer_Harrington_mf2_r2.xml').read().strip()
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

This creates a file called `images.txt` containing just the URLs of the images to be downloaded.  Now create a directory for the downloaded images:

```
mkdir image
```

Extract the images using a `bash` script.  Put the following in a file called `get-images.sh`.

```bash
#!/bin/bash

while read i
do
  wget -O - http://ids.si.edu/ids/deliveryService?id=$i > image/$i
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

Inspect the contents of the `image` directory to obtain the range (2 to 751 in this case) and the form of the filenames.  Then run the following from the command line:

```
for i in $(seq -f "%04g" 2 751) ; do convert image/NMNH-Harrington_mf2_r2_$i -quality 25 q25/$i.pdf ; done
```

Finally, stitch together the individual PDFs:

```
pdftk q25/*.pdf cat output harrington-papers-q25.pdf
```

