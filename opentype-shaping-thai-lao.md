# Thai and Lao shaping in OpenType #

This document details the shaping procedure needed to display text
runs in the Thai and Lao scripts.


**Table of Contents**

  - [General information](#general-information)
  - [Terminology](#terminology)
  - [Glyph classification](#glyph-classification)
      - [Shaping classes and subclasses](#shaping-classes-and-subclasses)
      - [Mark combining classes](#mark-combining-classes)
      - [PUA fallback classifications](#pua-fallback-classifications)
      - [Thai and Lao character tables](#thai-and-lao-character-tables)
  - [The `<thai>`-`<lao >` shaping model](#the-thai-lao-shaping-model)
      - [1: Applying the language substitution features from GSUB](#1-applying-the-language-substitution-features-from-gsub)
      - [2: Decomposing all Am vowel signs](#2-decomposing-all-am-vowel-signs)
      - [3: Reordering sequences of marks](#2-reordering-sequences-of-marks)
      - [3: Applying all positioning features from GPOS](#3-applying-all-positioning-features-from-gpos)
  - [The PUA fallback shaping model](#the-pua-fallback-shaping-model)
      - [1: Applying the language substitution features from GSUB](#1-applying-the-language-substitution-features-from-gsub)
      - [2: Applying all basic substitution features from GSUB](#2-applying-all-basic-substitution-features-from-gsub)
      - [3: Applying remaining positioning features from GPOS](#3-applying-remaining-positioning-features-from-gpos)



## General information ##

The Thai and Lao scripts are both descendants of the Brahmi script,
and follow many of the same general patterns found in [Indic
scripts](opentype-shaping-indic-general.md). They are distinct enough
from Indic scripts that they should not be supported by a
general-purpose Indic shaping engine.

Thai and Lao use different alphabets but are historically
related. They share common orthographic conventions and shaping
characteristics, which enables shaping engines to support both scripts
in a single implementation.

The Thai script is used to write multiple languages, most commonly
Thai, Pak Thai (or Southern Thai), Kuy, Isan, Lanna (or Northern
Thai), and Kelantan-Pattani Malay. In addition, the Thai script is
used to write Sanskrit and Pali. However, the Thai script is not used
for Vedic texts, therefore Thai And Lao text runs are not expected to
include any glyphs from the Vedic Extension block of Unicode.

The Lao script is used to write multiple languages, most commonly
Lao, Khmu', Hmong, and Isan. 

The Thai script tag defined in OpenType is `<thai>`. The Lao script
tag defined in OpenType is `<lao >`. Because OpenType script tags must
be exactly four letters long, the `<lao >` tag includes a trailing
space. 

A significant number of  older Thai fonts that do not use the OpenType
shaping model are still in usage; these fonts employ the Unicode
"Private Use Area" (`PUA`) to store contextual forms of
characters. Shaping engines may implement this PUA-base shaping model
as a fallback mechanism when such fonts are encountered.


## Terminology ##

OpenType shaping uses a standard set of terms for Brahmi-derived and
Indic scripts.  The terms used colloquially in any particular language
may vary, however, potentially causing confusion.

Both Thai and Lao feature inherent vowels for every consonant, and
employ **dependent vowel** signs to replace the inherent vowel with a
different vowel sound.

The Thai term for a dependent vowel sign is **sara**. The Lao term for
a vowel sign is **sala**. The official names of the Thai vowel signs
in the Unicode standard includes "sara" (for example, "Sara Am"),
while the official names of the Lao vowel signs uses "sign" (for
example, "Sign Am").

Some of these dependent-vowel signs are encoded as marks that attach
to the consonant in **above-base** or **below-base** position. Others
are encoded as letters that may appear in **pre-base** (left-side) or
**post-base** (right-side) position.

Thai and Lao differ from Indic scripts in that these pre-base dependent
vowels are entered before typing the consonant to which they
apply. Therefore, pre-base dependent vowels do not need to be
reordered by the shaping engine.

**Phinthu** is the term used for the Thai equivalent of the "halant"
or "virama" mark that supresses the inherent vowel of a consonant. It
is used only when writing Sanskrit or Pali text in the Thai script.

**Nikhahit** is the term for the Thai equivalent of "anusvara". It
is used only when writing Sanskrit or Pali text in the Thai
script. The equivalent mark in Lao is called **niggahita**.

Both Thai and Lao include several **tone markers** as combining marks
that are positioned with respect to the consonant and, possibly, to
any corresponding dependent-vowel marks.

Where possible, using the standard terminology is preferred, as the
use of a language-specific term necessitates choosing one language
over all of the others that share a common script.


## Glyph classification ##

Shaping Thai and Lao text depends on the shaping engine correctly
classifying each glyph in the run. As with most other scripts, the
classifications must distinguish between consonants, vowels
(independent and dependent), numerals, punctuation, and various types
of diacritical mark. 

For most codepoints, the `General Category` property defined in the Unicode
standard is correct, but it is not always sufficient to fully capture the
expected shaping behavior. Therefore, Thai and Lao glyphs may
additionally be classified by how they are treated when shaping a run
of text.

### Shaping classes and subclasses ###

The shaping classes listed in the tables that follow are defined so
that they capture the positioning rules used by Thai and Lao scripts. 

For most codepoints, the _Shaping class_ is synonymous with the `Indic
Syllabic Category` defined in Unicode. However, there are some
distinctions, where the defined category does not fully capture the
behavior of the character in the shaping process.

Numbers are classified as `NUMBER`, even though they evoke no special
behavior from the Indic shaping rules, because there are OpenType features that
might affect how the respective glyphs are drawn, such as `tnum`,
which specifies the usage of tabular-width numerals, and `sups`, which
replaces the default glyphs with superscript variants.

Marks, including diacritics, tone markers, and dependent vowels, are further labeled
with a mark-placement subclass, which indicates where the glyph will
be placed with respect to the base character to which it is
attached. The actual position of the glyphs is determined by the
lookups found in the font's GPOS table.

There are three basic _mark-placement subclasses_ for marks
in Thai and Lao. Each corresponds to the visual position of the mark with
respect to the consonant to which it is attached:

  - `TOP_POSITION` marks are positioned above the consonant.
  - `BOTTOM_POSITION` marks are positioned below the consonant.
  - `RIGHT_POSITION` marks are positioned to the right of the consonant.
  
Thai and Lao vowel marks can also appear to the left of the consonant
to which they are attached. However, in Thai and Lao text runs, these
vowels exist _before_ the consonant — that is, to the left of the
consonant in the character sequence. Thus, no reordering of these
vowels (as is done in several other Brahmi-derived scripts) is
required for Thai or Lao.

In order to unambiguously distinguish between this non-reordering
convention and the reordering conventions of other scripts, the
left-side vowels are not designated `LEFT_POSITION` in their
mark-placement subclass. Instead, these vowels are classified as `VISUAL_ORDER_LEFT`.

These positions may also be referred to elsewhere in shaping documents as:

  - _Above-base_ 
  - _Below-base_ 
  - _Pre-base_ 
  - _Post-base_ 
  
respectively. The `VISUAL_ORDER_LEFT`, `RIGHT`, `TOP`, and `BOTTOM` designations
corresponds to Unicode's preferred terminology. The _Pre_, _Post_,
_Above_, and _Below_ terminology is used in the official descriptions
of OpenType GSUB and GPOS features. Shaping engines may, internally,
use whichever terminology is preferred.

For most mark and dependent-vowel codepoints, the _mark-placement
subclass_ is synonymous with the `Indic Positional Category` defined
in Unicode. However, there may be some distinctions, where the defined
category does not fully capture the behavior of the character in the
shaping process. 


### Mark combining classes ###

The Unicode standard defines a _canonical combining class_ for each mark
codepoint that is used whenever a sequence of marks needs to be sorted
into canonical order. 

The Thai and Lao marks belong to standard combining classes, for example:

| Codepoint | Combining class | Glyph                              |
|:----------|:----------------|:-----------------------------------|
|`U+0E38`   | 103             | &#x0E38; Sara U                    |
|`U+0E47`   | 0               | &#x0E47; Maitaikhu                 |
|`U+0E4A`   | 107             | &#xOE4A; Mai Tri                   |
|`U+0EB9`   | 118             | &#x0EB9; Sign Uu                   |
|`U+0EBC`   | 0               | &#x0E47; Semivowel Sign Lo         |
|`U+0ECB`   | 122             | &#xOE4A; Tone Mai Catawa           |


The numeric values of these combining classes are used during Unicode
normalization.


### PUA fallback classifications ###

Older Thai fonts that implement the PUA-substitution fallback method
rather than modern OpenType script shaping rules incorporate
subclasses for consonants that indicate whether or not the consonant
includes an ascender, a normal descender, or a removable descender.

There are four possible values:

  - `NORMAL_CONSONANT` or `NC`
  - `ASCENDER_CONSONANT` or `AC`
  - `DESCENDER_CONSONANT` or `DC`
  - `REMOVABLE_DESCENDER_CONSONANT` or `RC`
  
Furthermore, vowels and marks in these fonts are classified by whether
they are positioned at the same baseline as consonants, below
consonants, above consonants, or must be positioned at the top of any
stacks of marks.

There are four possible values:

  - `CONSONANT_BASELINE_LEVEL` or `CV`
  - `BELOW_CONSONANT_LEVEL` or `BV`
  - `ABOVE_CONSONANT_LEVEL` or `AV`
  - `TOP_LEVEL` or `TV`

### Thai and Lao character tables ###

Separate character tables are provided for the Thai and Lao blocks as
well as for other miscellaneous characters that are used in `<thai>`
and `<lao >` text runs: 

  - [Thai character table](character-tables/character-tables-thai.md#thai-character-table)
  - [Miscellaneous character table](character-tables/character-tables-thai.md#miscellaneous-character-table)

  - [Lao character table](character-tables/character-tables-lao.md#lao-character-table)
  - [Miscellaneous character table](character-tables/character-tables-lao.md#miscellaneous-character-table)

The tables list each codepoint along with its Unicode general
category, its shaping class, its mark-placement subclass, and its
PUA-fallback category. The codepoint's Unicode name and an example
glyph are also provided.

For example:

| Codepoint | Unicode category | Shaping class     | Mark-placement subclass | PUA    | Glyph                         |
|:----------|:-----------------|:------------------|:------------------------|:-------|:------------------------------|
|`U+0E01`   | Letter           | CONSONANT         | _null_                  | NC     | &#x0E01; Ko Kai               |
| | | | | | |
|`U+0E48`   | Mark [Mn]        | TONE_MARKER       | TOP_POSITION            | TV     | &#x0E48; Mai Ek               |
| | | | | | |
|`U+0E81`   | Letter           | CONSONANT         | _null_                  | _null_ | &#x0E81; Ko                   |
| | | | | | |
|`U+0EC8`   | Mark [Mn]        | TONE_MARKER       | TOP_POSITION            | _null_ | &#x0EC8; Tone Mai Ek          |



Codepoints with no assigned meaning are
designated as _unassigned_ in the _Unicode category_ column. 

Assigned codepoints with a _null_ in the _Shaping class_
column evoke no special behavior from the shaping engine.

The _Mark-placement subclass_ column indicates mark-placement
positioning for codepoints in the _Mark_ category. Assigned, non-mark
codepoints have a _null_ in this column and evoke no special
mark-placement behavior. Marks tagged with [Mn] in the _Unicode
category_ column are categorized as non-spacing; marks tagged with
[Mc] are categorized as spacing-combining.

The _PUA_ column indicates which, if any, fallback-shaping category
the codepoint belongs to when found in older fonts using the PUA
fallback shaping scheme. Note that the PUA method was employed only
for Thai fonts, so Lao codepoints do not have a PUA fallback-shaping
category. Thai codepoints with a _null_ in the _PUA_ column were not
used in the PUA fallback-shaping scheme and evoke no special behavior
from the shaping engine.

Some codepoints in the tables use a _Shaping class_ that
differs from the codepoint's Unicode _General Category_. The _Shaping
class_ takes precedence during OpenType shaping, as it captures more
specific, script-aware behavior.

Other important characters that may be encountered when shaping runs
of Thai and Lao text include the dotted-circle placeholder (`U+25CC`), 
the no-break space (`U+00A0`), and the zero-width space (`U+200B`).

The dotted-circle placeholder is frequently used when displaying a
dependent vowel sign or a combining mark in isolation. Real-world
text syllables may also use other characters, such as hyphens or dashes,
in a similar placeholder fashion; shaping engines should cope with
this situation gracefully.

<!--- The zero-width joiner is primarily used to prevent the formation of a
subjoining form from a "_Consonant_,Halant,_Consonant_" sequence. The sequence
"_Consonant_,Halant,ZWJ,_Consonant_" blocks the substitution of a
subjoined form for the second consonant. --->

<!---
A secondary usage of the zero-width joiner is to prevent the formation of
"Reph". An initial "Ra,Halant,ZWJ" sequence should not produce a "Reph",
where an initial "Ra,Halant" sequence without the zero-width joiner
otherwise would.
--->

The no-break space is primarily used to insert spaces between
phrases. Thai and Lao texts do not employ inter-word spaces. Consequently,
when spaces are inserted into a text run, it is important that they be
preserved: line-breaking algorithms must not break lines after a
Thai or Lao space, so the no-break space character is used instead of the
traditional space. 

The no-break space may also be used to display those codepoints that
are defined as non-spacing (marks, dependent vowels (matras),
below-base consonant forms, and post-base consonant forms) in an
isolated context, as an alternative to displaying them superimposed on
the dotted-circle placeholder. 

## The `<thai>`-`<lao >` shaping model ##

Processing a run of `<thai>` or `<lao >` text involves four top-level stages:


1. Applying the language substitution features from GSUB
2. Decomposing all Am vowel signs
3. Reordering sequences of marks
4. Applying all positioning features from GPOS


As with other Brahmi-derived and Indic scripts, the basic substitution
features must be applied to the run in a specific order. The
positioning features in the final stage, however, do not have a
mandatory order.

Unlike many other Brahmi-derived and Indic scripts, shaping Thai and Lao
text does not require a syllable-identification stage.

Each syllable contains exactly one vowel sound. Valid syllables may
begin with either a consonant or an independent vowel. 

In addition to valid syllables, stand-alone sequences may occur, such
as when an isolated codepoint is shown in example text.


### 1: Applying the language substitution features from GSUB ###

The language-substitution stage applies mandatory substitution features
using the rules in the font's GSUB table. In preparation for this
stage, glyph sequences should be tagged for possible application 
of GSUB features.

The order in which these substitutions must be performed is fixed:

	locl
	ccmp


#### 1.1 locl ####

The `locl` feature replaces default glyphs with any language-specific
variants, based on examining the language setting of the text run.

> Note: Strictly speaking, the use of localized-form substitutions is
> not part of the shaping process, but of the localization process,
> and could take place at an earlier point while handling the text
> run. However, shaping engines are expected to complete the
> application of the `locl` feature before applying the subsequent
> GSUB substitutions in the following steps.

![Local-forms substitution](images/thai/thai-locl.png)


#### 1.2: ccmp ####

The `ccmp` feature allows a font to substitute mark-and-base sequences
with a pre-composed glyph including the mark and the base, or to
substitute a single glyph into an equivalent decomposed sequence of glyphs. 
 
In `<thai>` and `<lao >` text, this may include a decomposition for
the "Am" dependent-vowel sign. If such a decomposition is used in the
active font, the shaping engine must keep track of the fact that the
resulting components originated as an "Am" sign. 

If there is not an "Am" decomposition in the active font's `ccmp`
lookup, the shaping engine will decompose the codepoint in the
following stage.
  
If present, these composition and decomposition substitutions must be
performed before applying any other GSUB lookups, because
those lookups may be written to match only the `ccmp`-substituted
glyphs. 


### 2. Decomposing all Am vowel signs ###

The Thai and Lao alphabets each include one character that must be
decomposed for shaping purposes, the vowel sign "Am". The decomposition is
canonically defined, resulting in the sequence "_Anusvara_,Sara Aa" in
the appropriate script. 

  - Thai Sara Am (`U+0E33`) decomposes to "Nikhahit,Sara Aa" (`U+0E4D`,`U+0E32`).
  - Lao Sign Am (`U+0EB3`) decomposes to "Niggahita,Sign Aa" (`U+0ECD`,`U+0EB2`).

> Note: if the active font decomposed the "Am" sign via a `ccmp`
> feature lookup during stage one, then no further action is needed
> on the shaping engine's part during this stage.

The shaping engine must keep track of the fact that the "Nikhahit" or
"Niggahita" marks originated as part of an "Am" sign, because these
decomposed marks are handled differently during the mark-reordering
stage.

  
### 3. Reordering sequences of marks ###

A "Nikhahit" or "Niggahita"mark that originated as part of an "Am" sign

<!--- 

move the
   * NIKHAHIT backwards over any tone mark (0E48-0E4B).
   *
   * <0E14, 0E4B, 0E33> -> <0E14, 0E4D, 0E4B, 0E32>
   *
   * This reordering is legit only when the NIKHAHIT comes from a SARA AM, not
   * when it's there to start with. The string <0E14, 0E4B, 0E4D> is probably
   * not what a user wanted, but the rendering is nevertheless nikhahit above
   * chattawa.
   *
   * Same for Lao.
   *
   * Note:
   *
   * Uniscribe also does some below-marks reordering.  Namely, it positions U+0E3A
   * after U+0E38 and U+0E39.  We do that by modifying the ccc for U+0E3A.
   * See unicode->modified_combining_class ().  Lao does NOT have a U+0E3A
   * equivalent.

--->

### 4: Applying all positioning features from GPOS ###

In this stage, mark positioning, kerning, and other GPOS features are
applied. As with the preceding stage, the order in which these
features are applied is not canonical; they should be applied in the
order in which they appear in the GPOS table in the font.

        kern
		mark
		mkmk

> Note: The `kern` feature is usually applied at this stage, if it is
> present in the font. However, `kern` is not mandatory for shaping
> Thai and Lao text and may be disabled by user preference.

The `kern` feature adjusts the horizontal positioning of
glyphs.

![Application of the kern feature](/images/thai/thai-kern.png)


The `mkmk` feature positions marks with respect to preceding marks,
providing proper positioning for sequences of marks that attach to the
same base glyph.

