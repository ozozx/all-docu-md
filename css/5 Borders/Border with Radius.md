<html>
    <body>
    <div style="background: #FFF;color: #000">
        <p style="width: 150px; text-align: center; border: 2px solid #aabbcc;">radius</p>
        <p style="width: 150px; text-align: center; border: 5px solid #749dc7; border-radius: 5px;">radius</p>
        <p style="width: 150px; text-align: center; border: 2px solid #356ba0; border-radius: 10px;">radius</p>
        <p style="width: 150px; text-align: center; border: 5px solid #1a446e; border-radius: 20px;">radius</p>
	</div>
    </body>
</html>

```html
<!DOCTYPE html>
<html>
    <head>
        <style>
            p{
                width: 150px;
                text-align: center;
            }
            .first{
                border: 2px solid #aabbcc;
            }
            .second{
                border: 5px solid #749dc7;
                border-radius: 5px;
            }
            .third{
                border: 2px solid #356ba0;
                border-radius: 10px;
            }
            .fourth{
                border: 5px solid #1a446e;
                border-radius: 20px;
            }
        </style>
    </head>
    <body>
        <p class="first">radius</p>
        <p class="second">radius</p>
        <p class="third">radius</p>
        <p class="fourth">radius</p>
    </body>
</html>
```