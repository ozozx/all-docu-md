### Internal Links
Markdown link:
[Markdown](markdown/Markdown) `[Markdown](markdown/Markdown)`
Wikilink:
[[Markdown]] `[[Markdown]]` (select endpoint in droplist)

[[Markdown|Wikilink]] with aliased text `[[Markdown|Wikilink]]`
^md-alias

### External Links

^d4f635

URL:
[Obsidian Help](https://help.obsidian.md)
Obsidian URI:
[Shit I Want Done](obsidian://open?vault=know-bsidian&file=SHIT%20I%20WANT%20DONE/All%20the%20Shit.md)
*obsidian URI* is constructed in the following format:
> header: `obsidian://open?`
> vault parameter: `vault=`*vault-name*
> note parameter (optional, spaces in URI are replaced with `%20`): `&file=`*path_to/file.md*
the destination vault must be imported to obsidian for the link to work:
`[Shit I Want Done](obsidian://open?vault=know-bsidian&file=SHIT%20I%20WANT%20DONE/All%20the%20Shit.md)`

spaces must be swapped with `%20`
or the entire URI can be escaped with (`< >`)
[Shit I Want Done](<obsidian://open?vault=know-bsidian&file=SHIT I WANT DONE/All the Shit.md>)
`[Shit I Want Done](<obsidian://open?vault=know-bsidian&file=SHIT I WANT DONE/All the Shit.md>)`

### Footnotes
inline footnote^[Footnote Here]
`inline footnote^[Footnote Here]`

one-line[^1] and several-line[^2] footnotes

[^1]: a single line footnote
[^2]: add 2 spaces at the start of subsequent lines
  to write multi-line footnotes.
```
one-line[^1] and several-line[^2] footnotes

[^1]: a single line footnote
[^2]: add 2 spaces at the start of subsequent lines
  to write multi-line footnotes.
```

### Embed Pages
add a `!` before an internal link to a page:
![[Markdown]]
`![[Markdown]]`

### Embed Blocks

embed specific headings / blocks (select in dropdown menu):
![[css/Links|Links]]
`![[css/Links|Links]]`

embed list by block identifier:
in the target page add at the end of a block newline and `^blockname`
example:
![[Text Formatting#^supported-prism-list]]
`![[Text Formatting#^supported-prism-list]]`

### Embed Images
add a `!` before any link to an image:
![Oz PFP](https://lh3.googleusercontent.com/a/ACg8ocLKCl0TzagCeNvN__gWIvvTcgHUkwF_tUu85f_wfQ_L41izpKxyHeuj1HWgRVXgnEfofgc2OELX9wR23tGvbtClTw9rMqql=s288-c-no)
`![Oz PFP](https://lh3.googleusercontent.com/a/ACg8ocLKCl0TzagCeNvN__gWIvvTcgHUkwF_tUu85f_wfQ_L41izpKxyHeuj1HWgRVXgnEfofgc2OELX9wR23tGvbtClTw9rMqql=s288-c-no)`

add a `|wxh` for width and height numbers to change the image dimensions
using `|w` for width only will set the height according to the original aspect ratio ^2e38eb

![Oz PFP|64](https://lh3.googleusercontent.com/a/ACg8ocLKCl0TzagCeNvN__gWIvvTcgHUkwF_tUu85f_wfQ_L41izpKxyHeuj1HWgRVXgnEfofgc2OELX9wR23tGvbtClTw9rMqql=s288-c-no)
`![Oz PFP|64](https://lh3.googleusercontent.com/a/ACg8ocLKCl0TzagCeNvN__gWIvvTcgHUkwF_tUu85f_wfQ_L41izpKxyHeuj1HWgRVXgnEfofgc2OELX9wR23tGvbtClTw9rMqql=s288-c-no)`
![Oz PFP|128x64](https://lh3.googleusercontent.com/a/ACg8ocLKCl0TzagCeNvN__gWIvvTcgHUkwF_tUu85f_wfQ_L41izpKxyHeuj1HWgRVXgnEfofgc2OELX9wR23tGvbtClTw9rMqql=s288-c-no)
`![Oz PFP|128x64](https://lh3.googleusercontent.com/a/ACg8ocLKCl0TzagCeNvN__gWIvvTcgHUkwF_tUu85f_wfQ_L41izpKxyHeuj1HWgRVXgnEfofgc2OELX9wR23tGvbtClTw9rMqql=s288-c-no)`

### Embed Search Results
only works inside vault
```query
path:markdown
```
````
```query
path:markdown
```
````
