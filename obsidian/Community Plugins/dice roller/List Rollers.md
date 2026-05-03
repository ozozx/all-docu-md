The Dice Roller may also be given a link to a list in a note, which it will read and return a random result from the list.

In a note (such as `Note.md`), create a list & optionally give it a block id:

```markdown
-   a
-   b
-   c
-   d

^block-id
```

Then, in the dice formula, use a wikilink to the block reference of the table:

`​dice: [[Note^block-id]]`

example note:
![[example loot table]]

`dice: [[example loot table#^example-table1]]`

The plugin will read the list and return a random result.

To return multiple elements, use:

`​dice: Xd[[Note^block-id]]`

`dice: 2d[[example loot table#^example-table1]]`

Once in preview mode, you may Ctrl - click on the result to open the block reference in a new pane.
