### Unordered Lists
* item
* another item
* yet another item
```
* item
* another item
* yet another item
```
or
```
- item
- another item
- yet another item
```
or
```
+ item
+ another item
+ yet another item
```

list items of different prefix character will not connect

### Ordered List
1. first item
2. second item
3. third item
```
1. first item
2. second item
3. third item
```
or
```
1) first item
2) second item
3) third item
```

### Task List
- [x] task done and crossed
- [?] task done %%any character other than 'x' and ' '%%
- [ ] not done
```
- [x] task done and crossed
- [?] task done %%any character other than 'x' and ' '%%
- [ ] not done
```

### Nested list
indent list items with `Tab` / `Shift+Tab` to nest them:
1. First list item
	1. Ordered nested list item
2. Second list item
	- Unordered nested list item
```
1. First list item
	1. Ordered nested list item
2. Second list item
	- Unordered nested list item
```

tasks can also  be nested:
- [ ] Objective 1
	- [x] Sub-Objective 1.1
	- [ ] Sub-Objective 1.2
- [x] Objective 2
	- [x] Sub-Objective 2.1
	- [ ] Sub-Objective 2.2
```
- [ ] Objective 1
	- [x] Sub-Objective 1.1
	- [ ] Sub-Objective 1.2
- [x] Objective 2
	- [x] Sub-Objective 2.1
	- [ ] Sub-Objective 2.2
```

### Horizontal Rule
***
any three or more of the following characters in its own line, spaces between the characters are fine too:
```
***
---
___
```
### Tables
use `|` to separate columns and `-` to define headers

First Name|Last Name
-|-
Max|Planck
Marie|Curie

```
First Name|Last Name
-|-
Max|Planck
Marie|Curie
```
or for better asthetic:
```
|First Name|Last Name|
|----------|---------|
|Max       |Planck   |
|Marie     |Curie    |
```

text alignment is determined in the header seperator:

Left|Center|Right
:--|:--:|--:
ape|bee|sea

```
Left|Center|Right
:--|:--:|--:
ape|bee|sea
```

when using an alias^[![[Links#^md-alias]]] or resizing^[![[Links#^2e38eb]]] an image the `|` must be escaped with `\`:

First Column| Second Culumn
--|--
[[Markdown\|Markdown Home]]|![Oz PFP\|128](https://lh3.googleusercontent.com/a/ACg8ocLKCl0TzagCeNvN__gWIvvTcgHUkwF_tUu85f_wfQ_L41izpKxyHeuj1HWgRVXgnEfofgc2OELX9wR23tGvbtClTw9rMqql=s288-c-no)

```
First Column| Second Culumn
--|--
[[Markdown\|Markdown Home]]|![Oz PFP\|128](https://lh3.googleusercontent.com/a/ACg8ocLKCl0TzagCeNvN__gWIvvTcgHUkwF_tUu85f_wfQ_L41izpKxyHeuj1HWgRVXgnEfofgc2OELX9wR23tGvbtClTw9rMqql=s288-c-no)
```

### Callouts
callouts add info in a block

> [!info] Callout Title
> block content
> supports anything but tables

```
> [!info] Callout Title
> block content
> supports anything but tables
```

> [!info]+
> set callout as collapsible with a `+`, and as collapsed by default with a `-`

> [!info]- Collapsed callout
> hidden body

```
> [!info]+
> set callout as collapsible with a `+`, and as collapsed by default with a `-`

> [!info]- Collapsed callout
> hidden body
```

> [!question] Callouts
> > [!todo] Can Be
> > > [!example] Nested

```
> [!question] Callouts
> > [!todo] Can Be
> > > [!example] Nested
```

callout types:

> [!note]-
> ```
> > [!note]
> ```

> [!abstract]-
> ```
> > [!abstract / summary / tldr]
> ```

> [!todo]-
> ```
> > [!todo]
> ```

> [!info]-
> ```
> > [!info]
> ```

> [!tip]-
> ```
> > [!tip / hint / important]
> ```

> [!success]-
> ```
> > [!success / check / done]
> ```

> [!question]-
> ```
> > [!question / help / faq]
> ```

> [!warning]-
> ```
> > [!warning / caution / attention]
> ```

> [!failure]-
> ```
> > [!failure / fail / missing]
> ```

> [!danger]-
> ```
> > [!danger / error]
> ```

> [!bug]-
> ```
> > [!bug]
> ```

> [!example]-
> ```
> > [!example]
> ```

> [!quote]-
> ```
> > [!quote / cite]
> ```
