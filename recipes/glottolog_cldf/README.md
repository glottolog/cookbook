# Using the Glottolog data from the CLDF Dataset


## Recipes

- [Listing recent grammars](recent_grammars.md)
- [Mapping Tupi-Guarani subgroups](tg_subgroups.md)
- [Investigating documentation status for a language subgroup](documentation_status_for_subgroup.md)


## Overview

While using the "raw" Glottolog data from [glottolog/glottolog](https://github.com/glottolog/glottolog) is made easier with `pyglottolog`,
there is also an option to access Glottolog data with "off-the-shelve" tools
like excel or csvkit and from other programming environments like R or SQL: 
The [Glottolog CLDF dump](https://doi.org/10.5281/zenodo.3260727) at [DOI: 10.5281/zenodo.3260727](https://doi.org/10.5281/zenodo.3260727).

You can obtain the Glottolog CLDF dataset by
- downloading (and unpacking) an archived version from [Zenodo](https://doi.org/10.5281/zenodo.3260727) (look for "Versions" to access older releases)
- downloading (and unpacking) a release from [GitHub](https://github.com/glottolog/glottolog-cldf/releases)
- or cloning the [repository](https://github.com/glottolog/glottolog-cldf) and checking out a release tag.

Like all [CLDF](https://cldf.cld.org) datasets, this consists of
- a [JSON-LD encoded metadata file](https://github.com/glottolog/glottolog-cldf/blob/master/cldf/cldf-metadata.json)
- a [set of interrelated CSV tables](https://github.com/glottolog/glottolog-cldf/tree/master/cldf)
- and a [bibliography in BibTeX](https://github.com/glottolog/glottolog-cldf/blob/master/cldf/sources.bib)

This is a [CLDF StructureDataset](https://github.com/cldf/cldf/tree/master/modules/StructureDataset), i.e. 
- [Glottolog languoids](https://glottolog.org/meta/glossary#Languoid) along with some basic metadata are listed in the `LanguageTable` in `languages.csv`
- and values of these languoids for various parameters are listed in the `ValueTable` in `values.csv`.

Now, how would we go about the basic task of listing languoids with their values for a particular parameter in one table?

As an example we use the status of languages on the "Agglomerated Endangerment Scale" (AES) as used for [GlottoScope](https://glottolog.org/langdoc/status).

Inspecting the `ParameterTable`, we see that this is the parameter with ID `aes`:
```shell script
$ more cldf/parameters.csv 
ID,Name,Description,type
level,Level,,categorical
category,Category,,categorical
classification,Classification,,
subclassification,Subclassification,,
aes,Agglomerated Endangerment Status,,sequential
med,Most Extensive Description,,sequential
```

With this information, we can inspect the possible values:
```shell script
$ csvgrep -c Parameter_ID -m aes cldf/codes.csv 
ID,Parameter_ID,Name,Description,numerical_value
aes-not_endangered,aes,not endangered,,1
aes-threatened,aes,threatened,,2
aes-shifting,aes,shifting,,3
aes-moribund,aes,moribund,,4
aes-nearly_extinct,aes,nearly extinct,,5
aes-extinct,aes,extinct,,6
```

Since the code IDs are expressive enough, we only need to join information from two tables to get the desired table. We filter the AES values into a temporary table:
```shell script
$ csvgrep -c Parameter_ID -m aes cldf/values.csv > aes_values.csv
```
and join this table to the languoids, filtering only languoids with a status:
```shell script
$ csvjoin -c ID,Language_ID cldf/languages.csv aes_values.csv | csvcut -c ID,Name,Code_ID | csvgrep -c Code_ID -m aes
ID,Name,Code_ID
cani1243,Canichana,aes-extinct
morb1239,Mor (Bomberai Peninsula),aes-moribund
rama1271,Ramanos,aes-extinct
kimk1238,Kimki,aes-shifting
odia1239,Odiai,aes-shifting
yale1246,Yale,aes-shifting
...
```


## Using SQLite to access the CLDF data

While feasible with tools like `csvkit`, interrelated tabular data is - arguably - best accessed using SQL - the Structured Query Language.
And one of the main advantages of the CLDF format lies in the fact that
it can be automatically converted to an SQLite database using the
`pycldf` package:

```shell script
$ cldf createdb cldf/cldf-metadata.json glottolog.sqlite
INFO    <cldf:v1.0:StructureDataset at cldf> loaded in glottolog.sqlite
```

Once converted to SQLite, our desired table is just one query away:
```sql
SELECT
    l.cldf_glottocode AS Glottocode, l.cldf_name AS Name, v.cldf_codereference AS AES
FROM 
    languagetable AS l, valuetable AS v
WHERE
    l.cldf_id = v.cldf_languagereference AND v.cldf_parameterreference = 'aes'
```
which can be run as
```shell script
$ sqlite3 glottolog.sqlite "select l.cldf_glottocode as Glottocode, l.cldf_name as Name, v.cldf_codereference as AES from languagetable as l, valuetable as v where l.cldf_id = v.cldf_languagereference and v.cldf_parameterreference = 'aes' limit 10" -csv -header > glottolog_v4.2.1_aes.csv
```

Note that column names in the SQLite database are the local names of the
corresponding properties in the [CLDF Ontology](https://cldf.clld.org/v1.0/terms.rdf) (i.e. the names after the `#` in the term URL), prefixed with `cldf_`.


## Accessing the Glottolog data from R

While reading the CSV tables of a CLDF dataset from R is possible, doing this in a standard-compliant way would
require 
- first getting the table's file name according to the `propertyUrl` in the metadata
- then consulting the table's CSV dialect spec in the metadata,
- then converting the dialect spec into appropriate argument for a CSV reader function.

All this can be circumvented by accessing the data in a SQLite database as created above:

```r
> library(RSQLite)
> conn <- dbConnect(RSQLite::SQLite(), "glottolog.sqlite")
> dbListTables(conn)
[1] "CodeTable"              "LanguageTable"          "ParameterTable"        
[4] "SourceTable"            "ValueTable"             "ValueTable_SourceTable"
> dbGetQuery(conn, "SELECT * FROM LanguageTable LIMIT 10")
    cldf_id              cldf_name cldf_macroarea cldf_latitude cldf_longitude
1  more1255                    Yam                           NA             NA
2  mong1349        Mongolic-Khitan                           NA             NA
3  kolp1236 Kol (Papua New Guinea)      Papunesia      -5.21042        151.583
4  naml1239          Namla-Tofanma                           NA             NA
5  tana1288             Tanahmerah      Papunesia      -2.55230        133.180
6  jara1244            Jarawa-Onge                           NA             NA
7  gong1255           Ta-Ne-Omotic                           NA             NA
8  pomo1273                 Pomoan                           NA             NA
9  west2434           Western Daly                           NA             NA
10 yang1287              Yangmanic                           NA             NA
   cldf_glottocode cldf_iso639P3code Countries Family_ID Language_ID
1         more1255              <NA>                <NA>        <NA>
2         mong1349              <NA>                <NA>        <NA>
3         kolp1236               kol        PG      <NA>        <NA>
4         naml1239              <NA>                <NA>        <NA>
5         tana1288               tcm        ID      <NA>        <NA>
6         jara1244              <NA>                <NA>        <NA>
7         gong1255              <NA>                <NA>        <NA>
8         pomo1273              <NA>                <NA>        <NA>
9         west2434              <NA>                <NA>        <NA>
10        yang1287              <NA>                <NA>        <NA>
```
