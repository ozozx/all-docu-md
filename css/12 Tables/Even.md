<html>
    <body>
    <div style="color: #000000; background-color: #FFFFFF;">
        <h1>My Css Table</h1>
        <table>
            <thead>
                <tr>
                    <th style="background:rgb(158, 16, 16); color:white;">Name</th>
                    <th style="background:rgb(158, 16, 16); color:white;">Profession</th>
                </tr>
            </thead>
            <tbody>
                <tr>
                    <td>Dan</td>
                    <td>Teacher</td>
                </tr>
                <tr style="background:#da0a0ab2;">
                    <td>Elizabeth</td>
                    <td>Programmer</td>
                </tr>
                <tr>
                    <td>Uzi</td>
                    <td>Chafuzi</td>
                </tr>
            </tbody>
        </table>
	</div>
    </body>
</html>

```html
<!DOCTYPE html>
<html>
    <head>
        <style>
            table{
                direction: ltr;
            }
            tr:nth-child(even){
                background:#da0a0ab2;
            }
            th{
               background:rgb(158, 16, 16);
               color:white;
            }
        </style>
    </head>
    <body>
        <h1>My Css Table</h1>
        <table>
            <thead>
                <tr>
                    <th>Name</th>
                    <th>Profession</th>
                </tr>
            </thead>
            <tbody>
                <tr>
                    <td>Dan</td>
                    <td>Teacher</td>
                </tr>
                <tr>
                    <td>Elizabeth</td>
                    <td>Programmer</td>
                </tr>
                <tr>
                    <td>Uzi</td>
                    <td>Chafuzi</td>
                </tr>
            </tbody>
        </table>
    </body>
</html>
```