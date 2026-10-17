# Bionic Font Maker

A small web tool that turns any TTF/OTF font into a "bionic reading" version: the first letters of every word are drawn in bold, so the eye gets an anchor point for each word. You upload a font, adjust two sliders, and download the new font.

Everything runs in your browser (Python via WebAssembly). Your font files are never uploaded to any server.

**Live version:** https://jopidovba-a11y.github.io/Bionic-Font-Maker/

## How to use

1. Open the page and wait for the status line to say the runtime is ready (first load downloads about 10 MB).
2. Choose a **regular** font (`.ttf` or `.otf`).
3. Optionally choose the **bold** font of the same family. It looks much better than synthetic bold.
4. Set the bold portion of each word and, if you did not provide a bold font, the synthetic boldness.
5. Press **Generate font** and then **Download**. The file is named like the original with `-Bionic` added, and the font family name also gets a ` Bionic` suffix so it does not clash with the original.
6. Install the font on your device, or copy it to the fonts folder of your reading app.

## How it works

A font cannot know where a word starts or ends, so the tool uses OpenType contextual substitution (the `calt` feature):

- Every Latin, Greek and Cyrillic letter gets a bold copy.
- Substitution rules detect each word and swap its first letters (the percentage you choose) for the bold copies.

The result is a normal font file. The reading app only needs to shape text with HarfBuzz, so no changes to the book are needed.

## Limitations

**Fonts**

- Only **static** fonts are supported. Variable fonts are rejected.
- CID-keyed CFF fonts (some `.otf` files) are rejected.
- `.otf` files with CFF outlines are supported, but this path is less tested than TrueType (`.ttf`). If you hit an error, try the `.ttf` version of the font.
- Only **Latin, Greek and Cyrillic** letters are processed. Other scripts are left untouched.

**Bolding rules**

- Words are bolded exactly up to **14 letters**. In longer words, the bolding repeats in a second chunk.
- A word is a run of letters. Hyphenated words count as separate words ("barrel-chested" gets two bold parts).
- For contractions, the bold portion is computed from the part before the apostrophe, and up to 6 letters after the apostrophe are kept regular.
- If a book breaks a word across lines with a hyphen, each piece is treated as its own word, so the bold pattern can differ from the unbroken word.

**Size and performance**

- Every letter gets two extra glyphs, and synthetic bold multiplies outlines, so the generated font is **noticeably larger** than the original. Using a real bold font gives smaller files and better quality.
- Text shaping is heavier than with a normal font. In e-readers with modest hardware, opening a book for the first time with the font (when the layout is computed) can take longer. Test with a long book before relying on it.
- Generating a font can take several seconds in the browser, and very large fonts may be slow or run out of memory, especially on phones.

**Compatibility**

- The effect only appears in apps that apply OpenType `calt` through a shaping engine such as HarfBuzz. In apps without it, the font works but looks like the regular font. In KOReader, check that the font shaping/kerning mode uses HarfBuzz.
- The tool needs a modern browser with WebAssembly support.

## Fonts and licenses

You are responsible for the fonts you process. Many open fonts (for example SIL OFL) allow modification, but some require changing the font name or forbid derivative versions, and commercial fonts often forbid modification entirely. Check the license of your font before generating or sharing a modified version.

## Built with

- [Pyodide](https://pyodide.org/) to run Python in the browser
- [fontTools](https://github.com/fonttools/fonttools) to read, edit and write fonts and compile the OpenType rules
