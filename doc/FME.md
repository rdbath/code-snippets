FME
===

* [Batch process](#batch-process)
    * [Use multiple source datasets](#use-multiple-source-datasets)
* [Python scripts](#python-scripts)
    * [Check if file exists](#check-if-file-exists)
    * [Remove accents](#remove-accents)
* [Expressions](#expressions)
    * [Check if attribute is integer](#check-if-attribute-is-integer)
* [Remove geometry dimension](#remove-geometry-dimension)
* [Coordinates systems](#coordinates-systems)
    * [Projections](#projections)
    * [Transformers](#transformers)

Batch process
-------------

### Use multiple source datasets

```batchfile
for %%f in (*.ext) do fme <workspace>.fmw --parameter "%%f"
```

Python scripts
--------------

### Check if file exists

```python
import fmeobjects
import os

def processFeature(feature):
    doesFileExist = os.path.exists(feature.getAttribute("path"))
    feature.setAttribute("file_exists", doesFileExist)
```

### Remove accents

```python
import fmeobjects
import unicodedata as ud

def rmdiacritics(char):
    '''
    Return the base character of char, by "removing" any
    diacritics like accents or curls and strokes and the like.
    '''
    desc = ud.name(unicode(char))
    cutoff = desc.find(' WITH ')
    if cutoff != -1:
        desc = desc[:cutoff]
    return ud.lookup(desc)

def removeAccents(feature):
    attribute_list = ("name", "type", "state") # Modify as needed
    for attrib in feature.getAllAttributeNames():
        if attrib in attribute_list:
            value = feature.getAttribute(attrib)
            if value:
                value = unicode(value)
                new_value = ''.join([rmdiacritics(char) for char in value])
                feature.setAttribute(attrib, new_value)
```

Expressions
-----------

### Check if attribute is integer

```python
(@Value(attribute) == int(@Value(attribute))) ? 1 : 0
```

Remove geometry dimension
-------------------------

* Elevation: `2DForcer`
* Measure: `MeasureRemover`

Coordinates systems
-------------------

### Projections

MN03
- `EPSG:21781`
- CH1903 / LV03

MN95
- `EPSG:2056`
- CH1903+ / LV95 (Swiss CH1903+ / LV95)

### Transformers

ReframeReprojector
- FINELTRA (`reframereproject.dll`)
- Triangles
- ~ cm

Reprojector
- NTv2 (`CHENYX06.gsb`)
- Grids
- ~ dm
