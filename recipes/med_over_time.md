# Glottolog's MED over time

Glottolog's [GlottoScope](https://glottolog.org/langdoc/status/browser?macroarea=Africa&focus=sdt) allows you to investigate the descriptive status languages, coded as [Most Extensive Description](https://glottolog.org/meta/glossary#sec-mostextensivedescriptionmed),
over time.

As of Glottolog 4.5, the CLDF dataset at https://doi.org/10.5281/zenodo.3260727 includes this data as parameter `medovertime`. The `Value` column of
the `ValueTable` contains a python list serialized as string giving the text citations of the MEDs from latest to earliest, and the `Source` column
lists the keys of these MEDs to look up metadata in the accompanying sources file.

Thus, what the recipe below does can also be done in one (longish) shell command:
```shell
$ csvgrep -c Parameter_ID -m"medovertime" cldf/values.csv | \
  csvgrep -c Language_ID -m"stan1293" | \
  csvcut -c Value | \
  sed 1d | \
  sed --expression='s/"//g' | \
  sed --expression="s/'/\"/g"| \
  jq 'reverse | map(.[0:70])'
[
  "Lily, William. 1549. A Shorte Introduction of Grammar. London: R. Wolf",
  "Greaves, Paul. 1594. Grammatica Anglicana, praecipue quatenus a Latina",
  "Jonson, Benjamin. 1909 [1640]. The English Grammar made by Ben Jonson.",
  "Cooper, Christopher. 1685. Grammatica Linguae Anglicanae. Londres: J. ",
  "J. Wright. 1892. A Grammar of the Dialect of Windhill, in the West Rid",
  "Nida, Eugene Albert and Elson, Benjamin. 1962. A synopsis of English s",
  "Dutton, Tom E. 1966. The informal speech of Palm Island Aboriginal chi",
  "Shorrocks, Graham. 1980. A grammar of the dialect of Farnworth and dis",
  "Quirk, Randolph and Sidney Greenbaum and Geoffrey Leech and Jan Svartv",
  "Huddleston, Rodney D. and Pullum, Geoffrey K. 2002. The Cambridge gram"
]
```

This is somewhat expensive to compute, since it requires access to the
bulk of Glottolog data. Glottoscope does this on-the-fly, because it has
access to all data in a highly queryable relational database.
Nevertheless, we can also do this using `pyglottolog` and an export or clone
of [glottolog/glottolog](https://github.com/glottolog/glottolog).

In the following we compute the MED over time for [English](https://glottolog.org/resource/languoid/id/stan1293).

First, we acquire access to the Glottolog API:
```python
from pyglottolog import Glottolog

api = Glottolog('PATH/TO/glottolog/glottolog')
```

Now we can retrieve the `Languoid` object for English:
```python
l = api.languoid('stan1293')
```

[References](https://github.com/glottolog/pyglottolog/blob/7a1a43e4361ca909bc04a8419eceb34b0830c1f7/src/pyglottolog/languoids/models.py#L112) tagged for a `Languoid` are - somewhat confusingly - accessible via the `sources` attribute:

```python
>>> len(l.sources)
418
```

We are interested in the actual [bibiography entries](https://github.com/glottolog/pyglottolog/blob/7a1a43e4361ca909bc04a8419eceb34b0830c1f7/src/pyglottolog/references/bibfiles.py#L217), though. We can get these from
the references as follows (requiring reading data from the BibTeX files):
```python
sources = [ref.get_source(api) for ref in l.sources]
```

Bibliography `Entry`s support [sorting by MED weight](https://github.com/glottolog/pyglottolog/blob/7a1a43e4361ca909bc04a8419eceb34b0830c1f7/src/pyglottolog/references/bibfiles.py#L237-L265). Thus, we can
order `sources` from least to most extensive:
```python
sources = sorted(sources)
```

What remains to do, is weeding out "newer but worse" sources:
```python
meds = []
last_year = 10000
for source in reversed(sources):  # go through sources from "best" to "worst"
    if source.year_int and source.year_int < last_year:  # pick the next earlier source:
        last_year = source.year_int
        meds.append(source)
```

Now let's see what we got:
```python
import textwrap
for med in reversed(meds):
    print('{}\t{}\t{}'.format(med.year_int, med.med_type.name, textwrap.shorten(med.text(), 60)))
```
which will yield (for Glottolog 4.3):
```
1549	grammar sketch	Lily, William. 1549. A Shorte Introduction of Grammar. [...]
1594	grammar sketch	Greaves, Paul. 1594. Grammatica Anglicana, praecipue [...]
1640	grammar sketch	Jonson, Benjamin. 1909 [1640]. The English Grammar [...]
1685	grammar sketch	Cooper, Christopher. 1685. Grammatica Linguae [...]
1891	long grammar	R. Pearse Chope. 1891. The dialect of Hartland, [...]
1954	long grammar	Parker, Mary-Braeme. 1954. A Study of the Speech of [...]
1966	long grammar	Dutton, Tom E. 1966. The informal speech of Palm [...]
1980	long grammar	Shorrocks, Graham. 1980. A grammar of the dialect of [...]
1985	long grammar	Quirk, Randolph and Sidney Greenbaum and Geoffrey [...]
2002	long grammar	Huddleston, Rodney D. and Pullum, Geoffrey K. 2002. [...]
```

Note that Glottolog does not strive for completeness of the historical record, though. I.e. when we already know about a good, recent grammar for
a language, adding all less extensive earlier works to the bibliography is
not a high priority. Thus, the historical record - in particular for well-described languages - will typically be spotty.
