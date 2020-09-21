
## Listing recent grammars

While the Glottolog CLDF dataset does **not** contain all of Glottolog's
reference data (to keep it of reasonable size), we can still investigate
all grammars which are considered [MED (i.e. **M**ost **E**xtensive **D**escription)](http://hdl.handle.net/10125/24792) for a language.

Since this kind of query requires joining information for three different entities
(namely languages, references and parameters), it is best done [using SQLite](https://github.com/glottolog/cookbook/blob/master/recipes/glottolog_cldf/README.md#using-sqlite-to-access-the-cldf-data).

To piece together our query, we first take a peek at the relevant tables:

### Parameters

The Glottolog CLDF dataset provides data for the following parameters:

```sql
sqlite> .headers on
sqlite> select * from ParameterTable;
cldf_id|cldf_name|cldf_description|type
level|Level||categorical
category|Category||categorical
classification|Classification||
subclassification|Subclassification||
aes|Agglomerated Endangerment Status||sequential
med|Most Extensive Description||sequential
```

What we are interested in is the "Most Extensive Description" for each language. Using the `cldf_id` as foreign key value we can lookup the codes that
are defined for this parameter:

```sql
sqlite> select * from CodeTable where cldf_parameterreference = 'med';
cldf_id|cldf_parameterReference|cldf_name|cldf_description|numerical_value
med-long_grammar|med|long grammar|Grammar with more than 300 pages|0
med-grammar|med|grammar|Grammar with less than 300 pages|1
med-grammar_sketch|med|grammar sketch|Grammar sketch|2
med-phonology_or_text|med|phonology/text|New Testament, Text, Phonology, (typological) Study Of A Specific Feature or Dictionary|3
med-wordlist_or_less|med|Wordlist or less|Wordlist or less|4
```

### Languages

```sql
sqlite> .schema LanguageTable
CREATE TABLE `LanguageTable` (
    `cldf_id` TEXT NOT NULL,
    `cldf_name` TEXT,
    `cldf_macroarea` TEXT,
    `cldf_latitude` REAL CHECK (`cldf_latitude` >= -90 AND `cldf_latitude` <= 90),
    `cldf_longitude` REAL CHECK (`cldf_longitude` >= -180 AND `cldf_longitude` <= 180),
    `cldf_glottocode` TEXT,
    `cldf_iso639P3code` TEXT,
    `Countries` TEXT,
    `Family_ID` TEXT,
    `cldf_languageReference` TEXT,
    PRIMARY KEY(`cldf_id`),
    FOREIGN KEY(`Family_ID`) REFERENCES `LanguageTable`(`cldf_id`) ON DELETE CASCADE,
    FOREIGN KEY(`cldf_languageReference`) REFERENCES `LanguageTable`(`cldf_id`) ON DELETE CASCADE
);
```



### References

```sql
sqlite> .schema SourceTable
CREATE TABLE `SourceTable` (
    `id` TEXT,
...
    `author` TEXT,
...
    `title` TEXT,
...
    `year` TEXT,
...
    PRIMARY KEY(`id`)
);
```


### Values

These three entities are tied together in a [CLDF StructureDataset](https://github.com/cldf/cldf/tree/master/modules/StructureDataset) through
the `ValueTable`:

```sql
sqlite> .schema valuetable
CREATE TABLE `ValueTable` (
    `cldf_id` TEXT NOT NULL,
    `cldf_languageReference` TEXT NOT NULL,
    `cldf_parameterReference` TEXT NOT NULL,
    `cldf_value` TEXT,
    `cldf_codeReference` TEXT,
    `cldf_comment` TEXT,
    `codeReference` TEXT,
    PRIMARY KEY(`cldf_id`),
    FOREIGN KEY(`cldf_parameterReference`) REFERENCES `ParameterTable`(`cldf_id`) ON DELETE CASCADE,
    FOREIGN KEY(`cldf_codeReference`) REFERENCES `CodeTable`(`cldf_id`) ON DELETE CASCADE,
    FOREIGN KEY(`cldf_languageReference`) REFERENCES `LanguageTable`(`cldf_id`) ON DELETE CASCADE
);
```

and a many-to-many relation between values and sources:
```sql
sqlite> .schema valuetable_sourcetable
CREATE TABLE `ValueTable_SourceTable` (
    `ValueTable_cldf_id` TEXT,
    `SourceTable_id` TEXT,
    `context` TEXT,
    FOREIGN KEY(`ValueTable_cldf_id`) REFERENCES `ValueTable`(`cldf_id`) ON DELETE CASCADE,
    FOREIGN KEY(`SourceTable_id`) REFERENCES `SourceTable`(`id`) ON DELETE CASCADE
);
```

Putting it all together, we can query
```sql
select
    l.cldf_name, l.cldf_glottocode, l.cldf_iso639P3code, l.cldf_latitude, l.cldf_longitude,
    v.cldf_value,
    s.author, s.title, cast(s.year as int) as year, s.id 
from 
    valuetable as v, 
    valuetable_sourcetable as vs, 
    sourcetable as s,
    languagetable as l
where
    v.cldf_id = vs.valuetable_cldf_id and 
    vs.sourcetable_id = s.id and 
    v.cldf_parameterreference = 'med' and 
    v.cldf_languagereference = l.cldf_id and
    (v.cldf_value = 'long grammar' or v.cldf_value = 'grammar') and
    cast(s.year as int) > 2015
order by year desc;
```

and get results looking like this

cldf_name|cldf_glottocode|cldf_iso639P3code|cldf_latitude|cldf_longitude|cldf_value|author|title|year|id
--- | --- | --- | ---| ---| --- | --- | --- | --- | ---
Kuy|kuyy1240|kdt|14.6698|104.911|long grammar|Chareanraat, Chirat|The Kuay language of Suphanburi|2525|hh:g:Chareanraat:Kuay
Patwin|patw1250|pwi|39.2143|-122.0094|long grammar|Lawyer, Lewis C.|A Grammar of Patwin|2021|hh:g:Lawyer:Patwin
Min Nan Chinese|minn1241|nan|24.5|118.17|long grammar|Chen, Weirong|A Grammar of Southern Min: The Hui’an Dialect|2020|hh:g:Chen:Huian:2020
Seeku|seek1238|sos|11.1285|-4.58203|long grammar|McPherson, Laura|A grammar of Seenku|2020|hh:g:McPherson:Seenku

You can use the reference ID (from the last column) to lookup details on the web at `https://glottolog.org/resource/reference/id/<ID>`, e.g. https://glottolog.org/resource/reference/id/hh:g:Chareanraat:Kuay
and as always with SQLite, you can output the results to [a CSV file](recent_grammars.csv) via
```sqlite
sqlite> .headers on
sqlite> .mode csv
sqlite> .output recent_grammars.csv
sqlite> select
   ...>     l.cldf_name, l.cldf_glottocode, l.cldf_iso639P3code, l.cldf_latitude, l.cldf_longitude,
   ...>     v.cldf_value,
   ...>     s.author, s.title, cast(s.year as int) as year, s.id 
   ...> from 
   ...>     valuetable as v, 
   ...>     valuetable_sourcetable as vs, 
   ...>     sourcetable as s,
   ...>     languagetable as l
   ...> where
   ...>     v.cldf_id = vs.valuetable_cldf_id and 
   ...>     vs.sourcetable_id = s.id and 
   ...>     v.cldf_parameterreference = 'med' and 
   ...>     v.cldf_languagereference = l.cldf_id and
   ...>     (v.cldf_value = 'long grammar' or v.cldf_value = 'grammar') and
   ...>     cast(s.year as int) > 2015
   ...> order by year desc;
```