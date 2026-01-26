<html>
    <body>
    <div style="color: #000000; background-color: #FFFFFF;">
        <h1>My Css Table</h1>
        <table>
            <thead>
                <tr style="background-color: #f5f5f5;">
                    <th style="border-bottom:2px solid #ddd;">Name</th>
                    <th style="border-bottom:2px solid #ddd;">Profession</th>
                </tr>
            </thead>
            <tbody>
                <tr style="background-color: #f5f5f5;">
                    <td style="border-bottom:2px solid #ddd;">Dan</td>
                    <td style="border-bottom:2px solid #ddd;">Teacher</td>
                </tr>
                <tr style="background-color: #f5f5f5;">
                    <td style="border-bottom:2px solid #ddd;">Elizabeth</td>
                    <td style="border-bottom:2px solid #ddd;">Programmer</td>
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
            th,td{
                border-bottom:2px solid #ddd;
            }
            tr:hover{
                background-color: #f5f5f5;
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
            </tbody>
        </table>
    </body>
</html>
```