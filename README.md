# Turkish Morphology

![](https://github.com/google-research/turkish-morphology/workflows/Build%20Status/badge.svg)

A two-level morphological analyzer for Turkish.

This is not an official Google product.

## Components

This implementation is composed of three layers:

* **Lexicons:**

  This layer includes wide-coverage [Turkish lexicons][3] which are manually
  annotated and validated for part-of-speech and morphophonemic irregularities.
  They are intended to be used in building Turkish natural language processing
  tools, such as morphological analyzers. The set of base lexicons that we
  provide includes annotated lexical items for 47,202 words. The tagsets and
  the annotation scheme are described in the [lexicon annotation guidelines][4].

* **Morphotactics:**

  This layer includes [a set of FST definitions][5] which are implemented in a
  custom format which is similar to AT&T FSM format (only difference being that
  we can use strings as state names and input/output labels for each transition
  instead of integers). With each of these FSTs we define the suffixation
  patterns and the morpheme inventories together with their corresponding
  output morphological feature category-value pairs for a given part-of-speech.
  Overall morphotactic model and the morphological feature category-value
  tagsets are described in the [morphotactic model guidelines][6].

* **Morphophonemics:**

  This layer includes [a set of Thrax grammars][7], where each implements a
  standalone morphophonemic process (such as vowel harmony, vowel drop,
  consonant voicing and consonant drop and so on). Composition of the exported
  FSTs defined in these Thrax grammars yield the [morphophonemic model][8] of
  Turkish.

The first level of the morphological analysis is implemented by the
morphophonemic model, which takes a Turkish word and transforms it into the
intermediate representation. The output of the first level is all possible
hypotheses of word stem annotations with morphophonemic irregularities followed
by the meta-morphemes that correspond to the suffixes that are realized in the
surface form.

```
Input: affında
Output: af"+SH+NDA
```

Lexicon entries and morphotactic FST definitions are composed and compiled into
a single FST which acts as the second level of the morphological analysis,
namely the morphotactic model. Morphotactic model takes the intermediate
tape as the input and transforms it to all possible human-readable morphological
analyses that can be generated from the hypotheses generated by the first level.

```
Input: af"+SH+NDA
Output: (af[NN]+[PersonNumber=A3sg]+Hn[Possessive=P2sg]+NDA[Case=Loc])+[Proper=False]
```

See [Interpreting Human-Readable Morphological Analysis][12] section for a
description of such human-readable morphological analysis.

## How to Parse Words

To morphologically parse a word, simply run below from the project root
directory.

```
bazel run -c opt scripts:print_analyses -- --word=[WORD_TO_PARSE]
```

This will morphologically parse the input word against the two-level
morphological analyzer and output a set of human-readable morphological
analysis, as such:

```
bazel run -c opt scripts:print_analyses -- --word=geldiğinde
> Morphological analyses for the word 'geldiğinde':
> (gel[VB]+[Polarity=Pos])([NOMP]-DHk[Derivation=PastNom]+[PersonNumber=A3sg]+Hn[Possessive=P2sg]+NDA[Case=Loc]+[Copula=PresCop]+[PersonNumber=V3pl])+[Proper=False]
> (gel[VB]+[Polarity=Pos])([NOMP]-DHk[Derivation=PastNom]+[PersonNumber=A3sg]+Hn[Possessive=P2sg]+NDA[Case=Loc]+[Copula=PresCop]+[PersonNumber=V3pl])+[Proper=True]
> (gel[VB]+[Polarity=Pos])([NOMP]-DHk[Derivation=PastNom]+[PersonNumber=A3sg]+Hn[Possessive=P2sg]+NDA[Case=Loc]+[Copula=PresCop]+[PersonNumber=V3sg])+[Proper=False]
> (gel[VB]+[Polarity=Pos])([NOMP]-DHk[Derivation=PastNom]+[PersonNumber=A3sg]+Hn[Possessive=P2sg]+NDA[Case=Loc]+[Copula=PresCop]+[PersonNumber=V3sg])+[Proper=True]
> (gel[VB]+[Polarity=Pos])([NOMP]-DHk[Derivation=PastNom]+[PersonNumber=A3sg]+SH[Possessive=P3sg]+NDA[Case=Loc]+[Copula=PresCop]+[PersonNumber=V3pl])+[Proper=False]
> (gel[VB]+[Polarity=Pos])([NOMP]-DHk[Derivation=PastNom]+[PersonNumber=A3sg]+SH[Possessive=P3sg]+NDA[Case=Loc]+[Copula=PresCop]+[PersonNumber=V3pl])+[Proper=True]
> (gel[VB]+[Polarity=Pos])([NOMP]-DHk[Derivation=PastNom]+[PersonNumber=A3sg]+SH[Possessive=P3sg]+NDA[Case=Loc]+[Copula=PresCop]+[PersonNumber=V3sg])+[Proper=False]
> (gel[VB]+[Polarity=Pos])([NOMP]-DHk[Derivation=PastNom]+[PersonNumber=A3sg]+SH[Possessive=P3sg]+NDA[Case=Loc]+[Copula=PresCop]+[PersonNumber=V3sg])+[Proper=True]
> (gel[VB]+[Polarity=Pos])([VN]-DHk[Derivation=PastNom]+[PersonNumber=A3sg]+Hn[Possessive=P2sg]+NDA[Case=Loc])+[Proper=False]
> (gel[VB]+[Polarity=Pos])([VN]-DHk[Derivation=PastNom]+[PersonNumber=A3sg]+Hn[Possessive=P2sg]+NDA[Case=Loc])+[Proper=True]
> (gel[VB]+[Polarity=Pos])([VN]-DHk[Derivation=PastNom]+[PersonNumber=A3sg]+SH[Possessive=P3sg]+NDA[Case=Loc])+[Proper=False]
> (gel[VB]+[Polarity=Pos])([VN]-DHk[Derivation=PastNom]+[PersonNumber=A3sg]+SH[Possessive=P3sg]+NDA[Case=Loc])+[Proper=True]
```

If the input string is not accepted as a Turkish word, morphological analyzer
outputs an empty result.

```
bazel run -c opt scripts:print_analyses -- --word=foo
> 'foo' is not accepted as a Turkish word
```

## Interpreting Human-Readable Morphological Analysis

An example output human-readable morphological analysis is as follows;

**Input Word** (*evlerindekilerin* = those that belongs to ones in their
homes):

```
bazel run -c opt scripts:print_analyses -- --word=evlerindekilerin
```

**Sample Output Morphological Analysis String**:

```
(ev[NN]+[PersonNumber=A3sg]+lArH[Possessive=P3pl]+NDA[Case=Loc])([PRF]-ki[Derivation=Pron]+lAr[PersonNumber=A3sg]+[Possessive=Pnon]+NHn[Case=Gen])+[Proper=False]
```

Human-readable morphological analyses can be decomposed into parts:

* **Inflectional groups:**

  Each human-readable morphological analysis is composed of inflectional groups.
  An inflectional group is a sub-word span, and it is created by affixation of a
  derivational morpheme. Inflectional group analyses are enclosed in
  parenthesis. Above example contains two inflectional groups:
   - `(ev[NN]+[PersonNumber=A3sg]+lArH[Possessive=P3pl]+NDA[Case=Loc])`
   - `([PRF]-ki[Derivation=Pron]+lAr[PersonNumber=A3pl]+[Possessive=Pnon]+NHn[Case=Gen])`

* **Word stem:**

  First inflectional group contains the word stem (e.g. `ev` is the root form
  for the above example input word *evlerindekilerin*).

* **Analysis of morphemes:**

  Within each inflectional group meta-morphemes and
  their corresponding morphological feature category-value tags are separated
  with either `+` or `-` delimiters. (e.g. `+[PersonNumber=A3sg]`,
  `+lArH[Possessive=P3pl]`, `-ki[Derivation=Pron]`, etc.). Strings that are
  immediate followers of the delimiters `+` or `-` are the meta-morphemes (e.g.
  `NDA` is the meta-morpheme in morpheme analysis `+NDA[Case=Loc]`).
  Morphological feature category-value tags are enclosed in brackets right
  after the meta-morphemes (e.g. `Case` is the feature category and `Loc` is
  feature value in morpheme analysis `+NDA[Case=Loc]`).

* **Part-of-speech:**

  Part-of-speech tag of each inflectional group is the first bracketed tag of
  the inflectional group (e.g. `NN` is the part-of-speech of the first
  inflectional group and `PRF` is for the second inflectional group).

* **Inflectional vs. Derivational morphemes:**

  Meta-morphemes that are separated with `+` delimiter do not create a new
  inflectional group. They are inflectional morphemes (e.g.
  `+[PersonNumber=A3sg]`, `+NDA[Case=Loc]`, `+[Possessive=Pnon]`, etc.).
  Meta-morphemes that are separated with `-` delimiter create a new
  inflectional group. They are the derivational morphemes (e.g.
  `-ki[Derivation=Pron]`). Therefore, first meta-morpheme in an inflectional
  group always follows the delimiter `-`, but not `+`.

* **Surface realization of inflections:**

  Some meta-morphemes are not realized in the surface form. These meta-morphemes
  do not correspond to a span of characters in the input word. For them we do
  not output any meta-morpheme in the morpheme analysis (e.g.
  `+lArH[Possessive=P3pl]` and `+NDA[Case=Loc]` are realized in the surface
  form, thus they have explicit meta-morphemes `lArH` and `NDA` in their
  morpheme analysis. However, `+[PersonNumber=A3sg]` and `+[Possessive=Pnon]`
  are not realized in the surface form, therefore only morphological feature
  category-value tags are output for them in their morpheme analysis).

* **Surface realization of derivations:**

  Derivational morphemes must always realize in the surface form. They always
  correspond to a span of characters in the input word. Therefore, we always
  output non-empty meta-morphemes in the corresponding morpheme analysis of
  derivational morphemes. Meaning that no zero-derivations are allowed in the
  morphotactic model.

* **Proper noun analysis:**

  An optional proper noun feature analysis is output at the end of each
  inflectional group (e.g. `+[Proper=False]` which follows the second
  inflectional group). Proper noun feature category can take two values `True`
  or `False`. If it is specified as `True`, the inflectional group that it
  follows is considered to be a part of a proper noun. This feature is used to
  capture the internal structure of proper nouns that are composed of multiple
  words (e.g. for multi-word movie names the true part-of-speech and
  morphological feature of words that compose a multi-word movie name can be
  annotated, while marking the fact that they are part of a proper noun using
  this feature).

  Proper noun feature analysis is omitted for some of the inflectional groups
  to have a compact representation and to minimize the number of morphological
  analyses generated by the morphological analyzer. In such cases, proper noun
  feature analysis of an inflectional group applies to all preceding
  inflectional groups that does not have one (e.g. first inflectional group of
  the above example inherits its proper noun feature analysis `Proper=False`
  from the second inflectional group).

## Python API

We also provide a [Python API][15] that can be used to morphologically analyze
Turkish words, generate Turkish word forms from morphological analyses, parse
human-readable morphological analyses into protobuf messages, validate their
structural well-formedness and to generate human-readable analyses from them.
You can see some example use cases in [`//examples`][17].

If you are using [Bazel][16], you can depend on this repository as an external
dependency of your project by adding the following to your WORKSPACE file:

```
git_repository(
  name = "google_research_turkish_morphology",
  remote = "https://github.com/google-research/turkish-morphology.git",
  tag = "{version-tag}",
)
```

Then, you can simply use
`@google_research_turkish_morphology//turkish_morphology:analyze`
(or other modules of the API) as a dependecy of your relevant `py_library` or
`py_binary` BUILD targets.

The API is also available on PyPi. To install the latest release from PyPi, run:

```
python3 -m pip install turkish-morphology
```

To install from source, run below from the project root directory (preferably
within a Python virtual environment):

```
bazel build //...
bazel-bin/setup install
```

## Requirements

To build and run the morphological analyzer install [Bazel version 3.0.0][9],
[Python 3.7.5 (or a newer version)][10]. All other intrinsic dependencies will
be imported, built and taken care of by Bazel according to the [WORKSPACE][2]
setup throughout the first invocation of the morphological analyzer runtime.
If you are installing from PyPi, you need [pip][14].

## Citing

If you use or discuss the code, data or tools from this repository in your work,
please cite:

Öztürel, A., Kayadelen, T. & Demirşahin, I (2019, September). [A syntactically
expressive morphological analyzer for Turkish][13]. In Proceedings of the 14th
International Conference on Finite-State Methods and Natural Language
Processing (pp. 65-75).

```
@inproceedings{
    title = "A Syntactically Expressive Morphological Analyzer for Turkish",
    author = "\"{O}zt\"{u}rel, Adnan and Kayadelen, Tolga and Demir\c{s}ahin,
        I\c{s}{\i}n",
    booktitle = "Proceedings of the 14th International Conference on Finite-State
        Methods and Natural Language Processing",
    month = "23--25" # sep,
    year = "2019",
    address = "Dresden, Germany",
    publisher = "Association for Computational Linguistics",
    url = "https://www.aclweb.org/anthology/W19-3110",
    pages = "65--75",
}
```

## License

Unless otherwise noted, all original files are licensed under an
[Apache License, Version 2.0][1].

[1]: LICENSE
[2]: WORKSPACE
[3]: ./src/analyzer/lexicon/base
[4]: ./src/analyzer/lexicon/README.md
[5]: ./src/analyzer/morphotactics/model
[6]: ./src/analyzer/morphotactics/README.md
[7]: ./src/analyzer/morphophonemics
[8]: ./src/analyzer/morphophonemics/model.grm
[9]: https://docs.bazel.build/versions/master/install.html
[10]: https://www.python.org/downloads/
[12]: #interpreting-human-readable-morphological-analysis
[13]: https://www.aclweb.org/anthology/W19-3110
[14]: https://pip.pypa.io/en/stable/installing/
[15]: ./turkish_morphology
[16]: https://bazel.build/
[17]: ./examples
