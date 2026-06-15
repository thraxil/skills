---
name: readable
description: make a readable version of a web page
---

# Readable

Use this skill to make a readable local version of a web page,
stripping out ads and site-specific navigation.

## Core Prompt

Read the URL supplied and produce a local HTML version that strips out
all ads, site navigation, and things not relevant to the content. Use
simple semantic HTML and clean, minimal styles, focusing on being easy
to read.

## Naming

Use a meaningful but relatively short name for the resulting file
based on the article title (or an implied title).

## Resources

If there are images relevant to the content (but NOT ads or site
chrome), download them to a local directory with the same name as the
html file (minus the extension) with meaningful filenames and link to
them in the HTML file.

If there are other key resources in the article (eg, javascript, video
files, pdfs, etc) ask the user how they would like them handled.

## Metadata

As a footer, include the original URL and the current date (in
YYYY-MM-DD format).
