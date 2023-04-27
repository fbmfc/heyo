# Compliance training subjects

When people are talking about "open source compliance" as a generic term they
are usually not aware that it is actually not a single problem, but there are
several hiding under that umbrella term.

Usually one of the following problems need to be solved:

1. license text determination/extraction + copyright/author statement extraction
2. provenance/copied code detection
3. software composition analysis/dependency analysis/linking
4. keeping track of what is used with a component catalog
5. maturity
5. security

There are several tools on the market, both commercial offerings and open source
offerings, that can help solve these problems or at least assist with finding
the information needed to solve the problems.

To be as effective as possible it is best to first discuss the problems in
depth and discuss which tools can assist solving these problems.

## License text determination/extraction + copyright/author statement extraction

Source code archives and files typically contain license texts, license
references, copyright statements and author statements. Finding out under which
licenses code was released is important as various licenses have various
obligations, such as the requirement to pass on source code, granting patent
licenses, attribution, and so on.

This means that you need to detect which licenses are used. For this typically
a license scanner is used. License scanners typically scan for two things:

1. license texts
2. license references

License texts are complete license texts, listing all the terms and conditions.
License references on the other hand are short statements that indicate which
license is used, as usually it is impractical to include the whole license text
in a source code file (exceptions: MIT and BSD, where the license text tends to
be so short that it is usually used instead of a license reference). Sometimes
there is a pointer to where the actual license text can be found, at other
times it is very compact (for example only an SPDX identifier). Although there
are templates for license references that people are encouraged to use there
are many different variations, complicating the task of the scanner.

Depending on the license scanner it might also be that copyright and author
statements can be extracted. The quality of the scanners differs: some will
only extract very clear copyright notices (missing less obvious ones), while
others will try to be much more thorough, but at the cost of having some false
positives.

Example tools are:

* FOSSology (open source)
* Scancode (open source)
* Black Duck
* FossID
* Mend.io (formerly Whitesource)

where it should be noted that Scancode is becoming somewhat of an industry
standard (and is increasingly being used by other tools as well).

## Provenance detection for source code

Provenance detection can basically be summed up as "where did the code I am
scanning come from?". A typical use case is detecting copied code (known as
"code clone detection" in academia and often referred to as "snippet matching"
by tool vendors). The prime motivator for development of tools that can do this
is to find plagiarised code, or to find if developers had accidentily (or
sneakily) copied open source code into proprietary programs (putting IP at
risk by "contamination" by open source).

Most tool vendors implement this by chopping source code files into small parts
(snippets), hashing these snippets to get some sort of fingerprint and then
comparing the fingerprints to a database of fingerprints generated from known
open source software and then report what was found, in which file it was found
and to what package it belongs, along with any other associated metadata
(license, known security issues, and so on), and typically allow the user to do
a manual review of the results (known as "clearing").

Tools that implement this functionality:

* FossID
* Black Duck
* ScanOSS (open source, but also offering premium services)

At the moment there are no good open source snippet scanners that have the same
level of maturity as the commercial scanners. ScanOSS is open source and has
opened the scripts to create and maintain an open source knowledgebase, but
this is a non-trivial undertaking.

There are plans underway for a new open source/open data snippet scanner called
"Matchcode" (made by the Scancode authors) but this will take some time to
develop.

It should be noted that snippet matching forces the user in a specific work
flow and that it isn't well suited to other approaches of looking at source
code. For example, it is very useful to try to find out which file in a set
of known files is a closest match for the file that is being scanned, for
example to speed up scanning, or to see how much a source code file differs
from a known (open source) origin. With the snippet search algorithms this
is not easy to implement, but with other mechanisms it is[1].

## Provenance detection for binary code

Although ideally software deliverables from third parties should come with all
the source code and build scripts so a rebuild can be attempted this is hardly
ever the case. Often software is shipped in binary only form or in mixed form
(some parts as binary, some parts as source code). If the binary code is passed
on or used in some way that there could be some liabilities it should be
scanned as well. There are relatively few solutions on the market that can do
this and most of them are very expensive.

A good open source solution is Binary Analysis Next Generation (BANG), which is
an advanced framework for recursively unpacking binary files (such as firmware
files) and extracting data to make it available for further analysis and
checks.

Using the YARA engine and YARA rule files (which are a TODO) it is then
possible to accurately find the provenance of the binary data, if source code
is available to generate YARA rule files from.

## Dependency analysis/linking

There are very few programs that run standalone. Most programs need other
components (such as libraries) to work properly. This means that there is an
interaction between licenses of the different components. Licenses might have
conflicting conditions which will make it impossible to legally combine the
programs (although it would be not a problem technically).

Dependencies can be looked at in two ways: source code and binaries (depending
on the language).

In many languages dependency information can be retrieved from the source code
archive[3] (and sometimes from the binary archives as well). By inspecting
these files and cross correlating it with information from online repositories
(such as Maven, NPM, PyPI, and so on) a complete recursive dependency tree
can be retrieved and made available for further analysis or scanning.

When looking at how programs combine at run time then for some programs it
might be better to look at the binary files instead of the source code. An
example of this is the ELF file format, where the names of functions and
variables are stored, along with the names of libraries in which to find the
associated functions and variables. Using a fairly simple approach[2] it is
possible to find out (recursively) which ELF file links to which other file.
This method does not include getting the extra meta information about the
dependencies, which has to be injected into the process at some point: it just
reports which ELF files link with which other ELF files.

