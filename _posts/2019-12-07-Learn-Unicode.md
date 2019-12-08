---
layout: post
title: Learn Unicode
---

## [Unicode® Tutorials and Overviews](http://www.unicode.org/standard/tutorial-info.html)

## [The Unicode® Standard: A Technical Introduction](https://www.unicode.org/standard/principles.html)

To keep character coding simple and efficient, the Unicode Standard assigns each **character** a unique **numeric value** and **name**.  

The Unicode Standard and ISO/IEC 10646 support three **encoding forms** (UTF-8, UTF-16, UTF-32) that use a common repertoire of characters.  

The majority of common-use characters fit into the first 64K **code points**, an area of the **codespace** that is called the **basic multilingual plane**, or **BMP** for short. There are fourteen other, supplementary planes available for future encoding of other characters, with currently over 800,000 unused code points.  

### Encoding Forms

Character encoding standards define not only the identity of each character and its numeric value, or **code point**, but also how this value is **represented in bits**.  

The Unicode Standard defines three encoding forms that allow the same data to be transmitted in a byte, word or double word oriented format (i.e. in 8, 16 or 32-bits per code unit). All three encoding forms encode the same common character repertoire and can be efficiently transformed into one another without loss of data.  

UTF-8 is popular for HTML and similar protocols. UTF-8 is a way of transforming all Unicode characters into a variable length encoding of bytes. It has the advantages that the Unicode characters corresponding to the familiar ASCII set have the same byte values as ASCII, and that Unicode characters transformed into UTF-8 can be used with much existing software without extensive software rewrites.  

UTF-16 is popular in many environments that need to balance efficient access to characters with economical use of storage. It is reasonably compact and all the heavily used characters fit into a single 16-bit code unit, while all other characters are accessible via pairs of 16-bit code units.  

UTF-32 is useful where memory space is no concern, but fixed width, single code unit access to characters is desired. Each Unicode character is  encoded in a single 32-bit code unit when using UTF-32.  

All three encoding forms need at most 4 bytes (or 32-bits) of data for each character.  

### Defining Elements of Text

Written languages are represented by **textual elements** that are used to create words and sentences. These elements may be letters such as "w" or "M"; characters such as those used in Japanese Hiragana to represent syllables; or ideographs such as those used in Chinese to represent full words or concepts.  

The definition of **text elements** often changes depending on the process handling the text. For example, in historic Spanish language sorting, "ll"; counts as a single text element. However, when Spanish words are typed, "ll" is two separate text elements: "l" and "l".

To avoid deciding what is and is not a text element in different processes, the Unicode Standard defines **code elements** (commonly called "**characters**"). A code element is fundamental and useful for computer text processing. For the most part, code elements correspond to the most commonly used text elements. In the case of the Spanish "ll", the Unicode Standard defines each "l" as a separate code element. The task of combining two "l" together for alphabetic sorting is left to the software processing the text.

### Text Processing

Computer text handling involves **processing** and **encoding**. Consider, for example, a word processor user typing text at a keyboard. The computer's system software receives a message that the user pressed a key combination for "T", which it encodes as U+0054. The word processor stores the **number** in memory, and also passes it on to the display software responsible for putting the character on the screen. The display software, which may be a window manager or part of the word processor itself, uses the number as an **index** to find an image of a "T", which it draws on the monitor screen. The process continues as the user types in more characters.  

The Unicode Standard directly addresses only the encoding and semantics of text. It addresses no other action performed on the text. For example, the word processor may check the typist's input as it is being entered, and display misspellings with a wavy underline. Or it may insert line breaks when it counts a certain number of characters entered since the last line break. **An important principle of the Unicode Standard is that it does not specify how to carry out these processes as long as the character encoding and decoding is performed properly.**  

### Interpreting Characters and Rendering Glyphs

The difference between identifying a code point and rendering it on screen or paper is crucial to understanding the Unicode Standard's role in text processing. The character identified by a Unicode code point is an abstract entity, such as "LATIN CHARACTER CAPITAL A" or "BENGALI DIGIT 5." The mark made on screen or paper—called a glyph—is a visual representation of the character.

The Unicode Standard does not define glyph images. The standard defines how characters are interpreted, not how glyphs are rendered. The software or hardware-rendering engine of a computer is responsible for the appearance of the characters on the screen. The Unicode Standard does not specify the size, shape, nor style of on-screen characters.

### Character Sequences

**Text elements** are encoded as sequences of one or more **characters**. Certain of these sequences are called **combining character sequences**, made up of a **base letter** and one or more **combining marks**, which are rendered around the base letter (above it, below it, etc.). For example, a sequence of "a" followed by a combining circumflex "^" would be rendered as "â". For more information on how **sequences of characters** are used to represent text in different languages, see "[Where is my Character?](https://www.unicode.org/standard/where/)", and for information on grapheme clusters (what end-users think of as characters), see UAX #29, Unicode Text Segmentation.

