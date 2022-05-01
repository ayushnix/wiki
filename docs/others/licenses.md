---
title: "Copyright Licenses"
summary: "A list of licenses that I've found suitable for usage"
date: 2022-04-28
---

Just like [fonts][4], choosing an appropriate license for an original work might be confusing and
you may end up wasting time going through [SPDX][1], [GNU][2], [Open Source Initiative][3], and lots
of [heated opinions][5] online. If you're contributing to a project, there's not much to think about
a license — you'll use the one that's already being used. If you're creating something original but
within an ecosystem that usually sticks to a particular license, it's best to stick to that license.

This page is meant for choosing a license for an original work. Of course, this page represents my
personal views and IANAL. Any licenses that I haven't mentioned here are probably either too obscure
to be worth using, restricted within their own niche, too complex, controversial, or not suitable
for usage by the general public. It's also likely that I may not have heard of it. The ISC license
deserves a special mention because even though I like it a lot, it is surrounded by confusion about
[using `and/or`][21] or [not][22]. I'm not interested in debating about `and/or` from a lawyer's
perspective.

I've used SPDX license identifiers in the headers so finding the source of the license shouldn't be
difficult.

# Software and Source Code

## MIT

This is probably the most popular open source license in use on the planet and functionally
equivalent to the ISC license while being more verbose but without any confusion. You still receive
attribution, just like ISC.

One of the distinguishing features of the MIT license is that it also includes software
documentation under the same copyright which can make things convenient for a project. The source
code of the documentation website and the documentation itself can be released under the same MIT
license.

??? abstract "MIT"

    ```
    Copyright (c) [YYYY] [YOUR NAME HERE] [user@your.email]

    Permission is hereby granted, free of charge, to any person obtaining a copy
    of this software and associated documentation files (the "Software"), to deal
    in the Software without restriction, including without limitation the rights
    to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
    copies of the Software, and to permit persons to whom the Software is
    furnished to do so, subject to the following conditions:

    The above copyright notice and this permission notice shall be included in all
    copies or substantial portions of the Software.

    THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
    IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
    FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
    AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
    LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
    OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
    SOFTWARE.
    ```

## BSD-2-Clause

A somewhat explicit but venerable license that is equivalent to MIT and ISC.

??? abstract "BSD-2-Clause"

    ```
    Copyright (c) [YYYY] [YOUR NAME HERE] [user@your.email]

    Redistribution and use in source and binary forms, with or without
    modification, are permitted provided that the following conditions are met:

    1. Redistributions of source code must retain the above copyright notice, this
       list of conditions and the following disclaimer.

    2. Redistributions in binary form must reproduce the above copyright notice,
       this list of conditions and the following disclaimer in the documentation
       and/or other materials provided with the distribution.

    THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
    AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
    IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
    DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
    FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
    DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
    SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
    CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
    OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
    OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
    ```

## BSD-3-Clause

The 3-clause version the BSD license family adds another clause about restricting endorsement from
you in derivative works. In case you're confused, it means that someone can't take your BSD-3-Clause
licensed shit, distribute a derivative and proclaim that that derivative was made by you or is
endorsed by you. They can, and should, still add your copyright notice to their derivative work
being distributed.

??? abstract "BSD-3-Clause"

    ```
    Copyright (c) [YYYY] [YOUR NAME HERE] [user@your.email]

    Redistribution and use in source and binary forms, with or without
    modification, are permitted provided that the following conditions are met:

    1. Redistributions of source code must retain the above copyright notice, this
       list of conditions and the following disclaimer.

    2. Redistributions in binary form must reproduce the above copyright notice,
       this list of conditions and the following disclaimer in the documentation
       and/or other materials provided with the distribution.

    3. Neither the name of the copyright holder nor the names of its
       contributors may be used to endorse or promote products derived from
       this software without specific prior written permission.

    THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
    AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
    IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
    DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
    FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
    DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
    SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
    CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
    OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
    OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
    ```

## NCSA

