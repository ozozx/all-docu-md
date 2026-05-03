## Dice condition overview 

> [!info] **Feature**: This feature was added in version 6.0.0.

Conditional parameters are supported in dice rolls, allowing you to specify a set of requirements that the dice must meet to be included in the roll. If the dice meet the requirement, it will be considered a `1` (pass), and if it does not, it will be considered a `0` (failure).

### Dice rolls as their own conditionals 

> [!info] **Feature**: This Feature was added in version 8.15.0

To use a dice roll as a conditional, you will need to use a mathematical operator to define how the first roll should evaluate the second. All of the ***[[#N conditionals|N Conditionals]]*** are valid operators in this case.

> [!cite]- Screenshot of a Dice Roll as a Condition
> ![[IMG-Dice Conditions.png]]

### N conditionals 

You can also supply a "negative equals" condition; if the dice meet this requirement, it will be considered a `-1`. The following conditions are supported:

|Condition|Effect|
|---|---|
|`\={n}`|Only rolls that are equal to `{n}` are successful.|
|`​=!{n}`*|Only rolls that are not equal to `{n}` are successful.|
|`>{n}`|Only rolls that are greater than `{n}` are successful.|
|`<{n}`|Only rolls that are less than `{n}` are successful.|
|`>={n}`|Only rolls that are greater than or equal to `{n}` are successful.|
|`<={n}`|Only rolls that are less than or equal to `{n}` are successful.|
|`-={n}` or `​=-{n}`|Rolls equal to `{n}` will be considered -1.|

* Please note that `!={n}` is also a valid dice condition, however, it conflicts with the ***[[Explode]]*** modifier.

If you need to use this condition, you can replace `!={n}` with **=!{n}**.

## Modifier conditions 

The ***[[Explode]]***, ***[[Explode and Combine]]***, and ***[[Re-roll|Re-Roll]]*** modifiers each support an optional _condition_ operator, which changes the die rolls that the modifier is applied to.

The following conditions are supported:

|Condition|Effect|
|---|---|
|`\={n}`|Only rolls that are equal to `{n}` are modified.|
|`\=!{n}`|Only rolls that are not equal to `{n}` are modified.|
|`>{n}`|Only rolls that are greater than `{n}` are modified.|
|`<{n}`|Only rolls that are less than `{n}` are modified.|
|`>\={n}`|Only rolls that are greater than or equal to `{n}` are modified.|
|`<\={n}`|Only rolls that are less than or equal to `{n}` are modified.|

These conditions are fully chainable.

### Examples
|Formula|Description|Result|
|---|---|---|
|`​dice: 1d4!=3`|Explode rolls equal to 3|`[4, 2, 3!, 2] = 11`|
|`​dice: 1d4!i=!3`|Explode rolls not equal to 3 infinitely|`[4!, 2!, 3, 2!, 3, 2!, 1!, 4!, 3] = 24`|
|`​dice: 1d4r<3`|Re-roll rolls less than 3|`[4, 1, 2, 4] -> [4, 4r, 3r, 4] = 15`|
|`​dice: 1d4r<2>3`|Re-roll rolls less than 2, greater than 3|`[4, 1, 2, 4] -> [3r, 2r, 2, 2r] = 9`|
