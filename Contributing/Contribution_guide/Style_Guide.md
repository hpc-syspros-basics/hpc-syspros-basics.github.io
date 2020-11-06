---
sort: 3
---

# Style Guide

When writing new documentation please try to follow the style guide for adding in new content. If you don't follow the guide it will cause a delay in how quickly the content is pulled into the documentation as editors will need to adjust or correct the content. This project also uses a markdown linter to check for any overall stylistic inconsistencies and catch some common problems with syntax. For more information on what markdown rules we adhere to read to the bottom of this page for more information.

[![GitHub Super-Linter](https://github.com/hpc-syspros-basics/hpc-syspros-basics.github.io/workflows/Lint%20Code%20Base/badge.svg)](https://github.com/marketplace/actions/super-linter)

## Headers
At the very least you should have a main header which can be written as so below:

```
# This is our main header
```

You can also have other headers underneath the main header. Secondary headers will automaticly have a menu option generated for them to quickly bring readers to that header's section from the left hand side menu of the page.

```
## This is a secondary header

### This is a tertiary header
```

## Text

Text can be modified in several ways, you can **bold**, *italalicize*, or ~~strikethough~~ portions of text.

```
Text can be modified in several ways, you can **bold**, *italalicize*, or ~~strikethough~~ portions of text.
```

Additionally for calling out commands or files (eg. `sync` or `/etc/hosts`) in descriptions outside of code blocks you can use inline code blocks to seperate them out from the rest of the text for easy reading.

## Code Blocks

You can also insert code blocks into the documentation and even get some syntax highlighting

```bash
echo "How are you today?"
exit 0
```

~~~~
```bash
echo "How are you today?"
exit 0
```
~~~~

## Tables

Below you will find an example of a formatted table. In the second column you will notice that we have the column values center aligned, and the third column we have right aligned.

| Partition   | # Nodes    | # CPUs   |
|-------------|:----------:|---------:|
|SHAS         | 470        | 11,280   |
|SMEM         | 7          | 168      |
|SKNL         | 20         | 5,440    |

```
| Partition   | # Nodes    | # CPUs   |
|-------------|:----------:|---------:|
|SHAS         | 470        | 11,280   |
|SMEM         | 7          | 168      |
|SKNL         | 20         | 5,440    |
```

## Lists

You can include both unordered and ordered lists like seen below as well

1. This is the first item
2. Here is a second item
3. Finally a third item

- First unordered item
- Second unordered item
- Final unordered item

```
1. This is the first item
2. Here is a second item
3. Finally a third item

- First unordered item
- Second unordered item
- Final unordered item
```

## Notes

You can also write notes or info boxes to provide extra context or helpful notes when needed using the following syntax


```note
this is a note, ~~it can be pointless~~
```


```note
Notes are quite helpful for providing context
```

## Warnings
You can write warning boxes when you want to alert the reader that that there is important information to consider before they act or execute a command

```warning
**You have been warned!**
```

```warning
Warning boxes should be used sparingly and only when the reader should take caution before proceeding further
```

## Danger
Danger boxes should be used very sparingly and should alert the reader that they either should not do what is suggested in the box, or should avoid it all together.

```danger
Danger Dr. Smith! Do not use `rm -rf \`
```

```danger
Always backup important system files before you decide to blow them away
```

~~~~
# examples of Notes, Warnings, Tips, and Danger cards
```note
text here
```
~~~~

~~~~
```tip
text here
```
~~~~

~~~~
```warning
text here
```
~~~~

~~~~
```tip
text here
```
~~~~

~~~~
```danger
text here
```
~~~~

## Markdown Linter

For this project we use the markdown linting capabilities offered by [Markdownlint](https://github.com/DavidAnson/markdownlint), deployed as a github action to lint all commits to the master branch of this repository. We have a markdown lint rules file within the [.github/linters directory](https://github.com/hpc-syspros-basics/hpc-syspros-basics.github.io/blob/master/.github/linters/markdown-lint.yml) with the settings that we use for various rules and rule exceptions where they are needed.

A few notable rules to keep in mind are [MD003](https://github.com/DavidAnson/markdownlint/blob/main/doc/Rules.md#md003) which specifies that there can only be one H1 header in a document, and [MD026](https://github.com/DavidAnson/markdownlint/blob/main/doc/Rules.md#md026) which specifies that there should be no punctuation in a header element.

---
## References

[Jekyll RTD Theme documentation](https://jekyll-rtd-theme.rundocs.io/)