This is based on the MIT and the BSD-3-Clause licenses. The documentation advantage that MIT has
applies to NCSA as well. The endorsement caveat from BSD-3-Clause applies as well. If I'm reading
things right, this license sounds better than BSD-3-Clause so if you're considering using that,
might as well use the NCSA license.

??? abstract "NCSA"

    ```
    Copyright (c) [YYYY] [YOUR NAME HERE] [user@your.email]

    Developed by: [project]
                  [fullname]
                  [projecturl]

    Permission is hereby granted, free of charge, to any person
    obtaining a copy of this software and associated documentation files
    (the "Software"), to deal with the Software without restriction,
    including without limitation the rights to use, copy, modify, merge,
    publish, distribute, sublicense, and/or sell copies of the Software,
    and to permit persons to whom the Software is furnished to do so,
    subject to the following conditions:

    * Redistributions of source code must retain the above copyright notice,
      this list of conditions and the following disclaimers.

    * Redistributions in binary form must reproduce the above copyright
      notice, this list of conditions and the following disclaimers in the
      documentation and/or other materials provided with the distribution.

    * Neither the names of [fullname], [project] nor the names of its
      contributors may be used to endorse or promote products derived from
      this Software without specific prior written permission.

    THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS
    OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
    FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
    CONTRIBUTORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
    LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
    OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS WITH
    THE SOFTWARE.
    ```

## Apache-2.0

This is widely used by many professional open source projects and is also recommended by the FSF as
the lax license one should use. I, however, don't claim to understand it. It is longer than CC0-1.0
as well. Oh, it's incompatible with GPL-2.0-only which is something that should be kept in mind when
using it.

I won't include this license in this page. It's too long and meant for lawyers, not for mere
mortals. [The source for Apache-2.0][7] is, however, mentioned.

## LGPL-2.1-only

It's meant to be a weak copyleft license to be used for software libraries, such as notcurses or
tui-rs (neither of them are licensed under LGPL). A derivative library has to be released under the
same license and programs that link to a LGPL-2.1-only library can be released however the author
pleases.

I won't include this license in this page. It's too long and meant for lawyers, not for mere
mortals. [The source for LGPL-2.1-only][8] is, however, mentioned.

## MPL-2.0

This seems to be somewhat similar to the Apache-2.0 license but it adopts a weak copyleft style
wherein the original licensed source must be released under MPL-2.0 while the additional derivative
work may be distributed under any license. Mozilla Firefox is the probably the most popular software
released under this license. Syncthing is another good example.

I won't include this license in this page. It's too long and meant for lawyers, not for mere
mortals. [The source for MPL-2.0][9] is, however, mentioned.

## GPL-2.0-only

The most popular copyleft license used by the Linux kernel itself. A lot of the Linux ecosystem uses
this license. It is also a strong copyleft license and any derivate work should be licensed under
the same terms, at least in theory.

I won't include this license in this page. It's too long and meant for lawyers, not for mere
mortals. [The source for GPL-2.0-only][10] is, however, mentioned.

# Documentation

As mentioned before in [MIT][11] and [NCSA][12], in addition to the licenses mentioned in this
section, they can be used for documentation as well. Yes, GFDL-1.3-only and its variants aren't
included.

## FreeBSD-DOC

A subtle variant of the BSD-2-Clause license written for FreeBSD documentation and its man pages.
The source and compiled forms can be changed as needed.

??? abstract "FreeBSD-DOC"

    ```
    Copyright (c) [YYYY] [YOUR NAME HERE] [user@your.email]

    Redistribution and use in source (SGML DocBook) and 'compiled' forms (SGML,
    HTML, PDF, PostScript, RTF and so forth) with or without modification, are
    permitted provided that the following conditions are met:

        1. Redistributions of source code (SGML DocBook) must retain the above
           copyright notice, this list of conditions and the following disclaimer as
           the first lines of this file unmodified.
        2. Redistributions in compiled form (transformed to other DTDs, converted to
           PDF, PostScript, RTF and other formats) must reproduce the above
           copyright notice, this list of conditions and the following disclaimer in
           the documentation and/or other materials provided with the distribution.

    THIS DOCUMENTATION IS PROVIDED BY THE FREEBSD DOCUMENTATION PROJECT "AS IS" AND
    ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
    WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
    DISCLAIMED. IN NO EVENT SHALL THE FREEBSD DOCUMENTATION PROJECT BE LIABLE FOR
    ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES
    (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES;
    LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON
    ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
    (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
    DOCUMENTATION, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
    ```

