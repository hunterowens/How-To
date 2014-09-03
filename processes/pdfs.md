---
title: Working with PDFs
layout: default
---

{:toc}

# Introduction

The Portable Document Format (or "PDF") is an excellent way to share documents. Documents consistently render across all devices -- from desktop computers to smart televisions -- in the exact specification the publisher designed. Unfortunately, the advantages which the PDF format gives to publishers of *documents* are mostly the same as the disadvantages which the PDF format gives to publishers of *data*.

## Some Background on PDFs

<!-- TODO: link out this section to the PDF spec. -->

Pulling *data* out of PDF *documents* can be an arduous task. The PDF document format is mostly an image format. Text which is encoded within a PDF document is encoded according to lines which are given X, Y coordinates as to the corners of the block of text (which can be a line, a sidebar, a table, or any other block of text). So rather than having text be associated semantically, tabular, or any other way in which structured *data* should be kept, text is simply given an X, Y coordinate and a blob of text.

PDF documents are essentially an image format. This is why they are so often used when scanning and displaying documents -- and what allows for consistent rendering across devices. As such, text within a PDF is rendered in layers. When a document generating program renders a PDF what happens is that it compiles an image of the page. If there is text which is to be rendered that text is rendered as an additional (hidden) layer over the image. This is why when you often select text from a PDF viewer there can be an offset in the text that is selected and that which the human eye is seeing. More simply PDF viewers show you the image of the page but when you want to select text the PDF selects the hidden layer which contains the text.

Layers are a powerful device for rendering consistent documents but it does not give effect to the placement of text within the page, nor is there any inherent mechanism for semantically linking text together. For example, headers in HTML files (like the text blob above this text which contains the text, "Some Background on PDFs") are noted using an `<h2>` and `</h2>` tag. Headers come in levels so `h2` could be any number between 1 and 9, 2 is only chosen as an example and means second level header. Header tags allow machines to understand that the `h2` is (a) nested underneath the preceding `h1` and (b) the text which follows until the next `h2` or `h1` tag is linked to this particular `h2`. This is powerful when one is trying to parse text. PDF documents do not have such a semantic structuring. The PDF specification, which, again, was meant as a rendering format not a communication format, does not have a semantic system. Whatever program one is using to make a PDF will simply output the `h2` as text which is somehow different in font, or size, or however the document designer has meant for it to be.

Lastly, PDF documents need not consistently render text in any particular order. This is an advantage for working with a highly stylized report because text can be layered and added at different times. Because text within a PDF document only needs a blob of text and the X, Y coordinate for the corners, even when exporting to PDF from an office program or from a web browser one cannot rely upon the ordering of the text in the PDF document to establish the order of the text that will be displayed until one displays the whole page with the X, Y coordinates for each of the blobs of text. If one has ever tried to copy and paste from a PDF one has seen this because when one pastes the text block into another program the end of the lines as rendered by the PDF original are almost always pasted in as new lines into the new program. Lawyers have likely wasted, collectively, thousands of billable hours of client time simply backspacing out the new lines from a block of PDF text after pasting into word processors. Indeed, the author's very first piece of open source software was a [text editor plug-in](TODO:link paste pdf) which would ease such a process.

## Why is This Important

The above (albeit brief) overview is important to understand because PDF has become somewhat of a default standard for publication of *data*. This is particularly true in the legal field, but it is also true in other fields. This is suboptimal for a few reasons.

The main reason publishing information via PDF documents is suboptimal is that getting *information in context* out of PDF documents is difficult. Data alone is important, but data in context is even better. This is especially important when the data being communicated, published, or transmitted is not simply a raw data feed but instead is data of a more complex nature (e.g., legal data and other data which is highly context driven).

What can be frustrating to those within the open data, open law, and open government communities who consume such data feeds, is that the information is *usually* (but not always) built in context but then all of that contextual data is lost when the information is rendered to a PDF.

