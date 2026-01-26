
<html>
    <body>
    <div style="background: #FFF;color: #000">
        <p style="border-style: solid; border-width: 10px;">solid border</p>
        <p style="border-style: solid;">solid border</p>
        <p style="border-style: solid; border-width: 5px;">solid border</p>
        <p style="border-style: solid; border-width: 1em;">solid border</p>
    </div>
    </body>
</html>

```html
<!DOCTYPE html>
<html>
    <head>
        <style>
            .first{
                border-style: solid;
                border-width: 10px;
            }
            .second{
                border-style: solid;
            }
            .third{
                border-style: solid;
                border-width: 5px;
            }
            .fourth{
                border-style: solid;
                border-width: 1em;
            }
        </style>
    </head>
    <body>
        <p class="first">solid border</p>
        <p class="second">solid border</p>
        <p class="third">solid border</p>
        <p class="fourth">solid border</p>
    </body>
</html>
```

<html>
    <body>
    <div style="background: #FFF;color: #000">
        <p style="border-style: solid; border-width: 2px 5px;">solid border</p>
        <p style="border-style: solid; border-width: 10px 3px;">solid border</p>
        <p style="border-style: solid; border-width: 2px 20px 5px 30px;">solid border</p>
    </div>
    </body>
</html>

```html
<!DOCTYPE html>
<html>
    <head>
        <style>
            .first{
                border-style: solid;
                border-width: 2px 5px;
            }
            .second{
                border-style: solid;
                border-width: 10px 3px;
            }
            .third{
                border-style: solid;
                border-width: 2px 20px 5px 30px;
            }
        </style>
    </head>
    <body>
        <p class="first">solid border</p>
        <p class="second">solid border</p>
        <p class="third">solid border</p>
    </body>
</html>
```