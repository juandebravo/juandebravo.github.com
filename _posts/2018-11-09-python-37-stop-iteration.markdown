---
layout: post
title: "Python 3.7: StopIteration could be your next headache"
categories: [python, generators]
---

I upgraded the python interpreter in one of the components I ran in production
from 3.5 to 3.7. Main reason for that was testing [**dataclasses**](https://docs.python.org/3/library/dataclasses.html) functionality. Sometime ago
I [wrote a post](https://www.juandebravo.com/2014/01/04/python-auto-initializer/)
about how to build a simply automatic initializer, and eventually it's implemented
in the standard library, which I believe is a great new feature!

The upgrade was quite smooth, even though I had an issue due to a non backwards
compatible change in how a generator should flag it's exhausted.

*Making a long story short*: use `return` instead of `StopIteration` when you want
to stop sending items from your generator.

Next is the *long story*, things I learned while investigating the issue and
the reason why I didn't capture it before hand.

## Lesson 1: Read the release notes

This may sound obvious, but...

    Remember, a few hours of trial and error can save you several minutes
    of looking at the README.

[I am devloper (7 Nov 2018)](https://twitter.com/iamdevloper/status/1060067235316809729)

[Python 3.7 release notes](https://docs.python.org/3/whatsnew/3.7.html) explains
very clearly [changes that this new version brings into python behavior](https://docs.python.org/3/whatsnew/3.7.html#changes-in-python-behavior), being one
of them [PEP-479](https://www.python.org/dev/peps/pep-0479/). This PEP introduces a
non backwards compatible change in how a generator behaves.

I was so focused on the new feature I wanted to test, I didn't go over the
release notes.

## Lesson 2: The process

When you have a containerized environment, it's quite straight forward
to change/upgrade a specific component execution platform. We've been
using Python 3.5 as our default runtime for a while, but I was intrigued
to upgrade to newest versions and check the new functionalities provided
by the language.

To be specific, I read some months ago about [*dataclasses*](https://docs.python.org/3/library/dataclasses.html) and really looked to
me like a nice functionality to reduce boilerplate code. Also, [PEP563 -
Postponed evaluation of annotations](https://docs.python.org/3/whatsnew/3.7.html#whatsnew37-pep563)
is a great new feature, in case you're using annotations in your code.

In order to test the python interpreter upgrade, I chose as candidate an
offline component which main goal is accessing gDrive API. Our library implements
a tiny wrapper over [the official python client library](https://developers.google.com/api-client-library/python/start/get_started).

Find below a simplified snippet that, given a gDrive folderId, returns a generator
which includes every file under that folder. As filenames are returned in several
pages to prevent a very long response, we prefer using a generator instead
of a list, so it's up to the client to iterate as much as it's needed (and therefore
request additional pages under the hood):

```python
def get_files_in_folder(folder_id, next_page_token=None):
    # Request a page
    _files, next_page_token = _get_files_page(folder_id, next_page_token)

    # Return every file included in the results
    for f in _files:
        yield f

    if next_page_token is None:
        # Flag generator is exhausted
        raise StopIteration
    else:
        # Request next page
        yield from get_files_in_folder(folder_id, next_page_token=next_page_token)

def _get_files_page(folder_id, page_token):
    results = get_service().files().list(
        pageSize=25,
        pageToken=page_token,
        fields="files(id,name),incompleteSearch,kind,nextPageToken",
        q="'{0}' in parents".format(folder_id)
    ).execute()

    return results.get('files', []), results.get('nextPageToken')
```

Right, probably raising a StopIteration instead of simply return was not a good idea, but
this code works properly in python 3.5, but does not in 3.7 due to the change introduced
in the mentioned [PEP479](https://www.python.org/dev/peps/pep-0479/).

Simply changing:

```python
# Flag generator is exhausted
raise StopIteration
```
with

```python
# Flag generator is exhausted
return
```
solved the issue.

## Lesson 3: Check silent deprecation warnings

[PEP-479](https://www.python.org/dev/peps/pep-0479/#transition-plan) describes very clearly
the transition plan in regards to the backwards incompatible change:

- Python 3.5: Enable new semantics under __future__ import; silent deprecation warning if StopIteration bubbles out of a generator not under __future__ import.
- Python 3.6: Non-silent deprecation warning.
- Python 3.7: Enable new semantics everywhere.

As I upgraded from 3.5 to 3.7, I didn't get any deprecation warning. Configuring [PYTHONWARNINGS](https://docs.python.org/3.5/library/warnings.html) environment variable is a straight forward
way to get this kind of warnings.

## Lesson 4: Bring future default behaviour to your code

I took this opportunity to review what exactly [**__future__ statements**](https://docs.python.org/3.5/reference/simple_stmts.html#future-statements) tackles.

In regards to StopIteration breaking change, Python 3.5 introduced the chance to write
your code in the way that is required in 3.7 (check the first point in the transition
plan above). It's really convenient to be ready for your next interpreter/library
upgrade. In this specific case, all I needed to do was adding the following
line on top of the gdrive API wrapper:

```python
from __future__ import generator_stop
```
That way, running the code in a python 3.5 will behave as python 3.7, hence raising an error
unless `StopIteration` is changed by `return`.
