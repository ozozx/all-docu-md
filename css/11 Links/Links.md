<html>
    <body>
    <div style="color: #000000; background-color: #FFFFFF;">
        <h1>Links</h1>
        <a href="https://appschoolfront.web.app" id="ls" style="color: purple; background-color: burlywood;">appschool</a>
	</div>
    </body>
</html>

```html
<!DOCTYPE html>
<html>
    <head>
        <style>
            a#ls {
                color: purple;
                background-color: burlywood;
            }
            /* unvisited link */
            a#ls:link {
                color: purple;
            }
            /* visited link */
            a#ls:visited {
                color: green;
                background-color: yellow;
            }
            /* mouse over link */
            a#ls:hover {
                color: brown;
                font-size: 30px;
            }
            /* selected link */
            a#ls:active {
                color: pink;
            }
        </style>
    </head>
    <body>
        <h1>Links</h1>
        <a href="https://appschoolfront.web.app" id="ls">appschool</a>
    </body>
</html>
```