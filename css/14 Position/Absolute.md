<html>
    <body>
    <div style="color: #000000; background-color: #FFFFFF;">
        <div style="width: 150px; position: relative; height:150px; border:2px solid rgb(173, 49, 33);">
            This is paragraph 
            <div style="position: absolute; top:30px; right:40px; width:100px; height: 50px; border:2px solid rgb(173, 49, 33);">
                    Awesome text 
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
           .content{
                width: 150px;
                position: relative; 
                height:150px;
                border:2px solid rgb(173, 49, 33);
           }
           .a{
            position: absolute;
            top:30px;
            right:40px;
            width:100px;
            height: 50px;
            border:2px solid rgb(173, 49, 33);
           } 
        </style>
    </head>
    <body>
        <div class="content">
            This is paragraph 
            <div class="a">
                    Awesome text 
            </div>
        </div>
    </body>
</html>
```