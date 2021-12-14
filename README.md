#   `% query`:  A Framework for CLI-Based Tutorials

##  What is `%query`?

`%query` is an Urbit app that turns the console interface into an interactive lesson experience.  Students can learn how Hoon and Arvo work right where they will later be developing their own tools and programs later.

##  What is a `%query` Course?

A `%query` course is a desk containing all of the files and data associated with the course, analogous to a Git repo but in Urbit's Clay filesystem.

As students progress through the ordered lessons in a course, new lessons become available to them based on dependencies.

```
Divide 30 by half and add ten.
---
A. 40.5
B. 70
C. 25
D. None of the above
~zod:query> B
```

##  Components

### Course

A course is a top-level description of a lesson experience.  This broadly maps to the overall experience of Hoon School 101, for example, or a course can be defined at a finer level as a particular collection of tutorials.

A course contains data only, which will be interpreted by the associated `%query` agent.  `%query` will scan for any desks with names prepended by `course-` (much as how the `-test` thread scans for arms beginning with `test-`).

A course has a top-level manifest file containing metadata and information about the course, course structure, authorship, license, etc.

```
course-title
├── course/manifest
└── lesson-title
    ├── init/hoon
    ├── depends/json
    ├── lesson/manifest
    ├── questions/lesson
    └── tests/hoon
```

#### `manifest-course`

Similar to a desk's `docket` file, a manifest file is intended to describe a course or lesson sufficient for a student or teacher to know how it should be used.  A `manifest-course` file contains the following information about a course:

```
+$  url  cord
+$  author  ?(@p @t)
+$  iso-639-1  ?('ab' 'aa' 'af' 'ak' 'sq' 'am' 'ar' 'an' 'hy' 'as' 'av' 'ae' 'ay' 'az' 'bm' 'ba' 'eu' 'be' 'bn' 'bi' 'bs' 'br' 'bg' 'my' 'ca' 'ch' 'ce' 'ny' 'zh' 'cv' 'kw' 'co' 'cr' 'hr' 'cs' 'da' 'dv' 'nl' 'dz' 'en' 'eo' 'et' 'ee' 'fo' 'fj' 'fi' 'fr' 'ff' 'gl' 'ka' 'de' 'el' 'gn' 'gu' 'ht' 'ha' 'he' 'hz' 'hi' 'ho' 'hu' 'ia' 'id' 'ie' 'ga' 'ig' 'ik' 'io' 'is' 'it' 'iu' 'ja' 'jv' 'kl' 'kn' 'kr' 'ks' 'kk' 'km' 'ki' 'rw' 'ky' 'kv' 'kg' 'ko' 'ku' 'kj' 'la' 'lb' 'lg' 'li' 'ln' 'lo' 'lt' 'lu' 'lv' 'gv' 'mk' 'mg' 'ms' 'ml' 'mt' 'mi' 'mr' 'mh' 'mn' 'na' 'nv' 'nd' 'ne' 'ng' 'nb' 'nn' 'no' 'ii' 'nr' 'oc' 'oj' 'cu' 'om' 'or' 'os' 'pa' 'pi' 'fa' 'pl' 'ps' 'pt' 'qu' 'rm' 'rn' 'ro' 'ru' 'sa' 'sc' 'sd' 'se' 'sm' 'sg' 'sr' 'gd' 'sn' 'si' 'sk' 'sl' 'so' 'st' 'es' 'su' 'sw' 'ss' 'sv' 'ta' 'te' 'tg' 'th' 'ti' 'bo' 'tk' 'tl' 'tn' 'to' 'tr' 'ts' 'tt' 'tw' 'ty' 'ug' 'uk' 'ur' 'uz' 've' 'vi' 'vo' 'wa' 'cy' 'wo' 'fy' 'xh' 'yi' 'yo' 'za' 'zu' 'xx')
+$ version  [major=@ud minor=@ud patch=@ud]
+$  clause
  $%  [%title title=@t]
      [%info info=@t]
      [%readme readme=@t]
      [%authors authors=(list author)]
      [%citation citation=@t]
      [%l10n language=iso-639-1]
      [%version =version]
      [%website website=url]
      [%license license=cord]
  ==
```

(Since `%query` is a command-line interface tool, there is no need for a `glob` etc.)

