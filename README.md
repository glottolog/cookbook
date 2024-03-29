# Glottolog Cookbook

This cookbook contains recipes to tackle common tasks involving 
[Glottolog data](https://github.com/glottolog/glottolog),
typically using the Python library [`pyglottolog`](https://github.com/glottolog/pyglottolog).

- [Glottolog data as CLDF](recipes/glottolog_cldf/README.md): Accessing Glottolog data as CLDF dataset.
  - to [list recent grammars](recipes/glottolog_cldf/recent_grammars.md)
- [Putting Glottolog languages on maps](recipes/maps.md)
- [Examining changes across Glottolog releases](recipes/changes.md).
- [Computing MEDs over time](recipes/med_over_time.md).
- [treemaker](treemaker): Extracting a tree for a given set of languoids from the global tree.
- [`recipes/locations_of_child_languages.py`](recipes/locations_of_child_languages.py) is a script to extract locations for all languages in a given clade. It must be invoked specifying the local path to a clone of the Glottolog repository, the glottocode of the clade and the name of the CSV file to which to write the data, e.g. - if
invoked from within the `cookbook` directory of a repository clone:
  ```
  $ python locations_of_child_languages.py .. narr1281 narrow_bantu_locations.csv
  ```
  will result in a CSV file as follows:
  ```
  $ head narrow_bantu_locations.csv 
  name,glottocode,latitude,longitude
  Bube,bube1242,3.53638,8.68929
  Chikunda,kund1255,-15.7337,30.2804
  ```

