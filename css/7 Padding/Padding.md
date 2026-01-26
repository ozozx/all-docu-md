<html>
    <body>
    <div style="color: #000000; background-color: #FFFFFF;">
        <div style="padding:10px; border:1px solid green;">
            <p>I am padding</p>
        </div>
        <div style="padding-top: 30px; padding-bottom: 10px; padding-right: 20px; padding-left: 50px; border: 1px solid black;">
            <p>hello world</p>
        </div>
        <div style="padding:30px 20px 10px 50px; border: 1px solid blue;">
            <p>hello blue</p>
        </div>
	</div>
    </body>
</html>

```html
<!DOCTYPE html>
<html>
    <head>
        <style>
            div{
                padding:10px;
                border:1px solid green;
            }
            .mydiv{
                padding-top: 30px;
                padding-bottom: 10px;
                padding-right: 20px;
                padding-left: 50px;
                border: 1px solid black;
            }
            .mydiv2{
                padding:30px 20px 10px 50px;
                border: 1px solid blue;
            }
        </style>
    </head>
    <body>
        <div>
            <p>I am padding</p>
        </div>
        <div class="mydiv">
            <p>hello world</p>
        </div>
        <div class="mydiv2">
            <p>hello blue</p>
        </div>
    </body>
</html>
```