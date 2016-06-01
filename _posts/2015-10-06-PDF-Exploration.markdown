---
layout: post
title:  "Explore PDF"
date:   2015-10-06 15:53:00
categories: PDF
---

> *This article is to illustrate and explore the popular encoding format "PDF"*

Background: I was asked to develop an application to extract a pdf and convert those traditional Chinese to simplified. Sounds easy, ahuh?
At first, I think it's not that difficult until I really get suck into the "notorious" format. So, to begin with, let's look at the brief of what is a PDF.

#### 1. What is the PDF?

>***Portable Document Format (PDF) is a file format used to present documents in a manner independent of application software, hardware and operating systems.[2] Each PDF file encapsulates a complete description of a fixed-layout flat document, including the text, fonts, graphics and other information needed to display it. [From [WikiPedia](http://www.wikiwand.com/en/Portable_Document_Format)]***

Basically, the PDF is designed as a universal reading format so that the document can be viewed in various kinds of devices. Therefore, it's not expected to "edit" a pdf document as per the "universal read" design principle. But, if you are ambitious enough (just like me), you would forcibly process a PDF and make some kinds of modification on that with the help of an Apache project ["PDFBox"](https://pdfbox.apache.org/) . Fortunately, PDFBox is a powerful lib for processing a pdf document. There are apis for pdf extraction and parse. What's more, it is open source! So I can even customize the lib to suit for my needs. But I would say there is a little bit mess of the PDFBox source code, I mean the source code it's rather in a complicated nature as I read. Well, that will be another post to talk about the PDFBox.

Since there is no way to overwrite the PDF document, the only way to replace the traditional chinese characters with simplified is to re-write the them to a new file, a "copy-and-replace" solution. It seems that the strategy is reasonable and straightforward. But as you explore more in the PDF structure and its format encoding, you will find it's a nightmare to simplify the solution as "copy-and-replace". To sum up, you have to pay if you insist to edit the PDF!!! Now, let's go thru the PDF structure.

#### 2. PDF Structure

Significantly, a PDF document is composed of 3 key elements images, text and graphics (symbols/lines/arrows/etc). So if you want to completely rewrite a pdf, you'll have to extract all the information of those elements and use them to recover in the new file.

##### 2.1 Text

>***Text in PDF is represented by text elements in page content streams. A text element specifies that characters should be drawn at certain positions. The characters are specified using the encoding of a selected font resource.***

So to edit a text, you have to deal with two things, one is the font and the other is "encoding". English and ASCII characters are easy to handle in this case as the font and encoding would not be complicated for those characters. But for chinese characters, it's a catastrophe.

Let's say I have parsed the text position and get all the meta information I need for recovering. The first step is to detect what exactly is the "character", whether a English letter or chinese character. Fortunately, pdfBox provide api to retrieve the Unicode of a particular character. Assume that a traditional chinese character "檔" is found and wanna change it to simplified "档". Even though you can recognize the character by reading its unicode but again the nightmare occurs, you will have to specify a font which contains simplified chinese charaset and then put the corresponding Unicode to the font writer and then write it out.

##### 2.2 Images & symbols

The most critical part is to recover images & symbols as their nature of complication. Unfortunately, the meta-data for images and symbols would be hardly retrieved. Even though you can get the full meta data of images or symbols, yet pdfBox doesn't provide such a powerful api for fully images or symbols recovery. Frankly speaking, you'll have no way to recover the images and symbols without the help of pdfBox. So that's why I have to give up my "copy-and-replace" solution. 

***References:***

> <http://ccckmit.wikidot.com/pdf:streamcoding>
