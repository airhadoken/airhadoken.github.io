---
title: "JavaScript String Handling and Emoji"
author: "Bradley Momberger"
---

While working on a minor improvement for [@swagspiration](https://twitter.com/swagsipiration/) today, I found an few interesting facts about both the implementation of strings in JS which allows for emoji, and also a tricky behavior in Chrome (probably a bug).

# About UTF, Emoji, and JavaScript

JavaScript internally uses UTF-16 for storing strings.  The Unicode codepoints for emoji are outside the range of what UTF-16 can accomplish in one character.  UTF-16 has a reserved code page just for these cases, providing a way to store up to 20 bits of character data using two UTF-16 characters as *surrogates*.  What this ends up meaning in practice is that emoji are two characters in JavaScript.  If you want to regex for them, you have to test for the existence of a "head surrogate" character in the range of U+D800-U+DFFF **and also** include the next character.

The regex for a two-character pair where the first character is a head surrogate (and we'll assume that the next character is a tail surrogate) looks like this, then:  

```javascript
/[\uD800-\uDFFF]./ // This matches emoji
```

and does *not* look like any of these:

```javascript
/[\u1F300-\u1F6FF]/           // can only have four hex characters after \u
/\x01[\xF3-\xF6][\x00-\xFF]/  // \x only works for ASCII-compatible characters, not bytes of a string
/[\xD8-\xDF][\x00-\xFF]./     // Ditto.
```


# Magical Shrinking Strings (in Chrome)

What happens, then, if you only remove the head surrogate from your string, and leave the tail surrogate?  Let's find out.  Here's doing exactly that in Firefox console (remember that without the 'g' modifier to the regex, `replace()` only replaces the first match, which is the first non-newline character in the string for `/./`):

![remove from firefox](/images/firefox-surrogate-split.png)

That seems reasonable.  The tail surrogate is non-printable, so it gets a default Unicode glyph showing its codepoint.  Let's also try this in Chrome.

![remove from chrome](/images/chrome-surrogate-split.png)

WHOA!  The whole string seems to have disappeared!

In fact, it's not disappeared -- all the remaining text is still there, the length property is correct, `charCodeAt()` returns correct values, and removing the tail surrogate from the beginning causes the characters after it to correctly print again. In Chrome, text in a string ceases rendering immediately when a bare tail surrogate is encountered.  This is true of the console, and also of the DOM.  If you set the string above to be the text content of a text node, it would not show anything at all.

I'm unsure if this is a bug, but it is an interesting way of possibly hiding content in plain sight.