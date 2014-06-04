gigbag instant songbook
======

# Overview

This is a script designed to create nice songbooks in PDF format with minimal effort. It parses songs lazily copy-pasted from any songbook, website or text file, with very little or no hand editing, and produces lean LaTeX source code. This should compile into a pdf booklet with consistent layout, chord diagrams, indexing, formatting etc.

It won't be 100% automatic, but preparing a copy-pasted file for output can literally take seconds, possibly saving you hours of hand editing. If all goes well, all you need to do is mark the song title, and ensure that verses of the song are consistently separated by empty lines - the script should take care of the rest.

**Be warned, though:**
    
* this is my first project in Python, and first serious attempt at coding since high school. Because of this, the code has stability, efficiency and elegance of a bicycle with a tank tower, and would probably make a proffesional programmer cry
* this is a command line tool. And chances are that if you know some LaTeX and Python, you can use the command line anyway, so it probably will never get any GUI at all
* output is produced as LaTeX source code, which needs to be compiled in PDFLaTeX with [Songs](http://songs.sourceforge.net/)  package (by Kevin W. Hamlen, praise the man for awesome work!). 
* even when the code works perfectly (which, suprisingly, happens from time to time), there might be some corrections in the created LaTeX source to be made, like adjusting line and page breaking etc. This requires some basic understanding of LaTeX and Songs. Refer to the relevant documentation for help
* this is a working prototype, which has only rudimentary exception handling, input sanitizing and all this stuff that makes a piece of software foolproof. It behaves reasonably on most probable inputs, but if you feed it gibberish, it will obediently serve you random word salad as output 
* last thing - this was created in Debian and NOT tested anywhere near Windows yet. Should be portable, as both Python and LaTeX are, but...

Ok, you've been warned, so here's the good part.

## Features

* understands most popular conventions used in text files with songs and chords - chords in, above or at the end of the lyric line
* creates diagrams of all chords used in a song (for ukulele as for now, if you want to make a guitar config file, be my guest...)
* chords are easily customizable for alternative positions, other instruments or for simply adding the occasional missing one
* automatically creates index entries
* output file can be enhanced with all the goodness of the Songs package, including layout tinkering, automatic transposition and other fancy stuff
* a custom template used for the songbook of Polish Ukulele facebook group is provided, but can be easily adjusted to your needs

## Usage

Paste your songs into a file. Mark the titles and adjust formatting if needed - check below for details. Run the script.

    python3 gigbag.py [-options] <input_file> 
    
* -o (--output): Specify output file. By defaults appends to songs.sbd if not specified
* -k ( --keywords): Specify keywords file. Defaults to keywords.cfg if not specified
* -c (--chords): Specify chord definitions file. Defaults to chords_ukulele if not specified
* -w   ( --overwrite): Overwrites output file instead of appending

Review the output file, and let LaTeX do the rest:

    songidx titles.sxd titles.sbx
    songidx authors.sxd authors.sbx
    pdflatex chordbook.tex


## How to prepare input file

* use UTF-8 encoding if any strange characters are present in your input (or change encoding in appropriate TEX file). You might require a text editor slightly less retarded than default Notepad
* song titles must be explicitly formatted by adding a hash ("#") at the beginning, the rest of file hopefully will be understood as-it-is
* parts of song are separated by empty lines. This might cause most of the editing required, as many websites by default insert a new line between each line of lyrics, but this won't probably be fixed. Too much trouble may appear elsewhere, and the effort to delete a few lines seems not worth it
* certain parts (choruses, bridges etc.) are identified by a keyword in the first line, ex. "Chorus". Popular choices will be understood, and more can be easily added. 


### Song headers

* a line preceeded with a # will be interpreted as a song header. This is the only part of input formatting, which is really obligatory (syntax derived from Markdown)
* use a slash in the header to provide alternative title; this will not be displayed, but will be included in the index
* first line after song header is understood as artist name, second line as authors of the song. If an empty line is found instead, those fields will be empty

Example:

    # Love me do / That Love Song for Retards
    The Beatles
    Lennon, McCartney
    
### How are chords understood

The script tries to understand all popular styles of typing chords and song lyrics. At such, it will probably not be perfect, but this is intentional - it is easier to correct some strange behaviour in the LaTeX output, than not implement it at all and retype those parts by hand.

So, everything below will be processed correctly:

    Love [G] love me [C] do

    Love, love me do    G C
    
     C            G
    Love, love me do
    
    Intro:
    C G F G7

Be aware that a phrase in square brackets is unconditionally understood as a chord. It will cause a warning if the fingering is not known. A chord without brackets will be checked in the list of known chords. If it's not there, it will pass silently and incorectly as a word of lyrics.

Occasionaly there are problems with 'A' or 'a' at the end or beginning of a line, which the script confuses with a chord. If this happens, a quick workaround is to place a '%; character before this letter in the source, which will instruct the script to ignore chordifying this. So this will render correctly:
    
    %A times they are a-changin  G C
    
But this will NOT:

    A times they are a-changin C C

The script will find an abigous "A" at the beginning of line, and not render any chords at all here. So if you find that a line was wrongly processed as lyrics-only, this is the most probable explanation. Add a "%" and it will be ok. Alternatively, the same misbehaviour might result from an uknnown chord name in the sequence at the end of the line - in that case, just add the chord manually.
    
    

## How are song sections understood

All blocks of text (verses, choruses etc) should be separated by empty lines. If those are missing though, but the first line of a new block will be a keyword which indentifies it (ex. Verse 2), the parser will seperate them anyway.

A block of text, separated by empty lines (any number) will be by default understood as a verse. Chords will be placed as described above. If the first line is a keyword stating it is a verse (ex. Verse 1), it will be ignored in output. Verses are numbered in the PDF.

A block of text, which starts with a line describing a chorus (list of keywords can be changed in a config file) will be interpreted as a chorus, which are formatted differently and not numbered. First line of a chorus will be added to the index. Keyword describing a chorus will be ignored in output.

A block of text, which either contains only chords, or starts with a keyword describing an instrumental part (ex. Intro), will be formatted similar to a verse, but not numbered. This time, keyword will NOT be ignored, but added to the output as well.

Note that adding comments to a song hasn't been implemented yet, even though the keyword for that is already in the code.


## How are chord diagrams added

All chords in a song, which are on the list of chords supplied in a config file, will be rendered at the end of the song as diagrams. The supplied config file is for the ukulele, but it can be changed to anything else. 

If there are chords, which are not on the list, but are put in square brackets, there will be a warning of a missing chord, but the output should be correct (apart from a missing diagram). 

If there are chords, which are not on the list and are not in square brackets, there will silently pass as lyrics and cause a mess, so watch out for them.


## How are keywords processed

As for now, they're read from separate config file, and most common variations (numbering, a colon, a dot and case variations) are created automatically. Only keywords in a separate line will be understood, but this will probably be fixed later.

## Known problems

* A-major and A-minor, written as A and a, can be confused with a letter A used as a word inside a lyric line. This will usually not cause any problems, unless it is either the first, or the last letten in a line. Fixing the first case is possible, though not trivial, and the second one - probably not, so just edit the LaTeX output, if it happens.

* A keyword like "CHORUS" is often used at the beginning of a line, which is currently ignored by the parser. This will be fixed later. In the meantime, just separate as a new line in input.

* On some websites there will be an empty line between every line in the song. As the parser depends on empty lines for dividing songs into blocks, this is not trivial to fix, so you have to remove those manually for now.

## Changelog

* Gigbag 0.6.1
    * added all missing TEX files. Content can be unpacked and compiled with LaTeX without any additions to the working directory, if the environment is configured and Songs package installed
    * fixed a bug with displaying authors in the header
    * some cleanup and missing docstrings added in the code

## Ideas for the next update:
    
* understanding lines starting with '|' character as a chorus - quite common practice
* understanding blocks consistently idented as choruses
* implementing comments
* implementing repetitions and echo markers
* rewriting the keyword engine with regular expressions, as for now it is ugly and ridiculously inefficient