- `%title` is the top-level title of the course, such as “Hoon 101” (required)
- `%info` is a human-language description of the course, broadly equivalent to a an elevator pitch (required)
- `%readme` is a human-language longform description of the course, its goals, and so forth (optional)
- `%authors` is a list of patps or text names (required)
- `%citation` directs how citation of the materials, if any, should be made (optional)
- `%l10n` describes localization parameters, currently only the language as a [two-letter](https://en.wikipedia.org/wiki/ISO_639-1) [ISO-639 code](https://en.wikipedia.org/wiki/ISO_639), reserving `'xx'` for a language not present in that list; if multiple locales are supported, an effort should be made to synchronize major and minor version numbers of courses (required)
- `%version` is a triple of major/minor/patch version numbers (required)
- `%website` is a URL pointing to any additional associated resources (optional)
- `%license` is a [curriculum-appropriate](https://choosealicense.com/non-software/) license (such a `'cc0'`, `'cc-by-sa-4.0'`, `'public domain'`, etc.) (required)


### Lesson

A lesson is a particular thread of questions and answers.  Answers should be constrained and phrased to yield either one right answer or a narrowly-construed field of correct answers.

For instance, one lesson in a “Hoon 101” course could cover atoms and auras; another, lists and list operations.

Lessons have an order, altho some lessons may be marked as optional.  Lessons should form a directed acyclic graph to ensure that later lessons depend only on earlier lessons.

#### `manifest-lesson`

```
+$  clause
  $%  [%title title=@t]
      [%info info=@t]
      [%objectives objs=(list @t)]
      [%deps deps=(unit (list @t)]
      [%date date=@da]
      [%version =version]
  ==
```

- `%title` is the top-level title of the lesson, such as “Atoms and Auras” (required)
- `%info` is a human-language description of the lesson, broadly equivalent to a an elevator pitch (required)
- `%objectives` is a list of learning objectives associated with the lesson (required)
- `%deps` describes the lesson's antecedents, which must be completed before the lesson becomes available; use `~` if no lesson is available (required)
- `%date` describes when a lesson is made available or last modified; we note this explicitly because of the fast churn in Urbit-related media at the current time (required)
- `%version` is a triple of major/minor/patch version numbers (required)
  - Question:  how should lesson version and course version relate?

A lesson consists of many questions in an ordered list in the `questions/lesson` file.  This is essentially a JSON file with special validation mark.

```
[
  {
    "class": "short",
    "content": "Two roads diverged in a yellow wood.  Which did you take?",
    "type": "value",
    "answer": "the one less traveled by"
  },
  {
    "class": "mc",
    "content": "Two roads diverged in a wood, and which did you take?",
    "answers": ["the first kept for another day", "*the one less traveled by"]
  }
]
```

### Question

There are fundamentally four types of question entries we need to support:

0. Message (question with no answer, just text)
1. Short answer (single right value)
    1. Value (treated as text input and unparsed)
    2. Code (parsed by Hoon as code)
2. Multiple choice (select one from a field)
    1. Static (answers are known and fixed)
    2. Parametric (answers are programmatically generated from random values)
3. Long answer (as with a core definition)
    1. Value (seems very unlikely, last priority)
    2. Code (parsed by Hoon as code)

#### `%message`

A message question contains explanatory or prefatory text only, and expects no input (i.e. any input advances).

```
[
  {
    "class": "message",
    "content": "Two roads diverged in a yellow wood.  Which did you take?",
  }
]
```

#### `%short`

A short-answer question checks a value against a list of possible answers.  For instance, there are several equivalent ways to input a gate call, such as `(add 1 2)` and `%-  add  1  2`.  For a `%value` short-answer question, each case should be checked.  For a `%code` short-answer question, only the expected result need be checked.

```
[
  {
    "class": "short",
    "content": "Two roads diverged in a yellow wood.  Which did you take?",
    "type": "value",
    "answer": "the one less traveled by"
  }
]
```

#### `%mc`

A selection of possible answers is presented in random order.  Multiple choice questions may be static or parameterized.

```
[
  {
    "class": "mc",
    "content": "Two roads diverged in a wood, and which did you take?",
    "answers": ["the first kept for another day", "*the one less traveled by"]
  }
]
```

#### `%long`

The main difference for a long-form question from a short-answer question is that `Return` doesn't end the input.  Instead, two `Return`s are needed for `%query` to parse the result.

```
[
  {
    "class": "long",
    "content": "Produce a gate with a custom sample.  It should accept an atom `n` with default of 2 and add 10 to the value using the standard library `++add`.",
    "type": "code",
    "answer": "|:(n=`@`2 (add n 10))"
  }
]
```


##  References

- [Swirlify Docs](http://swirlstats.com/swirlify/introduction.html)
  > Swirl is an R package that turns the R console into an interactive learning evironment. Students are guided through R programming exercises where they can answer questions in the R console. There is no separation between where a student learns to use R, and where they go on to use R for thier own creative purposes. If you’ve never used swirl before we recommend demoing it first so you can see what we mean.
