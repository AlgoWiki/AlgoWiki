---
format: Markdown
---

# Navigating

The most natural way of navigating is by clicking wiki links that
connect one page with another. The "Front page" link in the navigation
bar will always take you to the Front Page of the wiki. The "All pages"
link will take you to a list of all pages on the wiki (organized into
folders if directories are used). Alternatively, you can search using
the search box. Note that the search is set to look for whole words, so
if you are looking for "gremlins", type that and not "gremlin".
The "go" box will take you directly to the page you type.

# Creating and modifying pages

## Registering for an account

In order to modify pages, you'll need to be logged in.  To register
for an account, just click the "register" button in the bar on top
of the screen.  You'll be asked to choose a username and a password,
which you can use to log in in the future by clicking the "login"
button.  While you are logged in, these buttons are replaced by
a "logout so-and-so" button, which you should click to log out
when you are finished.

Note that logins are persistent through session cookies, so if you
don't log out, you'll still be logged in when you return to the
wiki from the same browser in the future.

## Editing a page

To edit a page, just click the "edit" button at the bottom right corner
of the page.

You can click "Preview" at any time to see how your changes will look.
Nothing is saved until you press "Save."

Note that you must provide a description of your changes.  This is to
make it easier for others to see how a wiki page has been changed.

## Page metadata

Pages may optionally begin with a metadata block.  Here is an example$$

    ---
    format: latex+lhs
    categories: haskell math
    title: Haskell and
      Category Theory
    ...

    \section{Why Category Theory?}

The metadata block consists of a list of key-value pairs, each on a
separate line. If needed, the value can be continued on one or more
additional line, which must begin with a space. (This is illustrated by
the "title" example above.) The metadata block must begin with a line
`---` and end with a line `...` optionally followed by one or more blank
lines.

Currently the following keys are supported$$

format
:   Overrides the default page type as specified in the configuration file.
    Possible values are `markdown`, `rst`, `latex`, `html`, `markdown+lhs`,
    `rst+lhs`, `latex+lhs`.  (Capitalization is ignored, so you can also
    use `LaTeX`, `HTML`, etc.)  The `+lhs` variants indicate that the page
    is to be interpreted as literate Haskell.  If this field is missing,
    the default page type will be used.

categories
:   A space or comma separated list of categories to which the page belongs.

toc
:   Overrides default setting for table-of-contents in the configuration file.
    Values can be `yes`, `no`, `true`, or `false` (capitalization is ignored).

title
:   By default the displayed page title is the page name.  This metadata element
    overrides that default.

## Creating a new page

To create a new page, just create a wiki link that links to it, and
click the link.  If the page does not exist, you will be editing it
immediately.

## Reverting to an earlier version

If you click the "history" button at the bottom of the page, you will
get a record of previous versions of the page.  You can see the differences
between two versions by dragging one onto the other; additions will be
highlighted in yellow, and deletions will be crossed out with a horizontal
line.  Clicking on the description of changes will take you to the page
as it existed after those changes.  To revert the page to the revision
you're currently looking at, just click the "revert" button at the bottom
of the page, then "Save". 

## Deleting a page

The "delete" button at the bottom of the page will delete a page.  Note
that deleted pages can be recovered, since a record of them will still be
accessible via the "activity" button on the top of the page.

# Uploading files

To upload a file--a picture, a PDF, or some other resource--click the
"upload" button in the navigation bar.  You will be prompted to select
the file to upload.  As with edits, you will be asked to provide a
description of the resource (or of the change, if you are overwriting
an existing file).

Often you may leave "Name on wiki" blank, since the existing name of the
file will be used by default.  If that isn't desired, supply a name.
Note that uploaded files *must* include a file extension (e.g. `.pdf`).

If you are providing a new version of a file that already exists on the
wiki, check the box "Overwrite existing file." Otherwise, leave it
unchecked.

To link to an uploaded file, just use its name in a regular wiki link.
For example, if you uploaded a picture `fido.jpg`, you can insert the
picture into a (markdown-formatted) page as follows: `![fido](fido.jpg)`.
If you uploaded a PDF `projection.pdf`, you can insert a link to it
using:  `[projection](projection.pdf)`.



# Markdown

