# Examining changes across Glottolog releases

While it is possible, to examine changes just using the `git` version control tool and the "raw" data repository,
this is often overwhelming (see for example the [changes between release 4.2.1 and 4.1](https://github.com/glottolog/glottolog/compare/v4.1...v4.2.1)).
So unless one is interested in particular commits and associated commit comments, which may give some context for particular changes,
it is typically better to examine changes of the aggregated [CLDF dataset](https://github.com/glottolog/glottolog-cldf) 
(e.g. [the changes between 4.2.1 and 4.1](https://github.com/glottolog/glottolog-cldf/compare/v4.1...v4.2.1)).

For some cases, this may still be "too much". In the following we describe how to "condense" changes between two
Glottolog releases down to just a list of changed languoid names. We'll use the tools of the 
[csvkit](https://csvkit.readthedocs.io/en/1.0.2/index.html) package and optionally `git`.

1. Retrieve the table of languoids for the two releases we want to compare
   - either downloading and unpacking [![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.3754594.svg)](https://doi.org/10.5281/zenodo.3754594)
     and [![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.3554966.svg)](https://doi.org/10.5281/zenodo.3554966)
   - or from GitHub [`cldf/languages.csv` at 4.2.1](https://github.com/glottolog/glottolog-cldf/blob/v4.2.1/cldf/languages.csv) and 
     [`cldf/languages.csv` at 4.1](https://github.com/glottolog/glottolog-cldf/blob/v4.1/cldf/languages.csv).
   - or via `git` and a local clone of https://github.com/glottolog/glottolog-cldf
     ```shell script
     $ git show v4.2.1:cldf/languages.csv > languages-4.2.1.csv
     $ git show v4.1:cldf/languages.csv > languages-4.1.csv
     ```
2. Prune the two languoid tables to only the `ID` and `Name` columns using [`csvcut`](https://csvkit.readthedocs.io/en/1.0.2/scripts/csvcut.html):
   ```
   $ csvcut -c ID,Name languages-4.2.1.csv > languages-4.2.1-pruned.csv
   $ csvcut -c ID,Name languages-4.2.1.csv > languages-4.2.1-pruned.csv
   ```
3. Merge the two files into one, with two columns for the names in the two releases using [`csvjoin`](https://csvkit.readthedocs.io/en/1.0.2/scripts/csvjoin.html):
   ```
   $ csvjoin -c ID -q '"' languages-4.2.1-pruned.csv languages-4.1-pruned.csv > combined.csv
   ```
   The merged file looks like this:
   ```
   $ head -n 3 combined.csv 
   ID,Name,Name2
   kond1302,Konda-Yahadian,Konda-Yahadian
   ...
   ```
   i.e. `Name` is the column with names in release 4.2.1, `Name2` for release 4.1 respectively.
4. Narrow the list down to just the languoids with changed names using [`csvsql`](https://csvkit.readthedocs.io/en/1.0.2/scripts/csvsql.html):
   ```
   $ csvsql --query "select id, name, name2 from combined where name != name2 order by id" combined.csv 
   abkh1244,Abkhaz,Abkhazian
   alta1277,Altai-Kizi,Altai Proper
   amur1242,Amur-West Sakhalin Nivkh,Amur
   arti1237,Artial,Artialic
   babi1235,Witsuwit'en-Babine,Babine
   baga1275,Pukur,Baga Mboteni-Binari
   bari1283,Barian,Bari-Kakwa-Mandari
   bayr1238,Badre'i,Bayray
   ...
   ```

And if we are using [bash](https://en.wikipedia.org/wiki/Bash_(Unix_shell)) we can exploit the fact that all
`csvkit` tools are built for [pipes](https://swcarpentry.github.io/shell-novice/04-pipefilter/index.html) and
put all steps in one command:
```
$ csvjoin -c ID -q '"' <(git show v4.2.1:cldf/languages.csv | csvcut -c ID,Name) <(git show v4.1:cldf/languages.csv | csvcut -c ID,Name) | csvsql --query "select id, name, name2 from combined where name != name2 order by id" --tables combined
```
