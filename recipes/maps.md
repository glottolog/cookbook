# Mapping Glottolog languages

Often a selection of Glottolog languoids needs to be visualized on a geographic map.
https://glottolog.org does this for genealogical subgroups (e.g. the [Central Pacific linkage](https://glottolog.org/resource/languoid/id/cent2060.bigmap.html#3/-14.40/202.69)).

In the following we dicsuss methods to put any set of geocoded languoids (i.e. Glottolog languoids with geographic coordinates) on a map.


## `pyglottolog`'s `htmlmap` subcommand

Make sure you get the latest `pyglottolog` from [PyPI](https://pypi.org/project/pyglottolog/):
```shell
$ pip install -U pyglottolog
```

Next, we need a list of Glottocodes as input. While any list - one per line - in a text file will do, we may also replicate Glottolog's "subgroup-map" functionality. To get a list Glottocodes for anything in a subgroup, we use the [Glottolog CLDF data](https://github.com/glottolog/cookbook/blob/master/recipes/glottolog_cldf/README.md) (and the tools of [csvkit](https://csvkit.readthedocs.io/en/latest/)). The following chain of commands will
- pick out all values for the "classification" parameter from the dataset,
- filter out anything that does not have `cent2060` in its list of ancestors,
- reduce the output to just the `Language_ID` column:

```shell
$ csvgrep -c Parameter_ID -r"^classification" glottolog-cldf/cldf/values.csv | 
csvgrep -c Value -m "cent2060" | 
csvcut -c Language_ID > cent2060.txt
```

The resulting file `cent2060.txt` is suitable as input for the `htmlmap` command:
```shell
$ glottolog htmlmap --glottocodes cent2060.txt --open
```

The map you should see now in your browser looks rather unimpressive - although it does provide nice interactive functionality like popups with a bit of language metadata when clicking on dots.
The resulting map (split into a HTML page and a Javascript file with the data in GeoJSON format) can be hacked on to provide further customization, but
the `glottolog htmlmap` command itself doesn't provide many options to tune the output.


## `cldfviz.map`

More control over the appearance of the resulting map is possible with the `cldfviz.map` subcommand for [cldfbench](https://github.com/cldf/cldfbench).
And - maybe more importantly - `cldfviz.map` allows creating maps in various image formats, suitable for printing, too.

Alas, `cldfviz.map` requires CLDF data as input.

But our file `cent2060.txt` from above isn't too far from a [metadata-free CLDF StructureDataset](https://github.com/cldf/cldf#metadata-free-conformance).
In fact, all we have to do is add three more columns to it: `Parameter_ID`, `Value` and `ID` and call the result `values.csv`:

```shell
sed '1s/$/,Parameter_ID/; 2,$s/$/,p/' glottocodes.csv |
sed '1s/$/,Value/; 2,$s/$/,v/' |
awk -v ln=1 'NR==1{print $0 ",ID" } NR!=1{print $0 "," ln++}' >
values.csv 
```


## Still more control

If you open the "data" file created by `glottolog htmlmap` above, it should start like
```json
var geojson = {
    "features": [
        {
            "geometry": {
                "coordinates": [
                    177.066,
                    -12.5008
                ],
                "type": "Point"
            },
            "id": "rotu1241",
            "properties": {
                "name": "Rotuman",
                "color": "#F2F3F4",
                "family": "Austronesian",
                "family_id": "aust1307"
            },
            "type": "Feature"
        },
```

If you remove the `var geojson = ` and everything after the closing `}` for the `geojson` data (i.e. the `;` and everything from line starting with `var markers`), you end up with a file in [GeoJSON format](https://geojson.org/).
Such files can often be imported in GIS software like
[qgis or arcgis](https://opengislab.com/blog/2018/11/8/adding-and-viewing-geojson-in-qgis-and-arcgis) - giving you unlimited options for customization.