For example, to process judicial opinions one must understand how to deal with footnotes. Any lawyer who has sent a judicial opinion to their ebook reader knows the pain that this causes. It is certainly not the makers of the ebook reader's responsibility to handle footnotes, because the computer programs used to change the PDF format of the judicial opinion to the ebook reader's format has no way to connect a blob of smaller text at the bottom of the page with (perhaps) a leading number offset just a small amount on the X axis as a footnote and to be able to link that back to a superscript number higher in the page. When the document was written, either in a free and open source word processor such as LibreOffice or in a paid and proprietary word processor such as Microsoft Word, that semantic linking of the text of the footnote to the footnote citation is maintained. However, when the document is rendered to PDF the semantic linkage between the footnote and the citing text is lost. The same reasoning applies to legislation, regulations, and many other sets of data which require context and semantics in order for machines to process them properly -- which would increase the flexibility in how entities and individuals consume that data.

## Authenticity of Publication

One of the largest reasons for why lawyers in particular publish documents (which really should be thought of as "content data") in PDF format is because PDFs are more difficult to modify. This is a valid observation. And it is (mostly) true. However, as with all digital formats, PDF's *can* be changed. All one really needs to change a PDF format is some clever [GIMP](TODO:link GIMP) skills. Even Adobe Acrobat PRO can allow one to change a PDF document. To change a PDF one needs to change the image layer and the text layer, but this is not a difficult process.

To combat such an ability to change the document, Adobe allows document publishers to "lock" PDFs. <!-- TODO: Confirm the following... --> blah, blah, blah.

While it is true that the above *works*, it begets the question as to whether it is the optimal way to ensure that the *information* is authentic. The challenge which well intentioned individuals are trying to overcome is a laudable goal: ensuring the legitimacy of the information. Where the implementation of that goal is suboptimal is in locking a public agency into an expensive, proprietary format which, in truth, solves a different problem set. The problem set which Adobe Acrobat Pro's "locking" feature is trying to solve is to ensure the *presentation* of the information is accurate and complete. The feature does ensure that the information is accurate and complete, but only derivatively and indirectly. There are other methods of ensuring that the data one is publishing is accurate and complete which do not require expensive, proprietary vendor lock in. <!-- TODO: Write this and link it here. -->

# Working with PDFs

Despite PDFs distinct disadvantages for publishing data, many public agencies still publish data in PDF formats. The remainder of this page will discuss the various methods for extracting data from published PDFs. There are two major "flavors" of PDF documents: those that have a text layer and those which do not. For working with documents which have a text layer please skip the next section. For working with documents which do not have a text layer please start with the next section. If you are unsure whether the set of PDFs you are seeking to work with has a text layer or does not, try opening the PDF in a PDF viewer and try selecting the text. If you are able to select text with your mouse or keyboard then the PDF does have a text layer, if you are not able to select text with your mouse or keyboard then the PDF does not have a text layer.

As with any data-driven task, one of the most important aspects of extracting data from PDFs is to have a plan for how one will handle the task. The below sections cover some of the different aspects which may be included in the overall plan. Generally speaking, if the document one is working with does not have a text layer then there will be three main steps one must take to derive data from the PDF source:

1. Recognize the text on the document.
2. Ensure the text layer is "cleaned" of noise.
3. Process the text into the required data format.

When one is dealing with documents which already have a text layer to them, the first two steps would generally be ignored. The three step process outlined above is general in nature and is meant only to be illustrative. Each data derivation task will require its own plan for how to transform the PDF documents into machine usable data.

# Step 1 -- Recognize the Document's Text

Optical Character Recognition, or OCR, is the task of processing image data and extracting useful text out of the image. It is sort of like machines "reading" a document. In actuality what is happening is that the machine is analyzing the pixels on the screen and matching the boundaries of differences in the pixel colors against patterns which have been preprogrammed into the OCR software.

## Optical Character Recognition -- Background