**The Unicode Standard specifies the order of characters in a combining character sequence**. The **base character** comes first, followed by one or more **non-spacing marks**. If there is more than one non-spacing mark, the order in which the non-spacing marks are stored isn't important if the marks don't interact typographically. If they do interact, then their order is important. **The Unicode Standard specifies how successive non-spacing characters are applied to a base character, and when the order is significant**.

Certain sequences of characters can also be represented as a single character, called a **precomposed character** (or **composite** or **decomposable character**). For example, the character "ü" can be encoded as the single code point U+00FC "ü" or as the base character U+0075 "u" followed by the non-spacing character U+0308 "¨". The Unicode Standard encodes precomposed characters for compatibility with established standards such as Latin 1, which includes many precomposed characters such as "ü" and "ñ".

Precomposed characters may be **decomposed** for consistency or analysis. For example, in alphabetizing (collating) a list of names, the character "ü" may be decomposed into a "u" followed by the non-spacing character "¨". Once the character has been decomposed, it may be easier for the collation to work with the character because it can be processed as a "u" with **modifications**. This allows easier alphabetical sorting for languages where character modifiers do not affect alphabetical order. **The Unicode Standard defines the decompositions for all precomposed characters. It also defines normalization forms to provide for unique representations of characters.**

### Principles of the Unicode Standard

The character sets of many existing international, national and corporate standards are incorporated within the Unicode Standard. For example, its first 256 characters are taken from the widely used Latin-1 character set.  

Duplicate encoding of characters is avoided by unifying characters within scripts across languages; characters that are equivalent in form are given a single code. Chinese/Japanese/Korean (CJK) consolidation is achieved by assigning a single code for each ideograph that is common to more than one of these languages. This is instead of providing a separate code for the ideograph each time it appears in a different language. (These three languages share many thousands of identical characters because their ideograph sets evolved from the same source.)

The Unicode Standard specifies an algorithm for the presentation of text with bidirectional behavior, for example, Arabic and English. Characters are stored in logical order. The Unicode Standard includes characters to specify changes in direction when scripts of different directionality are mixed. For all scripts Unicode text is in logical order within the memory representation, corresponding to the order in which text is typed on the keyboard.

### Assigning Character Codes

A **single number** is assigned to each **code element** defined by the Unicode Standard. Each of these numbers is called a **code point** and, when referred to in text, is listed in hexadecimal form following the prefix "U+". For example, the code point U+0041 is the hexadecimal number 0041 (equal to the decimal number 65). It represents the character "A" in the Unicode Standard.

Each character is also assigned a **unique name** that specifies it and no other. For example, U+0041 is assigned the character name "LATIN CAPITAL LETTER A." U+0A1B is assigned the character name "GURMUKHI LETTER CHA." These Unicode names are identical to the ISO/IEC 10646 names for the same characters.

The Unicode Standard groups characters together by **scripts in blocks**. A script is any system of related characters. The standard retains the order of characters in a source set where possible. When the characters of a script are traditionally arranged in a certain order—alphabetic order, for example—the Unicode Standard arranges them in its **codespace** using the same order whenever possible. **Blocks vary greatly in size**. For example, the Cyrillic block does not exceed 256 code points, while the blocks for CJK ideographs contain many thousands of code points.

Characters are grouped logically throughout the range of code points, called the **codespace**. The coding starts at U+0000 with the standard ASCII characters, and continues with Greek, Cyrillic, Hebrew, Arabic, Indic and other scripts; then followed by symbols and punctuation. The codespace continues with Hiragana, Katakana, and Bopomofo. The unified Han ideographs are followed by the complete set of modern Hangul. The range of **surrogate code points** is reserved for use with UTF-16. Towards the end of the **BMP** is a range of code points reserved for private use, followed by a range of **compatibility characters**. The compatibility characters are character variants that are encoded only to enable transcoding to earlier standards and old implementations, which made use of them.

A range of code points on the BMP and two very large ranges in the supplementary planes are reserved as **private use areas**. These code points have no universal meaning, and may be used for characters specific to a program or by a group of users for their own purposes. For example, a group of choreographers may design a set of characters for dance notation and encode the characters using code points in user space. A set of page-layout programs may use the same code points as control codes to position text on the page. The main point of user space is that the Unicode Standard assigns no meaning to these code points, and reserves them as user space, promising never to assign them meaning in the future.

### Conformance to the Unicode Standard

Implementations of the Unicode Standard are conformant as long as they follow the rules for the encoding characters into sequences of bytes, words or double words that are in effect for the chosen encoding form and otherwise interpret characters according to the Unicode specification.  
