<html>
    <body>
    <div style="color: #000000; background-color: #FFFFFF;">
        <h1>My Css Table</h1>
        <table style="border-collapse: collapse; border:1px solid blue;">
            <thead>
                <tr>
                    <th style="border:1px solid blue;">Name</th>
                    <th style="border:1px solid blue;">Profession</th>
                </tr>
            </thead>
            <tbody>
                <tr>
                    <td style="border:1px solid blue;">Dan</td>
                    <td style="border:1px solid blue;">Teacher</td>
                </tr>
                <tr>
                    <td style="border:1px solid blue;">Elizabeth</td>
                    <td style="border:1px solid blue;">Programmer</td>
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
                border-collapse: collapse;
            }
            table,th,td{
                border:1px solid blue;
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