<html>
    <body>
        <ul style="list-style-type:circle;">
            <li>Ziv</li> 
            <li>Roman</li>
        </ul>
        <ul style="list-style-type: square;">
            <li>Ziv</li> 
            <li>Roman</li>
        </ul>
        <ol style="list-style-type:upper-roman;">
            <li>Asaf</li> 
            <li>Gilad</li>
        </ol>
        <ol style="list-style-type:lower-greek;">
            <li>Asaf</li> 
            <li>Gilad</li>
        </ol>
    </body>
</html>

```html
<!DOCTYPE html>
<html>
    <head>
        <style>
            .one{
                list-style-type:circle;
            }
            .two{
                list-style-type:square;
            }
            .three{
                list-style-type:upper-roman;
            }
            .four{
                list-style-type:lower-greek;
            }
        </style>
    </head>
    <body>
        <ul class="one">
            <li>Ziv</li> 
            <li>Roman</li>
        </ul>
        <ul class="two">
            <li>Ziv</li> 
            <li>Roman</li>
        </ul>
        <ol class="three">
            <li>Asaf</li> 
            <li>Gilad</li>
        </ol>
        <ol class="four">
            <li>Asaf</li> 
            <li>Gilad</li>
        </ol>
    </body>
</html>
```