Digital images, for those who are not aware, are a set of very small squares (also called pixels) which have a specific color. Below is an example of the English language character "e" which has been zoomed in using GIMP (which is a free and open source Photoshop equivalent).

<!-- TODO: add screenshot 1 of "e" here -->

In the image above, viewers are mostly likely able to recognize the pattern of an English language "e" as well as the pixels or squares of colors. In order to simplify the example, we built the image as black text on a white background. However, the viewer can see that some of the pixels are shades of colors in between pure black and pure white. This is what allows fonts to render smooth curves and generally appear "nice" to the human eye.

In the next image, which is the exact same image only with the English language "e" written in a different font, you can see a greater amount of pixels which are neither pure black nor pure white in color.

<!-- TODO: add screenshot 2 of "e" here -->

As the above two images demonstrate, when "looking" at pixels and trying to derive patterns from those pixels there is a great difference in how fonts are rendered as images. This is important because as explained above, what OCR software is attempting to do is to match the pixels of an image against various patterns that the OCR software knows about.

The patterns which a piece of OCR software understands are exceedingly important to the success of the software as they provide the basis which the software is using to match patterns of the image against. There is a process of "training" OCR software which can be used to improve the output of an OCR rendering process. This has been leveraged by the CAPTCHA and ReCPATCHA program to improve the scanning reliability of text (a good place to start if one is interested in learning more about this process is [this TED talk](http://www.ted.com/talks/luis_von_ahn_massive_scale_online_collaboration)).

OCR software is exceedingly complex for a number of reasons. Differences in how images are taken, differences in fonts and how those are rendered as pixels in a digital image, differences in language patters, and "noise" within the image all contribute to the challenges which OCR software developers face. For this reason, perhaps among others, the state of free and open source OCR software is not currently optimal. [Here is a Wikipedia comparison of OCR software](http://en.wikipedia.org/wiki/Comparison_of_optical_character_recognition_software).  Although it is only one test, this blog post is a demonstrator of [the differences in OCR software](http://www.splitbrain.org/blog/2010-06/15-linux_ocr_software_comparison).

## Optical Character Recognition -- Top Tips

Working with OCR software can be a challenge, but it is often more efficient than typing large amounts of pages. As stated above, there are numerous variables which contribute to the success of an OCR rendering process. There are two main variables which users can control to optimize the process: the "noise" on the page which a OCR software is forced to analyze, and the "area" of the page which the OCR software is forced to analyze.

Noise to an OCR rendering process is pixels on the digital image which render closer to the text color than to the background color but which are not actually text. An example of this is when there is a dot on a page (perhaps from a pen marking or any other process) which then the OCR software renders as a backtick ("`"). Noise also works in the reverse, pixels which are closer to the background color than to the text color but actually should be rendering as text. An example of this is when there may be a lighter few pixels in the midst of an "l" character so that the OCR software will "think" that the closest pattern is actually an "i" character instead of the "l".

Below are some examples of noise and how that renders after the OCR process.

<!-- TODO: add noise screenshots here -->

**TOP TIP**: *get good scans*. The best way to reduce noise in a digital image is a good scan of the document. Over the past two decades scanner technology has increased dramatically. So if the documents one is working with are older documents, it may be worth the time to rescan the document if it is an old one. However, if the only hard copy of the document which is available is an old photocopy with lots of noise on it, then there is little one can do about such a situation.

**TOP TIP**: *minimize the OCR-ed area*. Minimizing the OCR-ed area is not available with all OCR software. In some OCR software suites, one can select via a graphical user interface, the area which the OCR software will analyze. The benefit of closely matching the area of the image which the OCR software will analyze to that human determined area of text on the page is that it will minimize the amount of noise a bad scan will output, as noise which is not within text blocks will not be analyzed. The screenshots below demonstrate how to do this in one OCR software.

<!-- TODO: add OCR area determination screenshots here -->

## Optical Character Recognition -- Exporting


