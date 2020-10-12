---
sort: 3
---

# Style Guide

When writing new documentation please try to follow the style guide for adding in new content. If you don't follow the guide it will cause a delay in how quickly the content is pulled into the documentation as editors will need to adjust or correct the content.

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

## This is a secondary header

### This is a tertiary header


## Notes

You can also write notes or info boxes to provide extra context or helpful notes when needed using the following syntax

```
```note
this is a note
```
```

```note
Notes are quite helpful for providing context
```

## Warnings
You can write warning boxes when you want to alert the reader that that there is important information to consider before they act or execute a command

```
```warning
You have been warned!
```
```

```warning
Warning boxes should be used sparingly and only when the reader should take caution before proceeding further
```

## Danger!
Danger boxes should be used very sparingly and should alert the reader that they either should not do what is suggested in the box, or should avoid it all together.

```
```danger
Danger Dr. Smith! Do not use `rm -rf \`
```
```

```danger
Always backup important system files before you decide to blow them away
```

---
### References

[Jekyll RTD Theme documentation](https://jekyll-rtd-theme.rundocs.io/)