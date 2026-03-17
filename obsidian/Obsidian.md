---
title: properties
link: "[[Markdown]]"
url: https://help.obsidian.md/properties
year: 2026
python-version: 3.14
family:
  - Dad
  - Mom
  - Sister
marks:
  - "[[markdown/Links|Links]]"
  - "[[Text Formatting]]"
favorite: true
reply: false
last:
birthday: 2000-01-14
next-lesson: 2026-02-15T18:00
tags:
  - tagone
  - tagtwo
aliases:
 - ObsProps
 - Properties
---

# Properties
add properties by listing them in the beginning of the file:
```yaml
---
<properties>
---
```

supported property types:
- **[[Obsidian#Text|Text]]**
- **[[Obsidian#Number|Number]]**
- **[[Obsidian#List|List]]**
- **[[Obsidian#Checkbox|Checkbox]]**
- **[[Obsidian#Date|Date]]**
- **[[Obsidian#Date & Time|Date & Time]]**

default properties:
- **[[Obsidian#Tags|Tags]]**
- 

## Supported Types

### Text
add a line of text as a property:
```yaml
---
title: properties
---
```

also add internal/external links as text properties:
```yaml
---
link: "[[Markdown]]"
url: https://help.obsidian.md/properties
---
```

### Number
add a literal number (no expressions), int and float are valid:
```yaml
---
year: 2026
python-version: 3.14
---
```

### List
use ` - ` before values to list them under the same tag. text, numbers and internal links are valid:
```yaml
---
family:
 - Dad
 - Mom
 - Sister
marks:
 - "[[Links]]"
 - "[[Text Formatting]]"
---
```

### Checkbox
use boolean values to apply checkbox:
```yaml
---
favorite: true
reply: false
last: # indeterminate treated as false
---
```

### Date
add dates with format `YYYY-MM-DD`:
```yaml
---
birthday: 2000-01-14
---
```

### Date & Time
add date & time with format `YYYY-MM-DD`T`HH:mm` :
```yaml
---
next-lesson: 2026-02-15T18:00
---
```

## Default Properties

### Tags
tag a page by adding a `#<tag>`
#le_tag
`#le_tag`
or by using properties:
```yaml
---
￼tags:
	- tagone
	- tagtwo
---
```

#### Nested Tags
make nested tags by adding a `/` in the tag.
#nest/chick
`#nest/chick`
the tag will be recognized as both the parent tag and the child.

### Aliases
add an alias to link to a whole note with:
```yaml
---
aliases:
 - ObsProps
 - Properties
---
```


### CssClasses
import a css snippet to apply to this note:
```yaml
---
cssclasses:
 - red-border
---
```

Obsidian can install [[Obsidian Plugins |plugins]], look further.