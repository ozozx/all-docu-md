<html>
    <body>
    <div style="color: #000000; background-color: #FFFFFF;">
        <div style="position: relative; width: 300px; height: 300px;">
            <img src="green.png" style="width:300px; height:300px;">
            <div style="position: absolute; left:0px; top:50%; width:100%; height: 100%; text-align: center; font-size: 18px;">
                Awesome Text
            </div>
            <div style="position: absolute; bottom: 10px; right:25px; font-size: 18px;">
                Another Text
            </div>
        </div>
	</div>
    </body>
</html>

```html
<!DOCTYPE html>
<html>
    <head>
        <style>
            .container{
                position: relative;
                width: 300px;
                height: 300px;
            }  
            .a{
                position: absolute;
                left:0px;
                top:50%;
                width:100%;
                height: 100%;
                text-align: center;
                font-size: 18px;
            }
            .b{
                position: absolute;
                bottom: 10px;
                right:25px;
                font-size: 18px;
            }
            img{
                width:300px;
                height:300px;
            }
        </style>
    </head>
    <body>
        <div class="container">
            <img src="green.png">
            <div class="a">
                Awesome Text
            </div>
            <div class="b">
                Another Text
            </div>
        </div>
    </body>
</html>
```