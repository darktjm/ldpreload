Linux Preload Library Support
=============================

Thomas J. Moore

**Revision 199**

Abstract
--------
This document was extracted from ldpreload.html.  Read it for details.
This project serves three purposes:

  * To provide an LD_PRELOAD psuedo-filesystem to provide random data
    (which is better for bandwidth and storage testing than the commonly
    used all-zero data, because it is hard to compress) and verify it as
    well (again, all-zero data is very bad at detecting data integrity
    errors).  It uses a very bad set of random numbers that are
    probably too large for most dictionary compressors, but very quickly
    generated so that the random number generation time does not really
    affect performance tests.

  * To demonstrate my literate build system in action: how to use, and
    how to extend.

  * To provide a simple method to create LD_PRELOAD overrides of C
    library functions.


> This document describes and implements the support required for simple
> Linux preload libraries, in particular for emulating a filesystem in
> user space. It also includes a sample application: an extremely
> simplistic filesystem which returns random data on reads and verifies
> that data on writes.
>
> © 2008 Trustees of Indiana University. This document is licensed under
> the Apache License, Version 2.0 (the \`License'); you may not use this
> document except in compliance with the License. You may obtain a copy
> of the License at
> [http://www.apache.org/licenses/LICENSE-2.0](http://www.apache.org/licenses/LICENSE-2.0).
> Unless required by applicable law or agreed to in writing, software
> distributed under the License is distributed on an \`AS IS' BASIS,
> WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or
> implied. See the License for the specific language governing
> permissions and limitations under the License.
>
> Small improvements have been made since by the author, on his own
> time. These changes are available under the same conditions.
>

Usage
-----

To use the dynamic standard C library override support, add a dependency
to this document. It is possible to use the scripts outside of the build
system, as well. To do this, place `addldovr.sed` and `delldovr.sed` in
appropriate places, and apply `addldovr.sed` before building, and
`delldovr.sed` before editing. In either case, add the code in
`<Dynamic Override C Includes>` to the top of your source; for outside
the build system, the `<(build.nw) Common C Includes>` can probably be
replaced by whatever the code depends on.

For each overridden function, add an override comment, followed by a
blank line, as shown in `<Example open` prototype\>. Then, create the
override function with the same prototype. At the top of this function,
an initialization function should be called which executes the code in
`<Dynamic Override Init>`, which in turn depends on static variables
defined in the same file, provided by `<Dynamic Override Globals>`. The
initialization code uses a static guard variable to execute exactly
once, and assumes that it is returning from a `void` function.

To use the example random number filesystem, simply add it to
`LD_PRELOAD`, and optionally set the `RANDFS_PATH_PATTERN` environment
variable to change the regular expression used to match file names in
the virtual random filesystem from the default of `^/rand/`. The base
filename determines both the random number seed and the size. The size
is determined by a numeric prefix, optionally followed by an upper-case
K, M, G, or T to multiply by powers of 1024. If no size can be
determined from the file name, the file size is 100MB.

