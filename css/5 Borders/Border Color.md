<html>
    <body>
    <div style="background: #FFF;color: #000">
        <p style="border-style: dotted; border-color: brown;">I have a dotted border</p>
        <p style="border-style: solid; border-color: #a12290;">I have a solid border</p>
        <p style="border-style: dashed; border-color: rgb(255, 230, 0);">I have a dashed border</p>
    </div>
    </body>
</html>

```html
<!DOCTYPE html>
<html>
    <head>
        <style>
            .first{
                border-style: dotted;
                border-color: brown;
            }
            .second{
                border-style: solid;
                border-color: #a12290;
            }
            .third{
                border-style: dashed;
                border-color: rgb(255, 230, 0);
            }
        </style>
    </head>
    <body>
        <p class="first">I have a dotted border</p>
        <p class="second">I have a solid border</p>
        <p class="third">I have a dashed border</p>
    </body>
</html>
```