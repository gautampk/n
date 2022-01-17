# n

*Yet another command line note app*

`n` is a lightweight, stateless text file management system that integrates
smoothly with all other Unix tools.

## Advantanges of `n`

* Stateless: there are no index files, no databases, no nothing. Just your
  plaintext files in your normal folders.
* Lightweight: `n` is written in bash and is built entirely out of standard Unix
  tools such as `grep`, `sed`, and `ls`. There are no dependencies and nothing
  to compile.

## How does it work?

The main advantage of a "notes management system" is easy searching of notes and
the provision of a short unique identifier (so you don't need to remember the
filename and/or note title).

`n` does this in a stateless way by allowing the note ID to be slightly
volatile: in particular, if you add a new note or subfolder, or delete a note or
subfolder, the IDs of all notes can change. This is similar to how the IDs used
by Taskwarrior work.

The downside of this is that you will need to check your note IDs periodically.
The upside is that there's no database. And, let's face it, how many note IDs
were you ever going to remember long-term?

## Installation

1. Download the `n` shell script and put it somewhere in your `$PATH`.

2. Update your `.bashrc` (or `.zshrc` or `.profile` or other equivalent) so that
the `$NOTES` environment variable is always set when you log in to point to your
notes folder. For example:

```bash
export NOTES=/home/<user>/.notes
```

Make sure that this is the absolute path to your notes folder, or you may run
into some trouble.

All done!

## How to do things

### Shortnotes

Shortnotes are quick, one-line notes that you can add via a simple command:

```bash
% n a "This is a short note"
```

These get added to a daily shortnotes file formatted using Org Mode syntax,
which is a lot like Markdown. This is what the shortnotes file looks like:

```org
#+TITLE: 17th Jan 2022 Short Notes
#+DATE: <2022-01-17 Mon>

* A quick note
  /12:02:45/
```

The above command will add a new note to this file, or create it if it does not
yet exist. After running the command the file would look like:

```org
#+TITLE: 17th Jan 2022 Short Notes
#+DATE: <2022-01-17 Mon>

* A quick note
  /12:02:45/

* This is a short note
  /22:14:57/
```

You can easily get the absolute filepath to today's shortnotes file with the
command `n t`. Many features of `n` revolve around retrieving the absolute
filepath, as that is the thing you can use to manipulate your notes with other
standard GNU and Unix programs. For example, if you want to see what's currently
in today's shortnotes file:

```bash
% cat $(n t)
```

### Normal notes

You can create a new note with the `n c` command:

```bash
% n c [org|md] "My new note"
```

This will create a note with the filename `[date]-My_new_note` and the correct
file extension (either `.org` or `.md`) and then open it in your chosen text
editor (set by `$EDITOR`). If you don't specify a filetype, `n` will assume
Org syntax by default.

To create an empty note with a custom filename, you can use the `n p` command
to construct the correct path:

```bash
% vim $(n p)/my_note.org
```

You can also use this to directly open a note if you can already remember its
filename.

If you can't remember the filename (i.e., most of the time, probably), you can
use the list (`n ls`) and query (`n q [search string]`) functions to find it.
Both of these commands output a numbered list of notes and notebooks (i.e.,
folders; more on that later). For example:

```bash
% n ls
	1	A test file (220117-test.md)
	2	This is a note (220117-This_is_a_note.md)
	3	17th Jan 2022 Short Notes (220117-shortnotes.org)
	4	Another note (note.org)
	5	220116 Meeting Minutes (220116-meeting.org)
	6	work/
	7	home/
```

You can then use the command `n [ID]` to fetch the absolute filepath so you can
open the note. For example, to open the fourth note ('Another note') above in
Vim:

```bash
% vim $(n 4)
```

As you can see, `n ls` automatically fetches the title from inside the notes.
This works for both Org (using the `#+TITLE`) and Markdown (taking the first
top-level heading) files.

The query command (`n q`) searches through the contents of all notes in the
working notes folder. You can use whatever tagging system suits you best
and put the tags wherever you want inside your notes files, and `n q` will
find them. Or, you can not use tags and just search for keywords in the text
of your notes. It's entirely up to you. `n q` is built on a Grep call, so
anything that will work in Grep will work as a search string (including
regex). The output of `n q [search]` is similar to `n ls`:

```bash
% n q "test"
	1	220117-test.md
		1:# A test file
	5	220116-meeting.org
		7:John Smith is in charge of the project test run.
```

The IDs associated with each note are semi-volatile. They will not change as
long as no notes or sub-folders are added to or removed from the working notes
folder. If changes do happen, it is highly likely the IDs will change. That is
the trade-off being made for statelessness.

### Notebooks

Notebooks in `n` are just sub-folders of `$NOTES`. You can set the current
working sub-directory of `n` by setting the `$NB` environment variable. By
default, `n` operates in `$NOTES`, but if `$NB` is set, then it will operate in
`$NOTES/$NB` instead.

For example, suppose we have the following file structure:

```
.notes/
  |
  home/
  | |
  | notes...
  |
  work/
  | |
  | notes...
  |
  notes...
```

and have set `NOTES=/path/to/.notes` in `.bashrc`. By default, `n` will only be
operating on the notes directly in the `.notes/` folder. You can tell `n` to
operate on a notebook (sub-folder) for a specific command by passing command
variables:

```bash
% NB=work n ls
```

This will list all the notes in the `.notes/work/` folder. If you want to set
a default notebook, you can set `$NB` in your `.bashrc` along with `$NOTES`. In
that case, command variables can be used as an override. Note IDs are specific
to a given folder, so `NB=work n 1` and `NB=home n 1` refer to two different
files!

As you may have realised, the `n p` command simply returns `$NOTES` or
`$NOTES/$NB` as appropriate.

## Org Processing Scripts

The scripts `org2md` and `org2task` deal with Org files. `org2md` is a Pandoc
wrapper that deals with some of the issues Pandoc has in processing Org files.
`org2task` interfaces Org Mode TODOs with Taskwarrior.

### `org2md`

When Pandoc converts Org files to Markdown, it strips out the `#+TITLE` and
`#+DATE` lines completely. This is deeply unhelpful. `org2md` fixes this by
making `#+TITLE` into a top-level heading and making one-star Org headings into
second-level Markdown headings, etc. For example, `org2md` will convert the
above example Org shortnotes file to:

```md
# 17th Jan 2022 Short Notes

* 2022-01-17 Mon *

## A quick note
*12:02:45*

## This is a short note
*22:14:57*
```

`org2md` can accept a filename as an argument or can accept a file on STDIN.

### `org2task`

`org2task` takes the filename of an Org file as an argument and searches for
any outstanding Org Mode TODOs in the file. The TODOs are added to Taskwarrior,
a Taskopen-compatible reference to the Org file is added as an annotation, and
the TODO status is changed to FILED in the Org file.

I find it an extremely helpful way to ingest meeting minutes into Taskwarrior.
