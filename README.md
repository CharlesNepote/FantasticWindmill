Fantastic Windmill
==================

Fantastic Windmill (FW) is a _static web site generator_: it reads
content from a set of source files, applies a presentation template to them
and saves the output as a set of static HTML files. It is similar in this
respect to a lot of other (good) static generators, such as [Pelican](http://docs.getpelican.com/), [nanoc](http://nanoc.stoneship.org/), and [Blogofile](http://www.blogofile.com).

The main difference is that FW relies on a very small set of concepts to
provide the same functionality. As a matter of fact, if you know how to
write web pages in PHP, then setting up a static web site 
using FW should be very, very easy.
We hereby brag that FW has the shortest documentation of all static web site
generators: this single page is all you need to know to use Fantastic
Windmill.

## Quick links

*   [Why use a static generator?](#why)
*   [Quick start](#quickstart)
*   [Defining metadata for a page](#metadata)
*   [Switching templates](#switch-template)
*   [Clean URLs](#clean-urls)
*   [Using the example web site](#example-website)
*   [Using the Makefile](#makefile)
*   [Why another static generator?](#why-another)
*   [About the author](#about)

<a name="why"></a>

## Why use a static generator?

There exist a number of advantages for the use of a static generator over
traditional content management systems (CMS) like [Wordpress](http://www.wordpress.com/) or [Joomla](http://www.joomla.org/). Since the whole web site is
generated in advance, the server only needs to host a set of static HTML
files, thereby reducing processing time when a page is requested, and
eliminating the security vulnerabilities inherent in the use of a CMS. In
addition, the content used to generate the pages is hosted locally in a set
of files managed by the user; page contents, blog posts and images are not
"trapped" inside the CMS as some obscure database entry that is hard to pull
out or move around.

Obviously, some dynamic features present in a CMS are absent from a
statically-generated web site. For example, readers cannot post comments or
obtain results based on an input query (although some of these
functionalities can be achieved using JavaScript and external services, such
as [Disqus](http://www.disqus.com/)). However, a static generator
does create the HTML pages based on content; this means that a newly added
blog post will show up in the (static) list of posts (or wherever else) the
next time the site is rebuilt. As a rule, one regenerates the web site every
time some content changes, and uploads to the web server only the HTML files
that have been modified. It turns out that this process is adequate to
maintain many a web site.

<!-- ## Why use Fantastic Windmill?

FW has some features not very common in other static generators:
* manage multi-languages
* user-defined metadata
* choice for metadata place (.yaml or at the botom of .md files)
* .md file format compatible with Jekill
* no framework
* Content structure = output structure
-->


<a name="quickstart"></a>

## Quick start

First, [download
the Fantastic Windmill archive](https://github.com/sylvainhalle/FantasticWindmill/archive/master.zip). Then, create a folder that will contain
both the source files and output files of your web site. In this example we
will call it <tt>mysite/</tt>. From the downloaded archive, make sure you
copy at least the <tt>fw/</tt> folder and put it into <tt>mysite/</tt>.
Optionally, you can also copy the <tt>Makefile</tt> and the
<tt>templates/</tt> folder from that same archive.

Note that **php-yaml** has to be installed. See: https://code.google.com/p/php-yaml/wiki/InstallingWithPecl

### Step 1: write content

Write a set of files in [Markdown syntax](http://daringfireball.net/projects/markdown/syntax)
(note that you can write plain
HTML inside a Markdown file); save each file with the <tt>.md</tt> extension
into a folder called <tt>mysite/content/</tt>. You can put your files into any
folder structure inside <tt>content/</tt>. Each file <tt>filename.md</tt>
will become a single
_page_ called <tt>filename.html</tt>
in the resulting web site. If a page has links to other
documents (e.g. it contains images, or links to other files within your
web site), you can use relative links to
refer to those documents. Everything that is put into <tt>contents/</tt>
is intended to be present in the generated web site (with the exception of <tt>.md</tt>
files).

### Step 2: write a template

Write a set of template files and put them in a folder called
<tt>mysite/templates/</tt>. A template can
be any PHP file; it is called by FW to generate HTML output based on the
content of each Markdown document in the <tt>content/</tt> folder.
FW passes to the template two special variables:

*   <tt>$page</tt> is an object representing the page currently
  being rendered.
 This object has two important fields: <tt>$page->data</tt> is an array
 of metadata (this is where information such as title, author, etc. is
 placed; see below on [how to define metadata for a page](#metadata)); <tt>$page->dom</tt>
 is a PHP [DOMDocument](http://php.net/manual/en/class.domdocument.php) object
 containing the HTML rendition of <tt>filename.md</tt>. Use the
 <tt>saveXML()</tt> method of the DOMDocument to get the HTML string corresponding
 to the object.

*   <tt>$pages</tt> is an array of all page objects (hence the content and metadata
of all pages can be accessed by the template)

By default,
FW looks for a file called <tt>base.php</tt> to render a page: make sure
one of the files has this name. See below to
[select different template files](#switching-templates)
to render different pages.

### Step 3: generate the site

From the <tt>mysite/</tt> folder, call

<pre>
php fw/fw.php
</pre>

FW will process each input file in the <tt>mysite/contents/</tt> folder
and output the resulting HTML file in folder <tt>mysite/public_html/</tt>.
You are now ready to upload that folder to your web server.

The previous command only takes care of generating the HTML files, but
does not mirror the other files (images, etc.) from <tt>contents/</tt> to
<tt>public_html</tt>. You can either do it by yourself, or run

<pre>
make mirror
</pre>

to take care of this. The <tt>Makefile</tt> script can be called with
[other options](#makefile) as well.

<a name="metadata"></a>

## Defining metadata for a page

In addition to its actual
contents, each page in the web site can be associated
to bits of information, such as a date, an author name, a title or an
abstract. You are free to define whatever parameters and values you wish
for any page, using [YAML](http://www.yaml.org/) notation.
Please refer to the YAML web site for help on the (fairly intuitive)
syntax of this language.

### Inside the page

The first way to define metadata is to add a YAML section at the end
of a Markdown input file.  The start of YAML
declarations is marked by a line containing three dashes (<tt>---</tt>).
Everything that follows will be parsed as YAML and removed from the page
for further processing (hence the YAML part does not show up in the final
page). Here is a simple example that defines the value of a parameter
called <tt>author</tt> for a page:

<pre>
# A title for the page

Hello, this is a paragraph inside
the page, with a single sentence.

---
author: Emmett Brown
...
</pre>

FW itself does nothing with the metadata. However, your template files
can refer to it. In this example, the value of the <tt>author</tt> metadata
of the current page is accessible to the template by writing
<tt>$page->data["author"]</tt>. You can use it to display the author name;
however, as a template file can include any PHP code, you can use this value
in any way you like (for example, displaying a different message depending
on the author, etc.).

We stress that FW does not impose any parameter of any name (with a
single exception, see below). A page does not need to have an author, and
that field does not need to be called that way.

### Next to the page

The second way to define metadata is to put the YAML declarations in
a separate file, with the same name as the input file but with extension
<tt>.yaml</tt>. This is useful if you want to use a document in your web
site, but don't want to modify it directly.

### In <tt>_.yaml</tt>

The last way of defining metadata is to put YAML declarations in a file
called <tt>_.yaml</tt>. All pages in the same directory
inherit from the metadata present in that file (if present). Hence, if
all pages from a directory have the same author, you can put that declaration
in <tt>_.yaml</tt> instead of repeating it for every file.

### All together now

Metadata can be defined using a mix of all previous ways. When
processing <tt>filename.md</tt>, FW looks for
metadata in the following order:

1.  In <tt>mysite/contents/_.yaml</tt>, if it exists, and then recursively
into every <tt>_.yaml</tt> along the path to the page being processed
2.  In <tt>filename.yaml</tt>
3.  At the end of <tt>filename.md</tt>

When a metadata field with given name already exists, the newer one
overwrites the older one. Hence <tt>_.yaml</tt> can be used to set
default values that can then be overridden by some files or folders.
A file <tt>_.yaml</tt> placed at the root of the site (i.e. into
<tt>contents/</tt>) can be used to define site-wide metadata, such that
the site's name, etc. Hence in FW, there is no need for a separate
mechanism to define "site" properties vs. "page" properties.

### Inferred metadata

Although metadata has no particular meaning in FW, for convenience a
couple of fields are auto-generated from a page contents:

*   <tt>date</tt> is inferred from the file's modification date
*   <tt>title</tt> is the content of the first <tt>&lt;h1&gt;</tt>
  element in the page, if any
*   <tt>abstract</tt> is filled if the next element following <tt>&lt;h1&gt;</tt>
 is a <tt>&lt;blockquote&gt;</tt>; its value is the content of that
 <tt>&lt;blockquote&gt;</tt>

<a name="switch-templates"></a>

## Switching templates

Metadata fields in FW have no meaning, except for one. If a page
has a metadata field called <tt>template</tt>, its value will be used by
FW as the name of the template file used to render that page (overriding
the default value of <tt>base.php</tt>. Hence, if you put all blog posts
into a folder called <tt>mysite/contents/posts/</tt>, you can put there
a file <tt>_.yaml</tt> with the instruction <tt>template: post.php</tt>
to tell FW to use the <tt>post.php</tt> template file to render all posts
in the folder.

Of course, as with any other metadata field, the template can also be
defined on a per-page basis.

<a name="clean-urls"></a>

## Clean URLs

FW can be run with the <tt>--clean-urls</tt> command-line option.
This has for effect of removing the <tt>.html</tt> extension of all
_local_ links inside the pages. Use this option in conjunction with
Apache's mod_rewrite module by uploading a <tt>.htaccess</tt> file at the
root of your site, such
as this one:

<pre>
RewriteEngine On
RewriteCond %{SCRIPT_FILENAME} !-d
RewriteRule ^([^\.]+)$ $1.html [NC,L]
</pre>

<a name="example-website"></a>

## Using the example web site

<p>The FW archive is actually an example web site in itself, complete with
a <tt>contents/</tt> and a <tt>templates/</tt> folder. It shows how FW can
be used to generate a bilingual web site with a blog section. Look into
the <tt>templates/</tt> folder to learn how FW's variables can be used for
various pages (list of posts, main page, etc.).

<a name="makefile"></a>

## Using the Makefile

The archive
ships with a sample <tt>Makefile</tt> you can use to automate some
tasks outside of FW.

*   <tt>make html</tt> and <tt>make publish</tt> both run FW. The difference
is that <tt>make html</tt> generates pages for "local" viewing, while<tt>make publish</tt> produces pages ready to be uploaded to the web server.
For example, one can define some "local" code into a file <tt>code_local.php</tt>,
and some "remote" code <tt>code_remote.php</tt>. The site's template can
then include <tt>code.php</tt>; running make will copy into that file
the contents of either <tt>code_local.php</tt> or <tt>code_remote.php</tt>,
depending on the option make was called with.
*   <tt>make mirror</tt> copies all files from the <tt>content/</tt> folder
to the <tt>public_html/</tt> folder, with the exception of <tt>*.md</tt> and
<tt>*.yaml</tt>. By default running FW only puts HTML files into <tt>public_html/</tt>
but leaves everything else as is.
*   <tt>make ftp_upload</tt> uploads the contents of <tt>public_html/</tt>
to a web server; the URL and login credentials should be written at the
beginning of the Makefile. The FTP password should be written into a file
called <tt>ftp_password.txt</tt>; this is useful when using Git to manage
the site, since one can exclude the password file from versioning.
This option uses <tt>rsync</tt> to upload the
files, so that only the files modified since the last upload are transferred.

<a name="why-another"></a>

## Why another static generator?

FW achieves roughly the same functionality as most other static web site
generators. It distinguishes itself in a couple of design choices that
address what we feel are shortcomings of existing tools.

### Content structure = output structure

It is very hard to tell many static generators where you want the
resulting pages to go. Many of them disregard any folder structure you give
to your content and shuffle all pages according to their own (mostly flat)
directory scheme. Many times, even the filenames are transformed before
outputting the final pages. Mirroring the organization of your input to the
output folder generally involves hacks such as including the directory names
into page IDs, moving the files around after the generator processed them or
even fiddling with the generator's source code. This is even worse if one of
your pages is linked to other files (for example: by containing images):
those external files must be separated from the page and put into some other
("static") folder. This makes using each page as a stand-alone document
difficult.

FW applies the "[don't repeat
yourself](http://en.wikipedia.org/wiki/Don%27t_repeat_yourself)" principle. There is probably a reason why you place a file in
some folder and give it some name. Therefore, FW's output folder always
mirrors the structure and names of the input folder.


### User-defined metadata

Generators give access to very few metadata about a
page (e.g. date, author, title). In addition, this metadata is hard-coded
into the tool, which generally won't let you add any other parameter
you might want to use in your templates.

In FW, a page's metadata can have any structure you wish: anything
that can be written in [YAML](http://www.yaml.org/) can be
attached to a page and queried by any template. There is no
set of predefined parameters, and pages need not have the same parameters.


### Flexible location for page metadata

Not only is metadata relatively limited with existing generators,
the location of this metadata is also very rigid: it is generally located
at the beginning of an input file, before the actual text.
Hence, if one wants to use an input file independently of its use in
the web site, it must
be trimmed of that leading metadata to become legible. Moreover,
 no metadata can
be shared between e.g. all pages of the same folder. Hence if all pages in
a folder have the same author, the author field must be repeated in all the
input files.

In FW, a page's metadata can be put in three locations:

1.  At the _end_ of the source file (rather than at the beginning);
a page starts with its actual contents rather than its metadata
2.  In a separate file with the same
name, but with the extension <tt>.yaml</tt>; hence one does not need to modify a
file for it to be used in the web site
3.  In a file called <tt>_.yaml</tt>. All pages in the same directory
inherit from the metadata present in that file (if present), unless overriden
by declarations 1 or 2 (again, don't repeat yourself)


### PHP as the template engine

Most generators let you create templates for pages using one of many HTML
template "languages", such as [Jinja](http://jinja.pocoo.org/) or
[Django](https://djangoproject.com/), which offer only very basic
programming constructs for iterating over objects, evaluating conditions,
calling external functions or passing arguments. This makes seemingly
straightforward templates next to impossible to write --for example, listing
up to 5 blog entries with the same ID, but a different language as the
current page, sorted by author name. 

In FW, a template is anything you can write in PHP. You can create
variables and objects, call whatever function is available in the language,
use whatever construct you wish (loops, switches, you name it) or include
any other PHP file within your template file. Similarly, each template can
access the metadata of _all_ the site's pages, giving you complete
flexibility on the processing of what you want to display.

<a name="about"></a>

## About the author

Fantastic Windmill was written by [Sylvain Hallé](http://www.leduotang.com/sylvain/), professor at [Université du Québec à Chicoutimi](http://www.uqac.ca/), who got
tired of using a CMS, was dissatisfied with existing static web site
generators, and thinks windmills are fantastic.