## CC-BY-4.0

This is probably equivalent to the BSD-3-Clause license but it can be used for software
documentation, unlike BSD-3-License.

I won't include this license in this page. It's too long and meant for lawyers, not for mere
mortals. [The source for CC-BY-4.0][15] is, however, mentioned.

## CC-BY-SA-4.0

CC-BY-4.0 license with strong copyleft added in. Any derivatives should be CC-BY-SA-4.0 as well.

I won't include this license in this page. It's too long and meant for lawyers, not for mere
mortals. [The source for CC-BY-SA-4.0][16] is, however, mentioned.

# Opinions, Blog Posts, Microblog Posts

In addition to the licenses mentioned in this section, [CC-BY-4.0][13] and [CC-BY-SA-4.0][14] can be
used as well if you don't care about your opinions being used to distribute derivative works without
your consent or being sold online.

No, there's no such thing as CC-BY-NC-ND-SA because ND and SA are contradictory.

## CC-BY-ND-4.0

CC-BY-4.0 but you can't distribute derivative works.

I won't include this license in this page. It's too long and meant for lawyers, not for mere
mortals. [The source for CC-BY-ND-4.0][17] is, however, mentioned.

## CC-BY-NC-4.0

CC-BY-4.0 but you can't sell derivative works.

I won't include this license in this page. It's too long and meant for lawyers, not for mere
mortals. [The source for CC-BY-NC-4.0][18] is, however, mentioned.

## CC-BY-NC-ND-4.0

CC-BY-4.0 but you can neither sell nor distribute derivative works.

I won't include this license in this page. It's too long and meant for lawyers, not for mere
mortals. [The source for CC-BY-NC-ND-4.0][19] is, however, mentioned.

## CC-BY-NC-SA-4.0

CC-BY-4.0 but you can't sell derivative works. In addition, strong copyleft comes in and compels you
to distribute any derivative works under CC-BY-NC-SA-4.0.

I won't include this license in this page. It's too long and meant for lawyers, not for mere
mortals. [The source for CC-BY-NC-SA-4.0][20] is, however, mentioned.

# Public Domain Licenses

If you want your work to be used in the most unrestricted manner and free from any copyright jargon,
you want to forfeit your copyright and intellectual rights and dedicate it to the public domain.

## 0BSD

This is probably the simplest license you can choose to dedicate your *software* to the public
domain. It doesn't even require any sort of attribution and is best suited for small programs,
scripts, or snippets of code written casually on a blog.

??? abstract "0BSD"

    ```
    Copyright (c) [YYYY] [YOUR NAME HERE] [user@your.email]

    Permission to use, copy, modify, and/or distribute this software for any
    purpose with or without fee is hereby granted.

    THE SOFTWARE IS PROVIDED "AS IS" AND THE AUTHOR DISCLAIMS ALL WARRANTIES
    WITH REGARD TO THIS SOFTWARE INCLUDING ALL IMPLIED WARRANTIES OF
    MERCHANTABILITY AND FITNESS. IN NO EVENT SHALL THE AUTHOR BE LIABLE FOR ANY
    SPECIAL, DIRECT, INDIRECT, OR CONSEQUENTIAL DAMAGES OR ANY DAMAGES
    WHATSOEVER RESULTING FROM LOSS OF USE, DATA OR PROFITS, WHETHER IN AN ACTION
    OF CONTRACT, NEGLIGENCE OR OTHER TORTIOUS ACTION, ARISING OUT OF OR IN
    CONNECTION WITH THE USE OR PERFORMANCE OF THIS SOFTWARE.
    ```

## CC0-1.0

This is the lawyer friendly version of the public domain license although it should still be
comprehensible by mere mortals, albiet with some considerable effort, unlike other creative commons
licenses. It can be used for both source code and content.

