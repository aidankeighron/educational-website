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

### Your Task: Combine All Markdown Pages

Follow these steps to build the final syllabus content:

1. **Store the OCR response**

   You should already have a variable that holds the full response from your OCR request. If not, make sure you're calling the correct function to get that data.

2. **Create a variable to store all the text**

   Start with an empty string. This will hold the full Markdown content once you're done.

3. **Loop through each page**

   Use a `for...of` loop to go through the `pages` array in the response.

4. **Inside the loop, access the `markdown` field of each page**

   Each page object contains a `markdown` property — that's the extracted content from that page.

5. **Append each `markdown` snippet to your string**

   Add each page’s Markdown to your full text variable. Make sure to include a space or newline between pages so they don’t get mashed together.

6. **(Optional) Print the final Markdown**

   Once your loop is done, use `console.log()` to print the final result and make sure it looks correct.

>  **Why are we doing this?**
> 
> By combining all the page content into one Markdown string, we can pass it to an AI model in a single prompt and ask it to extract assignments for us — much easier than handling one page at a time!
{: .prompt-info }

Next, we’ll send that full Markdown string to an AI to find and extract a list of assignments.