This wiki's pages are written in [pandoc]'s extended form of [markdown].
If you're not familiar with markdown, you should start by looking
at the [markdown "basics" page] and the [markdown syntax description].
Consult the [pandoc User's Guide] for information about pandoc's syntax
for footnotes, tables, description lists, and other elements not present
in standard markdown.

[pandoc]: http://pandoc.org
[pandoc User's Guide]: http://pandoc.org/README.html
[markdown]: http://daringfireball.net/projects/markdown
[markdown "basics" page]: http://daringfireball.net/projects/markdown/basics
[markdown syntax description]: http://daringfireball.net/projects/markdown/syntax 

Markdown is pretty intuitive, since it is based on email conventions.
Here are some examples to get you started$$

<table>
<tr>
<td>`*emphasized text*`</td>
<td>*emphasized text*</td>
</tr>
<tr>
<td>`**strong emphasis**`</td>
<td>**strong emphasis**</td>
</tr>
<tr>
<td>`` `literal text` ``</td>
<td>`literal text`</td>
</tr>
<tr>
<td>`\*escaped special characters\*`</td>
<td>\*escaped special characters\*</td>
</tr>
<tr>
<td>`[external link](http://google.com)`</td>
<td>[external link](http://google.com)</td>
</tr>
<tr>
<td>`![folder](/img/icons/folder.png)`</td>
<td>![folder](/img/icons/folder.png)</td>
</tr>
<tr>
<td>Wikilink: `[Front Page]()`</td>
<td>Wikilink: [Front Page]()</td>
</tr>
<tr>
<td>`H~2~O`</td>
<td>H~2~O</td>
</tr>
<tr>
<td>`10^100^`</td>
<td>10^100^</td>
</tr>
<tr>
<td>`~~strikeout~~`</td>
<td>~~strikeout~~</td>
</tr>
<tr>
<td>
`$x = \frac{{ - b \pm \sqrt {b^2 - 4ac} }}{{2a}}$`
</td>
<td>
$x = \frac{{ - b \pm \sqrt {b^2 - 4ac} }}{{2a}}$^[If this looks like
code, it's because jsMath is
not installed on your system.  Contact your administrator to request it.]
</td>
</tr>
<tr>
<td>
`A simple footnote.^[Or is it so simple?]`
</td>
<td>
A simple footnote.^[Or is it so simple?]
</td>
</tr>
<tr>
<td>
<pre>
> an indented paragraph,
> usually used for quotations
</pre>
</td>
<td>

> an indented paragraph,
> usually used for quotations

</td>
<tr>
<td>
<pre>
    #!/bin/sh -e
    # code, indented four spaces
    echo "Hello world"
</pre>
</td>
<td>

    #!/bin/sh -e
    # code, indented four spaces
    echo "Hello world"

</td>
</tr>
<tr>
<td>
<pre>
- a bulleted list
- second item
    - sublist
    - and more
- back to main list
    1. this item has an ordered
    2. sublist
        a) you can also use letters
        b) another item
</pre>
</td>
<td>

- a bulleted list
- second item
    - sublist
    - and more
- back to main list
    1. this item has an ordered
    2. sublist
        a) you can also use letters
        b) another item

</td>
</tr>
<tr>
<td>
<pre>
Fruit        Quantity
--------  -----------
apples         30,200
oranges         1,998
pears              42

Table:  Our fruit inventory
</pre>
</td>
<td>

Fruit        Quantity
--------  -----------
apples         30,200
oranges         1,998
pears              42

Table:  Our fruit inventory

</td>
</tr>
</table>

For headings, prefix a line with one or more `#` signs:  one for a major heading,
two for a subheading, three for a subsubheading.  Be sure to leave space before
and after the heading.

    # Markdown

    Text...
 
    ## Some examples...
   
    Text...

## Wiki links

Links to other wiki pages are formed this way:  `[Page Name]()`.
(Gitit converts markdown links with empty targets into wikilinks.)

To link to a wiki page using something else as the link text$$
`[something else](Page Name)`.

Note that page names may contain spaces and some special characters.
They need not be CamelCase.  CamelCase words are *not* automatically
converted to wiki links.

Wiki pages may be organized into directories.  So, if you have
several pages on wine, you may wish to organize them like so$$

    Wine/Pinot Noir
    Wine/Burgundy
    Wine/Cabernet Sauvignon

Note that a wiki link `[Burgundy]()` that occurs inside the `Wine`
directory will link to `Wine/Burgundy`, and not to `Burgundy`.
To link to a top-level page called `Burgundy`, you'd have to use
`[Burgundy](/Burgundy)`.

To link to a directory listing for a subdirectory, use a trailing
slash: `[Wine/]()` will link to a listing of the `Wine` subdirectory.
