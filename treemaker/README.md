## Extracting a tree for a given set of languoids from the global tree

Often it is useful to extract a tree from Glottolog's global tree (i.e. the rake of all family trees) with only
a given set of languoids as leafs. This can easily be done using the Glottolog API and the
[`treemaker` package](https://pypi.python.org/pypi/treemaker).

The script [`glottolog_tree.py`](glottolog_tree.py) does exactly this: Given a set of Glottolog languoids, specified by name, 
ISO-639-3 code or Glottocode, it constructs the corresponding tree which can then be printed in various formats to the terminal
(or piped into a file):

- Nexus:
```
$ python glottolog_tree.py --format=nexus deu eng Welsh Pali scot1243 
#NEXUS

begin trees;
   tree root = (Welsh,(deu,(eng,scot1243)),Pali);
end;
```

- Newick:
```
$ python glottolog_tree.py --format=newick deu eng Welsh Pali scot1243
(Welsh,(deu,(eng,scot1243)),Pali)
```

- ASCII art:
```
$ python glottolog_tree.py deu eng Welsh Pali scot1243
           ┌─Welsh
           │          ┌─deu
           ├──────────┤
───────────┤          │          ┌─eng
           │          └──────────┤
           │                     └─scot1243
           └─Pali
```

Alternatively, you can just use `pyglottolog` as follows (after looking up Glottocodes for your languages):

```python
>>> from pyglottolog import Glottolog
>>> api = Glottolog('.')
>>> indo1319 = api.languoid('indo1319')
>>> tree = indo1319.newick_node(template='{l.id}', maxlevel=api.languoid_levels.language)
>>> tree.prune_by_names(['pali1273', 'wels1247', 'stan1295', 'stan1293', 'scot1243'], inverse=True)
>>> for node in tree.walk():
...     if node.name:
...             node.name = api.languoid(node.name).name
... 
>>> tree.remove_redundant_nodes()
>>> print(tree.ascii_art())
                ┌─Welsh
                │               ┌─German
                ├─West Germanic─┤
──Indo-European─┤               │               ┌─Scots
                │               └─Later Anglic──┤
                │                               └─English
                └─Pali
```
