================
copyright-header
================

--------------------------
Generate copyright headers
--------------------------

:Author: Nicolas Bourdaud <nicolas.bourdaud@gmail.com>
:Date: 2021-03-14
:Manual section: 1

SYNOPSIS
========

**copyright-header**

DESCRIPTION
===========

**copyright-autoheader** is a script that expand a set of tag in source code files
tracked by a git with a configurable content. Typically this is used to expand
the header of a file with the license and copyright reflecting accurately the
years of modification based on the information contained in the git repository.

When executed, an autoheader.yml file will be searched in the top directory of the
git repo. It is composed of 2 entries: 'headers' and 'owner_remap'.

The 'headers' entry:
--------------------
    The headers entry must be composed of a dictionary. Each key correspond to
    a token that can be recognized and expanded in any tracked source file. The
    expanded data will be the value of the dictionary. If a <copyrights> token
    is present in the associated value, it will be itself expanded to the
    copyright line reflecting the years of modification and the copyright owner
    of the file based on the data contained in the git repository (the author
    name and date of commits touching the file).

The 'owner_remap' entry:
------------------------
    This must be composed of list of 2-elements dictionaries with 'pattern' and
    'name' fields. During the copyright expansion described previously, if one
    commit author email match the regular expression in 'pattern' field, the
    copyright owner will be substituted with the value of field 'name'.


EXAMPLE
=======

autoheader.yml
--------------
.. code-block::

   headers:
     "@project_header@": |
       <copyrights>
   
       This program is free software: you can redistribute it and/or modify
       it under the terms of the GNU General Public License as published by
       the Free Software Foundation, either version 3 of the License, or
       (at your option) any later version.
   
       This program is distributed in the hope that it will be useful,
       but WITHOUT ANY WARRANTY; without even the implied warranty of
       MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
       GNU General Public License for more details.
   
       You should have received a copy of the GNU General Public License
       along with this program.  If not, see <https://www.gnu.org/licenses/>.
   owner_remap:
     - pattern: .*@mindmaze.(ch|com)
       name: MindMaze Holding SA
     - pattern: .*@alice-bob.com
       name: Alice & Bob SAS


src/file.c
----------
.. code-block:: c

   /* @project_header@ */
   #include <stdio.h>

   int main(int argc, char* argv[])
   {
   ...
   }


src/launcher
------------
.. code-block:: sh

   #!/bin/sh
   # @project_header@

   func() {
      arg1=$1
      arg2=$2
      ...
   }

   [ $# -eq 3 ] && func $3 $2
   ...


DESIGN RATIONALE
================

If a project is tracked by git, we have an precise tracking of who have
contributed to what. Also from a legal point of you, when a project shared
through git (or any other VCS), copyright header is useless and cumbersome.
However when a release tarball is produced and project is installed, the link
with the versioning system is lost. Hence there is a need to generate the
release tarball from the versioning system.

Why not updating the copyright header in each commit?
-----------------------------------------------------

If done manually, this is very tedious and it is more information to check
during review of each commit.

One can use its IDE to update, but configuration can be very tricky and IDE
dependent. Hence one cannot assume that this is done properly by each
contributor.

No matter the way it is updated (with the help of a tool or manually), doing
copyright/authorship update at each commit generates conflicts unnecessarily
when you merge or rebase those commits.


Why expanding a token and not prepending directly the copyright header?
-----------------------------------------------------------------------
Unfortunately, it is often not possible. Typically when a project imports code
from other projects, if the license is compatible, you can change the license
(you distribute over the code the new compatible license). However, it would be
illegal to claim the copyright over the imported code.

While regularly overlooked in projects, sometimes, copyright header must be
treated differently per file. With DVCS, you can trace back to the actual
proper copyright, but only if you are aware/remember that some file must be
treated differently than the others... If all files are written without header,
you will quickly forget that you cannot change the copyright, that you also
have restrictions about some possible license change.

But there is another practical reason justifying the token expansion. Imagine
you have a project, just for example, that mixes C and python code, and meson,
and Makefile, and julia, and rust, and bash, vim scripts, and scripts with
shebang or even worse windows cmd batch file... How do you generate properly
commented header without messing the code. Should you use '/\* ... \*/', '//',
'#', '#' after the initial line, '"' (vim scripts comments), 'rem' ... If you
provide a file specific template that is expanded, you have no issue, the
source code tells you exactly how and where the copyright notice should appear.
