headlines:

# Headline 1
`# Headline 1`

## Headline 2
`## Headline 2`

### Headline 3
`### Headline 3`

#### Headline 4
`#### Headline 4`

##### Headline 5
`##### Headline 5`

###### Headline 6
`###### Headline 6`
#

### Text Effects
*italicise*
`*italicise*`
`_italicise_`

**BOLD**
`**BOLD**`
`__BOLD__`

***BOTH***
`***BOTH***`
`___BOTH___`

~~Strikethrough~~
`~~Strikethrough~~`

==Highlight==
`==Highlight==`

### Blocks Styles

>"Quote block
for several lines"

```
>"Quote block
for several lines"
```
2 newlines to unbind Quote block

`code span`
`` `code span` ``

```
code block
for several lines
```
````md
```
code block
for several lines
```
````

or
```
~~~
code block
for several lines
~~~
```
or just indent with tab:
```
	code block
	for several lines
```

for a nested code block add more backticks in the outer header/tails:
`````
```` <- invisible backticks, define block
``` <- visible backticks
code
```
````
`````


language specific code blocks:

python example:
```python
def main():
	print("Python block")
```
````
```python
def main():
	print("Python block")
```
````
or
````
~~~python
def main():
	print("Python block")
~~~
````
other supported languages[^1]:
* sh zsh bash powershell
* batch
* html
* css
* javascript js
* c
* csharp cs dotnet
* cpp
* lua

^supported-prism-list

[^1]: [full list of supported languages](https://prismjs.com/#supported-languages)
### Comments and Escapes
you can make %%unseen%% comments in the markup file like so:
`you can make %%unseen%% comments in the markup file like so:`

%%
and also
block comments
%%
```
%%
and also
block comments
%%
```
***
escape markdown characters with `\`:
for example:
- Asterisk: `\*`
- Underscore: `\_`
- Hashtag: `\#`
- Backtick: `` \` ``
- Pipe (used in tables): `\|`
- Tilde: `\~`
- Numbered List Item: `1\.`