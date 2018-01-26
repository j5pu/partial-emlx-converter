📧 .emlx and .partial.emlx to .eml converter
============================================

This script converts `.emlx` and `.partial.emlx` files written by Apple’s [Mail.app](https://en.wikipedia.org/wiki/Mail_(Apple)) into fully self-contained, “stand alone” `.eml` files which can be imported and opened by a great variety of email applications (Mail.app, Thunderbird, …).

Apple uses these formats for internal storage (see `~/Library/Mail/V4`), and under normal circumstances you will not come in contact with those files. Unfortunately, one of my IMAP mailboxes went out of service and I was not able to copy all the messages to a different account with Mail.app, even though all mails and attachments were there (see [here](https://apple.stackexchange.com/questions/312942/recovering-emails-from-defunct-imap-account) for the story).

That’s why I created this script.

## Build

Use a current version of [Node.js](https://nodejs.org/en/) (built with v6.11.0). Install the dependencies, run the tests, and compile the TypeScript code with [yarn](https://yarnpkg.com/lang/en/) or npm:

```
$ yarn
$ yarn run test
$ yarn run build
```

## Run

Run the script with exactly two arguments: (1) Path to the directory which contains the `.emlx` and `.partial.emlx` files, (2) path to the existing directory where the results should be written to.

```
$ node build/src/converter.js /path/to/input /path/to/result
```

## About the file formats

**Disclaimer:** I figured out the following by reverse engineering. I cannot give any guarantee about the correctness. If you feel, that something should be corrected, please let me know.

`.emlx` and `.partial.emlx` are similar to `.eml`, with the following peculiarities:

### .emlx

These files start with a line which contains the length of the actual `.eml` payload:

```
2945      
Return-Path: <john@example.com>
X-Original-To: john@example.com
…
```

The number `2945` denotes, that the actual `.eml` payload is 2945 characters long, starting from the second line.

At the end, these files contain an XML [property list](https://en.wikipedia.org/wiki/Property_list) epilogue, which holds some Mail.app-specific meta data. Using the given character length at the file’s beginning, this epilogue can be stripped away easily and an `.eml` file can be created.

### .partial.emlx

Mail.app uses this format to save emails which contain attachments. Attachments are saved as separate, regular files relative to the `.partial.emlx` file. Afaik, Apple does this due to Spotlight indexing.

Mail.app’s internal file structure looks as follows (nested into two further hierarchies of directories named with number 0 to 9):

```
Attachments/
  1234/
    1.2/
      image001.jpg
    2/
      file.zip
  …
Messages/
  1234.partial.emlx
  …
```

`1234` is obviously the email’s ID. The `Attachments` directory contains the raw attachment files, whereas `Messages` contains the messages stripped of their attachments (and `.emlx` files, for messages which did not contain any attached files in first place).

The subdirectories `1.2` and `2` in above’s example are numbered according to their positions within the corresponding email’s [Multipart](https://www.w3.org/Protocols/rfc1341/7_2_Multipart.html) hierarchy.

To convert a `.partial.emlx` file into an `.eml` file, the separated attachments need to be re-integrated into the file.

## Credits

Without the following modules I would probably be still working on this script (or have given up on the way). Thank you for saving me so much time!

* [content-disposition](https://github.com/jshttp/content-disposition)
* [content-type](https://github.com/jshttp/content-type)
* [eml-format](https://github.com/papnkukn/eml-format)
* [libqp](https://github.com/nodemailer/libqp)
* [rfc2047](https://github.com/One-com/rfc2047)

Beside that, here are some resources which I found very helpful during development:

* [Test Cases for HTTP Content-Disposition header field (RFC 6266) and the Encodings defined in RFCs 2047, 2231 and 5987](http://test.greenbytes.de/tech/tc2231/)
* [The Content-Transfer-Encoding Header Field](https://www.w3.org/Protocols/rfc1341/5_Content-Transfer-Encoding.html)

## Contributing

Pull requests are very welcome. Feel free to discuss bugs or new features by opening a new issue. In case you submit any bug fixes, please provide corresponding test cases and make sure that existing tests do not break.

- - -

Copyright (c) 2018 Philipp Katz