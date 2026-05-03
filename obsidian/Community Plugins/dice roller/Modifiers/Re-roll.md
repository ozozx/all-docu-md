**Syntax**: `XdXr{n|i}`

Re-roll a minimum dice. If `{n}` or `{i}` is provided, it will continue re-rolling until a number greater than the minimum is rolled, or `{n}` attempts have been made.

Re-rolled dice _replace_ their original roll, unlike explode, which _add_ new rolls.

Re-rolled dice will display as `Xr` in the tooltip.

## Examples 

|Formula|Result|
|---|---|
|`​dice: 2d20r`|`[7r, 18] = 15`|
|`​dice: 2d4r3`|`[3, 3r] = 6`|
|`​dice: 1d2ri`|`[2r] = 2`|
`dice: 2d20r`
`dice: 2d4r3`
`dice: 1d2ri`
