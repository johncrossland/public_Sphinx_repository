========================================================
Sphinx with Markdown walkthrough for HTML and PDF output
========================================================

This walkthrough installs Sphinx and configures it to output HTML and PDF from ``.md``.
If you install it on a VM, allocate over 25GB storage and multiple processors.
You'll need Ubuntu 16.04 LTS, an internet connection, and ``sudo`` rights.

.. contents:: :depth: 2

PDF output from Markdown
------------------------

This edition of the walkthrough corrects an issue with using ``make latexpdf`` previously.
``make latexpdf`` would produce ``.tex`` output from ``.md`` for Sphinx to make a ``.pdf``.
But ``make latexpdf`` did not successfully make tables in ``.pdf`` from tables in ``.md``.

This was due to the ``sphinx-markdown-tables`` package not outputting ``.tex`` from ``.md``.
``sphinx-markdown-tables`` will output ``.html`` from ``.md``, but not ``.tex`` from ``.md``.
``.tex`` is required for ``pdflatex``, which is run by Sphinx to make ``.pdf`` from ``.tex``.

``sphinx-markdown-tables`` has a issue with ``.md`` tables for ``.pdf`` that won't be fixed.
(https://github.com/ryanfox/sphinx-markdown-tables/issues/3). ``pandoc`` is the work-around.

``pandoc`` and ``texlive`` are installed and ``make latex`` is used to do this successfully.
To make these steps portable, put ``sudo`` commands in a ``.sh`` script to run from ``bash``.
You'll also need to edit the ``Makefile`` produced by Sphinx to run ``pandoc`` for ``.pdf``.

Customizing HTML output
-----------------------

When generating HTML documentation with Sphinx, you can adjust some theme related settings with
``html_theme_options`` in ``conf.py``. However, the available options are restricted since they
are restricted to the ``[options]`` section in ``theme.conf``, which are specific to a given theme.

So to customise the theme related settings on a global basis, you can define ``htm_context``
to add custom CSS defined in ``_static/custom.css`` that gets included in the generated HTML.

The steps to configure ``conf.py`` after intallation have been extended for ``custom.css``.
``custom.css`` is applied to override the default CSS applied by bootstrap and bootswatch.
Sphinx imports bootstrap and bootswatch at build time, so ``custom.css`` is needed as well.
Using ``custom.css`` makes these steps portable to run Sphinx on another VM, or via Jenkins.

You must download a ``Bootswatch 3.3.7`` CSS to edit for a ``custom.css`` from https://bootswatch.com/3/
The default Bootswatch URL (https://bootswatch/com) will download ``Bootswatch 4.1.3`` that won't work.

Installation
============

Update Ubuntu
-------------

::

  $ sudo apt update -y
  $ sudo apt upgrade -y
  $ sudo apt dist-upgrade -y

.. note::

   Updates may take 5 minutes to complete.

Install Python and update pip
-----------------------------

::

  $ sudo apt-get install python-pip -y
  $ sudo pip install --upgrade pip

Install Sphinx and HTML themes
------------------------------

::

  $ sudo pip install sphinx
  $ sudo pip install sphinxjp.themes.basicstrap
  $ sudo pip install sphinx_bootstrap_theme

Install TeX Live for PDF
------------------------

::

  $ sudo apt-get install texlive-full -y

.. note::

   Installing and unpacking Tex Live may take 20 minutes.

Install ``recommonmark`` to parse ``.md``
-----------------------------------------


::

  $ sudo pip install recommonmark

Install ``sphinx-markdown-tables`` version ``0.0.3`` to parse ``.md`` tables for HTML output
--------------------------------------------------------------------------------------------

::

  $ sudo pip install sphinx-markdown-tables==0.0.3

Running just ``sudo pip install sphinx-markdown-tables`` without specifying the version installs the latest version.
I initially installed version ``0.0.3`` that worked then reinstalled the latest version ``0.0.8``, which did not run.
``sudo pip install sphinx-markdown-tables==0.0.3`` installs version ``0.0.3``, which I've tested and know works well.

Install ``pandoc`` version ``1.16.0.2`` to parse ``.md`` tables for PDF output
------------------------------------------------------------------------------

::

  $ sudo wget https://github.com/jgm/pandoc/releases/download/1.16.0.2/pandoc-1.16.0.2-1-amd64.deb
  $ sudo dpkg -i pandoc-1.16.0.2-1-amd64.deb

Running just ``sudo apt-get install pandoc`` without specifying the version installs the latest version.
I had problems running the latest version so I install the version I have tested and know works reliably.
``sudo dkpg -i pandoc-1.16.0.2-1-amd.deb`` installs version ``1.16.0.2`` after first downloading it with:
``sudo wget http://github.com/jgm/pandoc/releases/download/1.16.0.2/pandoc-1.16.0.2-1-amd.deb``.

.. note::

   This is the last installation step. Close the terminal and open a new terminal without ``sudo``, as the ``sphinx-quickstart``
   script creates privileged files to edit. If you don't then you'll need to ``chown`` privileged files, so you can edit them.

Configuration
=============

Run Sphinx Quickstart
---------------------

::

  $ cd Documents
  /Documents$ sphinx-quickstart
  Welcome to the Sphinx quickstart utility.

  Please enter values for the following settings (just press Enter to accept a default value, if one is given in brackets).

::

  > Separate source and build directories (y/n) [n]:
  y

::

  > Name prefix for templates and static dir [_]:
  <Enter>

::

  > Project name:
  <your_project>

::

  > Author name(s):
  <your_name>

::

  > Project release []:
  <Enter>

::

  > Project language [en]:
  <Enter>

::

  > Source file suffix [.rst]:
  <Enter>

::

  > Name of your master document (without suffix) [index]:
  <Enter>

::

  Sphinx can also add configuration for epub output:
  <Enter>

::

  > Do you want to use the epub builder (y/n) [n]:
  <Enter>

::

  > autodoc: automatically insert docstrings from modules (y/n) [n]:
  y

::

  > doctest: automatically test code snippets (y/n) [n]:
  y

::

  > intersphinx: link between Sphinx documentation of different projects (y/n) [n]:
  <Enter>

::

  > todo: write "todo" entries that can be shown or hidden on build (y/n) [n]:
  <Enter>

::

  > coverage: checks for documentation coverage (y/n) [n]:
  y

::

  > imgmath: include math, rendered as PNG or SVG images (y/n) [n]:
  <Enter>

::

  > mathjax: include math, rendered in the browser by MathJax (y/n) [n]:
  y

::

  > ifconfig: conditional inclusion of content based on config values (y/n) [n]:
  y

::

  > viewcode: include links to the source code of documented Python objects (y/n) [n]:
  <Enter>

::

  > githubpages: create .nojekyll file to publish the document on GitHub pages (y/n) [n]:
  y

::

  > Create Makefile? (y/n) [y]:
  <Enter>

::

  > Create Windows command file? (y/n) [y]:
  <Enter>

::

  Creating file ./source/conf.py.
  Creating file ./source/index.rst.
  Creating file ./Makefile.
  Creating file ./make.bat

  Finished: An initial directory structure has been created.

Confirm you can see the initial directory structure
---------------------------------------------------

::

  /Documents/
  | ---------------/build/
  |----------------Makefile
  |----------------make.bat
  |---------------/source/
                  |------conf.py
                  |------index.rst
                  |------/_static/
                  |------/_templates/

Install and configure Atom to edit ``.md``
------------------------------------------

::

  $ sudo add-apt-repository ppa:webupd8team/atom
  $ sudo apt update
  $ sudo apt install atom

Enter ``Y`` when prompted: ``After this operation, 112MB of additional disk will be used. Do you want to continue? [Y/n]``.

.. note::

   This step to install and unpack Atom may take 5 minutes to complete.

Install Atom support for ``.md``
--------------------------------

Select **Edit > Preferences > Install**. Search for and install ``language-markdown``.

``markdown-preview`` installed by default gives an preview with **<Ctrl><Shift>M**.

Edit ``index.rst`` in Atom to add ``.md`` files
-----------------------------------------------

::

  .. toctree::
     :glob:
     :maxdepth: 2

      <your_document>

.. note::

   Don't add an ``.md`` suffix for ``.md`` files added to ``index.rst``.

   Put a blank line after ``:maxdepth: 2`` before ``<your_document>``.

Add lines at the start of ``conf.py``
-------------------------------------

::

  import recommonmark
  from recommonmark.transform import AutoStructify
  from recommonmark.parser import CommonMarkParser
  source_parsers - {
     '.md': CommonMarkParser
  }

  import sphinx_bootstrap_theme

Add these lines at the end of ``conf.py``
-----------------------------------------

::

  def setup(app):
      app.add_config_value('recommonmark_config', {
              'enable_math': True,
              'enable_eval_rst': True,
              'enable_auto_doc_ref': True,
              'auto_code_block': True,
              }, True)
      app.add_transform(AutoStructify)

Find and replace ``html_theme`` in ``conf.py``
----------------------------------------------

::

  html_theme - 'bootstrap'
  html_theme_path - sphinx_bootstrap_theme.get_html_theme_path()

  html_theme_options - {
      'bootswatch_theme': "darkly",
      'bootstrap_version': "3",
  }

Add ``sphinx_markdown_tables`` in ``conf.py``
---------------------------------------------

::

  extensions = [
      'sphinx.ext.autodoc',
      'sphinx.ext.doctest',
      'sphinx.ext.coverage',
      'sphinx.ext.mathjax',
      'sphinx.ext.ifconfig',
      'sphinx.ext.githubpages',
      'sphinx_markdown_tables']


Find and replace ``source_suffix`` in ``conf.py``
-------------------------------------------------

::

  source_suffix - ['.rst', '.md']

Add ``html_context`` in ``conf.py`` for ``custom.css`` in ``source/_static`` folders
------------------------------------------------------------------------------------

::

  html_context = {
      'css_files': {'_static/custom.css'}

Make ``custom.css`` from ``bootstrap.css`` and copy to ``source/_static`` folders
---------------------------------------------------------------------------------

- Download a suitable ``bootstrap.css`` for the Bootswatch theme (https://www.bootswatch.com/3/) ``darkly``, which you will edit and rename to make the ``custom.css`` file.

- Edit a downloaded ``bootstrap.css`` to save a ``custom.css``, changing text font and background color options with hexadecimal color codes (https://www.htmlcolorcodes.com).

- Copy a saved ``custom.css`` into the ``source/_static`` folder for each of the guides you need to apply overrides to ``bootstrap-sphinx.css``, which is applied by default.

- Use the broswer inspector to find hex color values applied to the output HTML from ``bootstrap.css`` that you then modify in ``custom.css`` to override default CSS colors.

.. note::

   You must download a ``Bootswatch 3.3.7`` CSS to edit for a ``custom.css`` from https://bootswatch.com/3/

   The default Bootswatch URL (https://bootswatch/com) will download ``Bootswatch 4.1.3`` that won't work.

Update ``Makefile`` to run ``pandoc`` from ``make latex``
---------------------------------------------------------


In ``Makefile`` find:

::

  %:Makefile
    @$(SPHINXBUILD) -M $@ "$(SOURCEDIR)" "$(BUILDDIR)" $(SPHINXOPTS) $(0)

and replace it with:

::

  html:
    @$(SPHINXBUILD) -M $@ "$(SOURCEDIR)" "$(BUILDDIR)" $(SPHINXOPTS) $(0)

  latexpdf:
    @$(SPHINXBUILD) -M $@ "$(SOURCEDIR)" "$(BUILDDIR)" $(SPHINXOPTS) $(0)

  latex:
    @pandoc --toc --latex-engine=pdflatex -V fontfamily="cmbright" -V geometry=margin=0.5in -V documentclass="book" -o /build/<output>.pdf -s <input_1>.md <input_N>.md <copyright>.md

.. note::

   Creating ``latexpdf:``, ``html:``, and ``latex:`` entries in ``Makefile`` enables you to output PDF and HTML from ``.md`` and ``.rst``.

   ``make html`` will call ``html:`` in ``Makefile`` to output HTML from ``.md`` and ``.rst``.

   ``make latexpdf`` will call ``latexpdf:`` in ``Makefile`` to output PDF from ``.rst`` only.

   ``make latex`` will call ``latex:`` in ``Makefile`` to run ``pandoc`` to make PDF from ``.md`` only.

   ``<output>.pdf`` is the filename for the PDF output file with a ``.pdf`` suffix.

   ``<input_1>.md`` is the filename for the first Markdown file in a series of content files for a single book with an ``.md`` suffix.

   ``<input_N>.md`` is the filename for the last Markdown file in a series of content files for a single book with an ``.md`` suffix.

   ``<copyright>.md`` is the filename for the Markdown source file required for the copyright notice to match ``index.rst``.

   ``pandoc`` will combine multiple Markdown source files into a single PDF file so each Markdown file can be a chapter in a book.

   ``fontfamily`` specifies the ``texlive`` font for the main font. This is a sans-serif font that doesn't affect monospaced fonts in code blocks.

   ``margin`` specifies the margin width in inches. 0.5 inch margins will print on most printers. 0.25 inch margins may cut off on some printers.

   ``documentclass`` specifies the format applied by ``pandoc``. ``"book"`` is applied because it follows a book structure used with ``fncychp``.


Update your initial ``.md`` file with cover information
-------------------------------------------------------

Add the below cover information at the start of your initial ``.md`` file:

::

  ---
  title: <document-title>
  author: <author>
  date: <current-date>
  output: pdf_document
  ---

``pandoc`` will put this information on a cover page in the ``.pdf`` output.

Add an ``.md`` copyright file for ``pandoc`` to include
--------------------------------------------------------

Make an ``.md`` copyright notice for ``pandoc`` to append:

::

  # Copyright Notice

  This work is copyrighted. Apart from any use as permitted under the
  Copyright Act 1968, information contained within this manual cannot be
  used for any other purpose other than the purpose for which it was
  released. No part of this publication may be reproduced, stored in a
  retrieval system, or transmitted in any form or by any means,
  electronic, mechanical, photocopying, recording or otherwise, without
  the written permission.

  Words mentioned in this book that are known to be trademarks, whether
  registered or unregistered, have been capitalised or use initial
  capitals. Terms identified as trademarks include Cisco®, Microsoft®,
  Microsoft Windows®, Apple®, AirPort®, Mac®, Linksys®, Symantec®.

``index.rst`` includes a copyright notice for the Sphinx PDF output from ``.rst`` and the Sphinx HTML output from ``.rst`` and ``.md``.
However, ``pandoc`` does not use ``index.rst`` to specify ``.md`` chapters to make a PDF book so needs a separate copyright ``.md`` file.

Make HTML from ``.md`` and ``.rst``
-----------------------------------

::

  /Documents$ make html

View HTML output in ``/Documents/build/html``

Make PDF from ``.md``
---------------------

::

  /Documents$ make latex

View PDF output in ``/Documents/build/``

Make PDF from ``.rst``
----------------------

::

  /Documents$ make latexpdf

View PDF output in ``/Documents/build/latex``
