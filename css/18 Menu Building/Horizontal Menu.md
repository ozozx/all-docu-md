<html>
    <body>
    <div style="color: #000000; background-color: #FFFFFF; height: 4em;">
        <header>
            <ul style="margin:0; padding: 0; list-style-type: none; background-color: black; overflow: hidden;">
                <li style="float: left;"><a href="#" style="display: block; text-align: center; text-decoration: none; color:white; padding:10px 10px;">Home Page</a></li>
                <li style="float: left;"><a href="#" style="display: block; text-align: center; text-decoration: none; color:white; padding:10px 10px;">Products</a></li>
                <li style="float: left;"><a href="#" style="display: block; text-align: center; text-decoration: none; color:white; padding:10px 10px;">Prices</a></li>
                <li style="float: left;"><a href="#" style="display: block; text-align: center; text-decoration: none; color:white; padding:10px 10px;">About</a></li>
            </ul>
        </header>
    </div>
    </body>
</html>

```html
<!DOCTYPE html>
<html>
    <head>
        <style>
            .myNav{
                margin:0;
                padding: 0;
                list-style-type: none;
                background-color: black;
                overflow: hidden;
            }
            .myNav li{
                float: left;
            }
            .myNav li a{
                display: block;
                text-align: center;
                text-decoration: none;
                color:white;
                padding:10px 10px;
            }
            .myNav li a:hover{
                background-color: whitesmoke;
                color:black;
            }
        </style>
    </head>
    <body>
        <header>
            <ul class="myNav">
                <li><a href="#">Home Page</a></li>
                <li><a href="#">Products</a></li>
                <li><a href="#">Prices</a></li>
                <li><a href="#">About</a></li>
            </ul>
        </header>
    </body>
</html>
```