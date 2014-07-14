Crawling a Smithsonian book (arcane version)
--------------------------------------------

This document describes how to download the individual pages of a Smithsonian online book and assemble them into a PDF.

Before beginning: 

* Pull up a terminal running `bash`.  
* Make sure you have `xmlstarlet`, `wget`, ImageMagick (`convert`), and `pdftk`.  
* Get the XML file with links to the images to be downloaded.  In this case it's `viewer_Harrington_mf2_r2.xml`.

Extract the links by running:

```bash
xmlstarlet sel -T -t -m '//NAME' -v 'text()' -n viewer_Harrington_mf2_r2.xml > images.txt
```

Extract the images:

```bash
(mkdir -p image; cd image; while read i ; do wget -O ${i#*=} $i ; sleep 10 ; done) < images.txt
```

Convert the images to individual PDFs:

```bash
mkdir -p q25; for i in image/* ; do convert $i -quality 25 q25/$(basename $i).pdf ; done
```

Finally, stitch together the individual PDFs:

```bash
pdftk q25/*.pdf cat output harrington-papers-q25.pdf
```

