# wordkit

This is the repository of the `wordkit` package, a Python 3.X package for the featurization of words into orthographic and phonological vectors.

A paper that describes `wordkit` was accepted at LREC 2018.
If you use `wordkit` in your research, please cite the following paper::

```
bibtexref
```

The subpackages are extensively documented using separate `README.MD` files.
Feel free to click ahead to find descriptions of the contents of subpackages.

Additionally, if you use any of the corpus readers in `wordkit`, you MUST cite the accompanying corpora.
All of these references can be found in the docstrings of the applicable classes.

`wordkit` makes heavy use of [ipapy](https://github.com/pettarin/ipapy>), a package which parses unicode IPA characters into their IPA representations.


## Overview

`wordkit` is a modular system, and contains 5 broad families of components.

* Readers
* Feature extractors
* Feature sets
* Transformers
* Samplers

In general, a `wordkit` pipeline consists of one or more readers, which extract structured information from corpora.
This information is then sent to one or more transformers, which are either assigned pre-defined features or a feature extractor.
If a feature extractor is assigned to a transformer, the feature set of that transformer is extracted automatically.

Finally, a sampler can be used to sample from the resulting items.

## Experiments

The code for replicating the experiments in the `wordkit` paper can be found [here](https://github.com/stephantul/lrec2018)

## Requirements

- sklearn
- ipapy
- numpy

## Example

```python
from wordkit.readers import Celex
from wordkit.transformers import LinearTransformer, WickelTransformer
from wordkit.features import fourteen
from sklearn.pipeline import FeatureUnion

# The fields we want to extract from our corpora.
fields = ('orthography', 'frequency', 'phonology', 'syllables')

# Filter function
# A filter function can be added to a corpus to filter out any
# words which do not conform to some criterion, e.g. frequency
# constraints.
# In this case, we use it to remove punctuation marks for which
# we do not have any features.
# We also use it to select only monosyllables.
fil = lambda x: not set(x['orthography']).intersection({"'", ',', '-', '/', '.'}) and len(x['syllables']) == 1

# We set merge duplicates to True because some of
# the distinctions in Celex (e.g. POS) do not matter to us.

# Link to epl.cd
english = Celex("epl.cd",
              fields=fields,
              merge_duplicates=True)

# Link to dpl.cd
dutch = Celex("dpl.cd",
            fields=fields,
            language='nld',
            merge_duplicates=True)

words = english.transform(filter_function=fil)
words.extend(dutch.transform(filter_function=fil))

# words[0] =>
# {'frequency': 1267035,
# 'orthography': 'a',
# 'phonology': ('e', 'ɪ'),
# 'syllables': (('e', 'ɪ'),)}

# You can also query specific words
wind = corpora.transform(["wind"])

# This gives
# wind =>
#[{'orthography': 'wind', 'syllables': (('w', 'a', 'ɪ', 'n', 'd'),), 'phonology': ('w', 'a', 'ɪ', 'n', 'd'), 'frequency': 298},
# {'orthography': 'wind', 'syllables': (('w', 'ɪ', 'n', 'd'),), 'phonology': ('w', 'ɪ', 'n', 'd'), 'frequency': 2170},
# {'orthography': 'wind', 'syllables': (('w', 'ɪ', 'n', 't'),), 'phonology': ('w', 'ɪ', 'n', 't'), 'frequency': 4702}],

# Now, let's transform into features
# Orthography is a linear transformer with the fourteen segment feature set.
o = LinearTransformer(fourteen, field='orthography')
# For phonology we use Wickelphones.
p = WickelTransformer(n=3, field='phonology')

featurizer = FeatureUnion([("o", o), ("p", p)])

# Fit and transform the featurizers.
X = featurizer.fit_transform(words)
# A (7650, 4795) matrix.

# Get the feature vector length for each featurizer
o.vec_len # 112
p.vec_len # 4683

# Inspect the features of the Wickeltransformer
p.features
```

## Contributors

Stéphan Tulkens

## License

GPL v3