This is the only CC license which has its text included on this page.

??? abstract "CC0-1.0"

    ```
    Creative Commons Legal Code

    CC0 1.0 Universal

        CREATIVE COMMONS CORPORATION IS NOT A LAW FIRM AND DOES NOT PROVIDE
        LEGAL SERVICES. DISTRIBUTION OF THIS DOCUMENT DOES NOT CREATE AN
        ATTORNEY-CLIENT RELATIONSHIP. CREATIVE COMMONS PROVIDES THIS
        INFORMATION ON AN "AS-IS" BASIS. CREATIVE COMMONS MAKES NO WARRANTIES
        REGARDING THE USE OF THIS DOCUMENT OR THE INFORMATION OR WORKS
        PROVIDED HEREUNDER, AND DISCLAIMS LIABILITY FOR DAMAGES RESULTING FROM
        THE USE OF THIS DOCUMENT OR THE INFORMATION OR WORKS PROVIDED
        HEREUNDER.

    Statement of Purpose

    The laws of most jurisdictions throughout the world automatically confer
    exclusive Copyright and Related Rights (defined below) upon the creator
    and subsequent owner(s) (each and all, an "owner") of an original work of
    authorship and/or a database (each, a "Work").

    Certain owners wish to permanently relinquish those rights to a Work for
    the purpose of contributing to a commons of creative, cultural and
    scientific works ("Commons") that the public can reliably and without fear
    of later claims of infringement build upon, modify, incorporate in other
    works, reuse and redistribute as freely as possible in any form whatsoever
    and for any purposes, including without limitation commercial purposes.
    These owners may contribute to the Commons to promote the ideal of a free
    culture and the further production of creative, cultural and scientific
    works, or to gain reputation or greater distribution for their Work in
    part through the use and efforts of others.

    For these and/or other purposes and motivations, and without any
    expectation of additional consideration or compensation, the person
    associating CC0 with a Work (the "Affirmer"), to the extent that he or she
    is an owner of Copyright and Related Rights in the Work, voluntarily
    elects to apply CC0 to the Work and publicly distribute the Work under its
    terms, with knowledge of his or her Copyright and Related Rights in the
    Work and the meaning and intended legal effect of CC0 on those rights.

    1. Copyright and Related Rights. A Work made available under CC0 may be
    protected by copyright and related or neighboring rights ("Copyright and
    Related Rights"). Copyright and Related Rights include, but are not
    limited to, the following:

      i. the right to reproduce, adapt, distribute, perform, display,
         communicate, and translate a Work;
     ii. moral rights retained by the original author(s) and/or performer(s);
    iii. publicity and privacy rights pertaining to a person's image or
         likeness depicted in a Work;
     iv. rights protecting against unfair competition in regards to a Work,
         subject to the limitations in paragraph 4(a), below;
      v. rights protecting the extraction, dissemination, use and reuse of data
         in a Work;
     vi. database rights (such as those arising under Directive 96/9/EC of the
         European Parliament and of the Council of 11 March 1996 on the legal
         protection of databases, and under any national implementation
         thereof, including any amended or successor version of such
         directive); and
    vii. other similar, equivalent or corresponding rights throughout the
         world based on applicable law or treaty, and any national
         implementations thereof.

    2. Waiver. To the greatest extent permitted by, but not in contravention
    of, applicable law, Affirmer hereby overtly, fully, permanently,
    irrevocably and unconditionally waives, abandons, and surrenders all of
    Affirmer's Copyright and Related Rights and associated claims and causes
    of action, whether now known or unknown (including existing as well as
    future claims and causes of action), in the Work (i) in all territories
    worldwide, (ii) for the maximum duration provided by applicable law or
    treaty (including future time extensions), (iii) in any current or future
    medium and for any number of copies, and (iv) for any purpose whatsoever,
    including without limitation commercial, advertising or promotional
    purposes (the "Waiver"). Affirmer makes the Waiver for the benefit of each
    member of the public at large and to the detriment of Affirmer's heirs and
    successors, fully intending that such Waiver shall not be subject to
    revocation, rescission, cancellation, termination, or any other legal or
    equitable action to disrupt the quiet enjoyment of the Work by the public
    as contemplated by Affirmer's express Statement of Purpose.

    3. Public License Fallback. Should any part of the Waiver for any reason
    be judged legally invalid or ineffective under applicable law, then the
    Waiver shall be preserved to the maximum extent permitted taking into
    account Affirmer's express Statement of Purpose. In addition, to the
    extent the Waiver is so judged Affirmer hereby grants to each affected
    person a royalty-free, non transferable, non sublicensable, non exclusive,
    irrevocable and unconditional license to exercise Affirmer's Copyright and
    Related Rights in the Work (i) in all territories worldwide, (ii) for the
    maximum duration provided by applicable law or treaty (including future
    time extensions), (iii) in any current or future medium and for any number
    of copies, and (iv) for any purpose whatsoever, including without
    limitation commercial, advertising or promotional purposes (the
    "License"). The License shall be deemed effective as of the date CC0 was
    applied by Affirmer to the Work. Should any part of the License for any
    reason be judged legally invalid or ineffective under applicable law, such
    partial invalidity or ineffectiveness shall not invalidate the remainder
    of the License, and in such case Affirmer hereby affirms that he or she
    will not (i) exercise any of his or her remaining Copyright and Related
    Rights in the Work or (ii) assert any associated claims and causes of
    action with respect to the Work, in either case contrary to Affirmer's
    express Statement of Purpose.

    4. Limitations and Disclaimers.

     a. No trademark or patent rights held by Affirmer are waived, abandoned,
        surrendered, licensed or otherwise affected by this document.
     b. Affirmer offers the Work as-is and makes no representations or
        warranties of any kind concerning the Work, express, implied,
        statutory or otherwise, including without limitation warranties of
        title, merchantability, fitness for a particular purpose, non
        infringement, or the absence of latent or other defects, accuracy, or
        the present or absence of errors, whether or not discoverable, all to
        the greatest extent permissible under applicable law.
     c. Affirmer disclaims responsibility for clearing rights of other persons
        that may apply to the Work or any use thereof, including without
        limitation any person's Copyright and Related Rights in the Work.
        Further, Affirmer disclaims responsibility for obtaining any necessary
        consents, permissions or other rights required for any use of the
        Work.
     d. Affirmer understands and acknowledges that Creative Commons is not a
        party to this document and has no duty or obligation with respect to
        this CC0 or use of the Work.
    ```

