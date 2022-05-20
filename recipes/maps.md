# Mapping Glottolog languages

Often a selection of Glottolog languoids needs to be visualized on a geographic map.
https://glottolog.org does this for genealogical subgroups (e.g. the [Central Pacific linkage](https://glottolog.org/resource/languoid/id/cent2060.bigmap.html#3/-14.40/202.69)).

In the following we dicsuss methods to put any set of geocoded languoids (i.e. Glottolog languoids with geographic coordinates) on a map.


## `pyglottolog`'s `htmlmap` subcommand

$ csvgrep -c Parameter_ID -r"^classification" glottolog-cldf/cldf/values.csv | csvgrep -c Value -m "cent2060" | csvcut -c Language_ID > cent2060.txt

$ glottolog htmlmap --glottocodes cent2060.txt --open

While the resulting map (split into a HTML page and a Javascript file with the data in GeoJSON format) can be hacked on to provide further customization,
the `glottolog htmlmap` command itself doesn't provide many options to tune the output.


## `cldfviz.map`

More control over the appearance of the resulting map is possible with the `cldfviz.map` subcommand for [cldfbench](https://github.com/cldf/cldfbench).
And - maybe more importantly - `cldfviz.map` allows creating maps in various image formats, suitable for printing, too.

Alas, `cldfviz.map` requires CLDF data as input.
