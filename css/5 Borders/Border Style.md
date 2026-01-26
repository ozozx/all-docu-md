<html>
    <body>
    <div style="background: #FFF;color: #000">
        <p style="border-style: dotted;width: 100px;">dotted border</p>
        <p style="border-style: dashed;width: 100px;">dashed border</p>
        <p style="border-style: solid;width: 100px;">solid border</p>
        <p style="border-style: double;width: 100px;">double border</p>
        <p style="border-style: groove;width: 100px;">groove border</p>
        <p style="border-style: ridge;width: 100px;">ridge border</p>
        <p style="border-style: inset;width: 100px;">inset border</p>
        <p style="border-style: outset;width: 100px;">outset border</p>
        <p style="border-top-style: none; border-right-style: groove; border-bottom-style: dotted; border-left-style: double; width: 100px;">this is mixed</p>
	</div>
    </body>
</html>

```html
<!DOCTYPE html>
<html>
    <head>
        <style>
            .dotted{
                border-style: dotted;width: 100px;
            }
            .dashed{
                border-style: dashed;width: 100px;
            }
            .solid{
                border-style: solid;width: 100px;
            }
            .double{
                border-style: double;width: 100px;
                }
            .groove{
                border-style: groove;width: 100px;
            }
            .ridge{
                border-style: ridge;width: 100px;
            }
            .inset{
                border-style: inset;width: 100px;
            }
            .outset{
                border-style: outset;width: 100px;
            }
            .mixed{
                border-top-style: none;width: 100px;
                border-right-style: groove;
                border-bottom-style: dotted;
                border-left-style: double;
            }
        </style>
    </head>
    <body>
        <p class="dotted">dotted border</p>
        <p class="dashed">dashed border</p>
        <p class="solid">solid border</p>
        <p class="double">double border</p>
        <p class="groove">groove border</p>
        <p class="ridge">ridge border</p>
        <p class="inset">inset border</p>
        <p class="outset">outset border</p>
        <p class="mixed">this is mixed</p>
    </body>
</html>
```