[1]: https://spdx.org/licenses/
[2]: https://www.gnu.org/licenses/license-list.html
[3]: https://opensource.org/licenses/
[4]: fonts.md
[5]: https://marc.info/?l=openbsd-misc&m=120618313520730&w=2
[6]: https://www.isc.org/licenses/
[7]: https://www.apache.org/licenses/LICENSE-2.0.txt
[8]: https://www.gnu.org/licenses/old-licenses/lgpl-2.1-standalone.html
[9]: https://www.mozilla.org/en-US/MPL/2.0/
[10]: https://www.gnu.org/licenses/old-licenses/gpl-2.0-standalone.html
[11]: #mit
[12]: #ncsa
[13]: #cc-by-40
[14]: #cc-by-sa-40
[15]: https://creativecommons.org/licenses/by/4.0/legalcode.txt
[16]: https://creativecommons.org/licenses/by-sa/4.0/legalcode.txt
[17]: https://creativecommons.org/licenses/by-nd/4.0/legalcode.txt
[18]: https://creativecommons.org/licenses/by-nc/4.0/legalcode.txt
[19]: https://creativecommons.org/licenses/by-nc-nd/4.0/legalcode.txt
[20]: https://creativecommons.org/licenses/by-nc-sa/4.0/legalcode.txt
[21]: https://groups.google.com/g/comp.protocols.dns.bind/c/D8VLXloVfxc/m/4yweyAjRCecJ
[22]: https://marc.info/?l=openbsd-misc&m=120618313520730&w=2

--8<-- "include/abbreviations.md"
