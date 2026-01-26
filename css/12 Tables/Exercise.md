<html>
    <body>
    <div style="color: #000000; background-color: #FFFFFF;">
        <h1>My Css Table</h1>
        <table style="border:1px solid black;">
            <thead>
                <tr>
                    <th>Name</th>
                    <th>Profession</th>
                    <th>Age</th>
                </tr>
            </thead>
            <tbody>
                <tr style="background-color: #134ac0b2; color:white;">
                    <td>Dan</td>
                    <td>Teacher</td>
                    <td>28</td>
                </tr>
                <tr>
                    <td>Elizabeth</td>
                    <td>Programmer</td>
                    <td>30</td>
                </tr>
                <tr style="background-color: #134ac0b2; color:white;">
                    <td>Uzi</td>
                    <td>Chafuzi</td>
                    <td>12</td>
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
            tbody tr:nth-child(odd){
                background-color: #134ac0b2;
                color:white;
            }
            table{
                border:1px solid black;
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
                    <th>Age</th>
                </tr>
            </thead>
            <tbody>
                <tr>
                    <td>Dan</td>
                    <td>Teacher</td>
                    <td>28</td>
                </tr>
                <tr>
                    <td>Elizabeth</td>
                    <td>Programmer</td>
                    <td>30</td>
                </tr>
                <tr>
                    <td>Uzi</td>
                    <td>Chafuzi</td>
                    <td>12</td>
                </tr>
            </tbody>
        </table>
    </body>
</html>
```