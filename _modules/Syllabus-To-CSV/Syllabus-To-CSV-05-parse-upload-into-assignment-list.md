---
title: "Parse Upload into Assignment List"
parent_post: Syllabus-To-CSV
module_number: 5
layout: module
media_subpath: /assets/tutorials/csv
---

## Parse upload into assignment list

Now that you’ve received the OCR response from Mistral, it's time to prepare the text for assignment extraction.

The OCR response (`ocrJson`) contains a list of pages — and each page includes a `markdown` field with the text that Mistral pulled from that page.

We want to loop through all those pages and combine the Markdown into one big string we can send to an AI model later.

> **TASK 3:** Loop through `ocrJson.pages` and concatenate the `markdown` property from each page into a single combined Markdown string.
{: .prompt-warning }

> **NOTE:** By combining all page content into one Markdown string, we can pass it to an AI model in a single prompt and ask it to extract assignments for us — much easier than handling one page at a time!
{: .prompt-info }

Next, we’ll send that full Markdown string to an AI to find and extract a list of assignments.

