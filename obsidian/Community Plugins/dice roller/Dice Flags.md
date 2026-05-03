## Introduction 

Dice flags are essential parameters that modify the behavior of the dice roller tool. This documentation provides an overview of various flags that can be used to customize dice roll results.

## Nodice flag 

**Syntax**: `|nodice`

The `|nodice` flag disables the display of the dice button in roll results. This feature can also be turned off in settings.

`dice: 1d6|nodice`

## Form and noform flags 

**Syntax**: `|form` and `|noform`

The `|form` and `|noform` flags control whether the dice formula is shown when using dice modifiers.

`dice: 1d6|form`
`dice: 1d6|noform`

## Render and norender flags 

**Syntax**: `|render` and `|norender`

The `|render` flag displays dice results in the chat box, while the `|norender` flag suppresses dice results in the chat box.

`dice: 1d6|render`
`dice: 1d6|norender`

## Avg flag 

**Syntax**: `|avg`

The `|avg` flag calculates results using the average value of dice rolls. The tooltip displays this as an average. For example, `2d6+3|avg` results in `10`, with the tooltip indicating `average: [3.5,3.5]+3`. Subsequent rolls use random dice results unless Alt-click is used to reroll.

`dice: 2d6+3|avg|norender`

> [!warning] currently only works with `|norender`
> a [github issue](https://github.com/Obsidian-TTRPG-Community/dice-roller/issues/204) has been opened about it.

## None flag 

**Syntax**: `|none`

The `|none` flag initializes results with an empty value. The tooltip indicates "none." Subsequent rolls show random results unless Ctrl-click is used to reroll.

`dice:2d6|none|norender`

> [!warning] currently only works with `|norender`
> a [github issue](https://github.com/Obsidian-TTRPG-Community/dice-roller/issues/204) has been opened about it.

## Noparens flag 

**Syntax**: `|noparens`

The `|noparens` flag disables the display of dice rolls in parentheses (e.g., **(1d20)**) if the "Display Formula in Parenthesis After" setting is enabled.

`dice: 1d6|noparens`

## Text flag 

**Syntax**: `|text(my text)`

The `|text(my text)` flag replaces numeric results with user-defined text. Clicking the text triggers a roll, with the result only visible in the tooltip. For example, `1d20 + 2|nodice|text(Dexterity +2)` displays "Dexterity +2" and reveals a random saving throw result on hover.

`dice: 1d20 + 2|nodice|text(Dexterity +2)`

> [!faq]- Q1. Can I use formulas within `text(my text)` such as `text(1d20-4)`?
>
> At this time, Dice Roller does not support the ability for the text field to have its own formulas or conditionals.

> [!faq]- Q2. Is there a way to just add the button by itself without displaying the output or anything?
>
> Yes! `dice: 1d20|text( )`  
> There is a simple space within the `()` field. Different operating systems may have to use `""` instead.