For other binary formats (such as Java JARs) this is a less viable approach,
because of the compilation process, polymorphism constructs in the language,
and so on. In other binary files (such as Android Dalvik files like
`classes.dex`) everything (except external system libraries or platform specific
libraries in C) is simply included in a single binary.

Good tools to explore are Open Source Review Toolkit (ORT) for source code
dependency checking and BANG for ELF linking.

## Component catalogue and maturity of components

It is important for an engineering team to know whether or not they have picked
the right components when building a product: if a component that was chosen
turns out to be unmaintained (security bugs, no new features, many open bugs)
then it either means replacing the component (which could be non-trivial) or
maintain the component by yourself (which could also be non-trivial). Having
access to this kind of information is essential when building products that
need to be maintained for quite some time.

Most of the commercial systems offer some of this information (functionality
might differ per vendor) and it might also be good to look at open source
alternatives from the CHAOSS project[4].

Commercial systems to consider are nexB's Dejacode platform.

## Security

Somewhat connected to the code maturity: security is another aspect to consider
when building a product. There are two types of security scanning: reactive
(making sure that there are no known defects in products that are used or
shipped) and proactive (finding previously unknown bugs in code). These are
two distinct problems.

The first one (known bugs in existing code) can be found in a number of ways
and many tools (commercial, but also open source) can report information about
known bugs (as recorded in for example CVE reports), typically at the package
level (and not at for example the file level). Examples are Snyk, Black Duck,
Dejacode, and so on. On the open source side there is Vulnerablecode (which is
powering Dejacode).

Proactively finding bugs requires different tools and often needs some human
verification. Commercial tools that can help in this space are Snyk as well
as a few others (that I am not familiar with). A few open source tools to
mention are Coccinelle[5] (for the Linux kernel), CVEhound[6] (using Coccinelle
to detect known bugs in Linux kernel sources from vendors that are not mentioned
in CVE reports) and semgrep[7] (inspired by Coccinelle) which can run all kinds
of rules, for example for vulnerabilities, on source code and report. There
is a fairly extensive collection of rules available directly from the semgrep
developers but these have been released under a non-free license (LGPL 2.1 with
Commons Clause).


# Summary and recommendations

| Problem to solve  | Recommended tools to use | Notes |
|-------------------|--------------------------|-------|
| License scanning  | Scancode                 |       |
| Author/copyright  | Scancode                 |       |
| Snippet matching  | FossID                   | Until there is a better open source tool |
| Binary matching   | BANG                     | Needs data files|
| Proximity matching | Use Armijn's proximity matching stuff | Bit rough around the edges |
| dependency analysis for ELF | BANG          |       |
| dependency analysis for the rest | ORT      |       |
| component catalog | Dejacode                |       |
| maturity          | CHAOSS project tools    |       |


## Security

| Problem to solve | Recommended tools to use | Notes |
|------------------|--------------------------|-------|
| smells/patterns  | semgrep           | open source, but unfree rule files |
| smells/patterns  | Snyk              | proprietary |
| patterns for Linux kernel | Coccinelle | |
| known CVEs for Linux kernel | CVEhound | |
| finding CVEs and vulnerable packages | VulnerableCode | Rough around the edges |

# Appendix A: questions to ask

A list of some questions that should be asked in any compliance process:

1. Where do we get our code from and who are our suppliers?
2. Do we get the code in source code form, binary form or mixed form?
3. What licenses are associated with the source code that we get?
4. What licenses are associated with the binary code that we get?
5. Do we have permission (apart from the license) from the upstream to
   redistribute, or do contracts prohibit us from redistributing code that
   we need to redistribute (meaning contracts need to be rewritten)?
6. What are we distributing to our downstream?
7. Under what licenses are the components that we are distributing to our
   downstream?
8. How are we combining components?
9. Can we rebuild the code that we are getting from our upstream? Do we have
   proper rebuild instructions as well as the build files?
10. Do we have source code for the binary code that we have received? Can we
    rebuild the source code and verify if it is equivalent to the binary code
    we have received?

For code that was written in house and not coming from a supplier:

1. Is there any code written by freelancers or students? If so, do we have
   a contract in place that gives us either ownership of the code or a license
   to redistribute?
2. Can we maintain the code and do we still have the internal expertise to
   do so?

# Appendix B: recommended workflow

Some recommended workflows.

## License scanning, copyright statement extraction and author statement extraction

Run Scancode on any available source code.

## Binary scanning

BANG

## Dependency scanning for source code

Run ORT.

## Security scanning

Use VulnerableCode to find known bugs in your known open source code (ask
nexB). Run semgrep (using the non-free rules) to find unknown bugs. Write and
publish semgrep rules under a license that actually is free. Optionally run
Snyk to find more vulnerabilities.

## Maturity scanning

Use Dejacode. If you have time run and install the tools from the CHAOSS
project for additional information.

## Snippet matching

For now: use FossID, Black Duck, ScanOSS or any of the other commercial
solutions. Better: help nexB further develop Matchcode and PurlDB.


# References

[1] <https://github.com/armijnhemel/proximity_matcher_webservice>
[2] <https://lwn.net/Articles/548216/>
[3] <https://github.com/oss-review-toolkit/ort#analyzer>
[4] <https://chaoss.community/>
[5] <https://coccinelle.gitlabpages.inria.fr/website/>
[6] <https://github.com/evdenis/cvehound>
[7] <https://github.com/returntocorp/semgrep>
