<html>
    <body>
    <div style="color: #000000; background-color: #FFFFFF;">
        <header>
            <h1>My Vertical Navigation Menu</h1>
            <ul style="margin:0; padding: 0; list-style-type: none; background-color: chocolate; width: 250px;">
                <li><a href="#" style="display: block; text-align: center; text-decoration: none; color:white; padding:10px 10px;">Home Page</a></li>
                <li><a href="#" style="display: block; text-align: center; text-decoration: none; color:white; padding:10px 10px;">Products</a></li>
                <li><a href="#" style="display: block; text-align: center; text-decoration: none; color:white; padding:10px 10px;">Prices</a></li>
                <li><a href="#" style="display: block; text-align: center; text-decoration: none; color:white; padding:10px 10px;">About</a></li>
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
                background-color: chocolate;
                width: 250px;
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
    <body dir="rtl">
        <header>
            <h1>My Vertical Navigation Menu</h1>
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