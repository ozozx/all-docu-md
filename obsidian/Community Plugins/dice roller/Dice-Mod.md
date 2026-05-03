It is possible to tell the plugin to replace the file contents of the note with the calculated dice roll using the `dice-mod: <formula>` syntax.

The plugin will replace the contents of the note with the syntax:

`<formula> -> <full results> -> <combined results>`

Example:

221 => `3d100 + 12 -> [75, 20, 75] + 12 -> 182`

## Displaying formula 

By default, the plugin will display the formula along with the result.

This can be turned off globally by turning off `Add Formula When Modifying` in settings, or by appending the ***[[Dice Flags|Dice Roller/Dice Flags]]*** `|noform` to a `dice-mod` roll.

## Replacing blocks 

If `dice-mod` is used on a ***[[Section Rollers]]***, the plugin will attempt to find a [[Internal links#Link to a block in a note|block id]] for the resulting section, so it can be embedded.

If a block id does not exist for that section, the plugin will attempt to create one for the section. This will modify the file being rolled.