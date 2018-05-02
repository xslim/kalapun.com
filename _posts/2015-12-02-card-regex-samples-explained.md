---
published: true
title: Card RegEx samples explained
---

Let me explain a few Card number RegEx patterns here. 

## `^3[47][0-9]{5,}$`

- `^` means to start from first character
- `3` - match the first character to be a number `3`
- `[47]` - match either `4` or `7`
- `[0-9]{5,}` - any numerical character, at least `5` times or more
- `$` - end of string

So this will match at least 7 or more numerical characters

## `^4[0-9]{6,}([0-9]{3})?$`
Similar as above, except 

- `()?` - it may (or may not) match the pattern inside `(` and `)`
- `([0-9]{3})?` - maybe 3 numerical characters at the end

## `^(5[1-5][0-9]{4}|677189)[0-9]{5,}$`

- `(x|y)` - match `x` or `y`
- `[0-9]{4}` - exactly 4 numerical characters
- `(5[1-5][0-9]{4}|677189)` - match `5[1-5][0-9]{4}` or `677189`
