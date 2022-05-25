# Investigating the documentation status for languages of a subgroup

Glottolog's [GlottoScope](https://glottolog.org/langdoc/status) does not allow filtering for language subgroups (as opposed to top-level families)
like [Chadic](https://glottolog.org/resource/languoid/id/chad1250). Fortunately, the relevant data is available in `glottolog-cldf`. So how can we
go about this?

Of course we'll reach for [csvkit](https://csvkit.readthedocs.io/en/latest/) for this task, but it should become clear how to do this on any other data analysis
platform, e.g. R or Excel as well.

First, we'll get a one-column table listing all Glottocodes of members of the Chadic subgroup. The relevant data are the values for the 
CLDF parameter `classification` - the list of ancestor glottocodes of a languoid. Any languoid listing `chad1250` among its ancestors belongs to Chadic:

```shell
$ csvgrep -c Parameter_ID -r"^classification" cldf/values.csv | csvgrep -c Value -m "chad1250" | csvcut -c Language_ID > chadic.csv
```

Next, we'll filter the ValueTable to just values for the `med` parameter, specifying the source with the "**m**ost **e**xtensive **d**escription" for a 
language:

```shell
$ csvgrep -c Parameter_ID -r"^med$" cldf/values.csv > meds.csv
```

Values here are the following documentation levels:
```shell
$ csvgrep -c Parameter_ID -r"^med$" cldf/codes.csv 
ID,Parameter_ID,Name,Description
med-long_grammar,med,long grammar,Grammar with more than 300 pages
med-grammar,med,grammar,Grammar with less than 300 pages
med-grammar_sketch,med,grammar sketch,Grammar sketch
med-phonology_or_text,med,phonology/text,"New Testament, Text, Phonology, (typological) Study Of A Specific Feature or Dictionary"
med-wordlist_or_less,med,Wordlist or less,Wordlist or less
```

Now we join the list of Chadic Glottocodes with the table of MEDs, thereby reducing the latter to just values for Chadic languages:
```shell
$ csvjoin -c Language_ID chadic.csv meds.csv
```

We can get some summary statistics right away:
```shell
$ csvjoin -c Language_ID chadic.csv meds.csv | csvstat
  1. "Language_ID"

	Type of data:          Text
	Contains null values:  False
	Unique values:         206
	Longest value:         8 characters
	Most common values:    suku1272 (1x)
	                       mina1276 (1x)
	                       mbed1242 (1x)
	                       gava1241 (1x)
	                       buwa1243 (1x)

  2. "ID"

	Type of data:          Text
	Contains null values:  False
	Unique values:         206
	Longest value:         12 characters
	Most common values:    suku1272-med (1x)
	                       mina1276-med (1x)
	                       mbed1242-med (1x)
	                       gava1241-med (1x)
	                       buwa1243-med (1x)

  4. "Value"

	Type of data:          Number
	Contains null values:  False
	Unique values:         5
	Smallest value:        0
	Largest value:         4
	Sum:                   517
	Mean:                  2,51
	Median:                3
	StDev:                 1,457
	Most common values:    4 (77x)
	                       2 (47x)
	                       0 (33x)
	                       3 (33x)
	                       1 (16x)

  5. "Code_ID"

	Type of data:          Text
	Contains null values:  False
	Unique values:         5
	Longest value:         21 characters
	Most common values:    med-wordlist_or_less (77x)
	                       med-grammar_sketch (47x)
	                       med-long_grammar (33x)
	                       med-phonology_or_text (33x)
	                       med-grammar (16x)

  7. "Source"

	Type of data:          Text
	Contains null values:  False
	Unique values:         169
	Longest value:         41 characters
	Most common values:    hh:hvw:JungraithmayrIbriszimow:CLR (11x)
	                       hh:hw:Kraft:Chadic:II (5x)
	                       hh:hs:Schuh:Bole-Tangale (5x)
	                       hh:hw:Kraft:Chadic:III (4x)
	                       hh:w:Brye:Jimjimen-Gude-Tsuvan-Sharwa (3x)

Row count: 206
```

