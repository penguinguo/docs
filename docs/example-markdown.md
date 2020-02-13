# Example Markdown

This page shows some example markdown code

<!-- ----------------------------------------------------------------------- -->
## Cheat sheet
https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet

### Links

!!! example "External link example"
    ```no-highlight
    [LINK TEXT <i class="fa fa-external-link"></i>](LINK-URL){target="\_blank"}
    ```
    [LINK TEXT <i class="fa fa-external-link"></i>](LINK-URL){target="\_blank"}

### Images

!!! example "Image Example"
    ```no-highlight
    Inline-style: 
    ![alt text](http://via.placeholder.com/100x100 "Title Text 1")

    Reference-style: 
    ![alt text][logo]

    [logo]: http://via.placeholder.com/100x100 "Title Text 2"
    ```
    Inline-style: 
    ![alt text](http://via.placeholder.com/100x100 "Title Text 1")

    Reference-style: 
    ![alt text][logo]

[logo]: http://via.placeholder.com/100x100 "Title Text 2"


!!! example "Image with size constraint, e.g. 50% width"
    ```no-highlight
      ![ALT TEXT GOES HERE](http://via.placeholder.com/600x100){style="width:50%"}
    ```
![ALT TEXT GOES HERE](http://via.placeholder.com/600x100){style="width:50%"}


!!! example "Clickable image with figure text"
    ```no-highlight
      [![ALT TEXT GOES HERE](http://via.placeholder.com/600x100){style="width:50%"}](http://via.placeholder.com/600x100)
      <p style="text-align: center;">**Figure N:** Centered figure text.</p>
    ```
[![ALT TEXT GOES HERE](http://via.placeholder.com/600x100){style="width:50%"}](http://via.placeholder.com/600x100)
<p style="text-align: center;">**Figure N:** Centered figure text.</p>


<!-- ----------------------------------------------------------------------- -->
## Admonition
https://squidfunk.github.io/mkdocs-material/extensions/admonition/

```no-highlight
!!! note
    Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla et euismod.
!!! abstract
    Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla et euismod.
!!! question
    Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla et euismod.
!!! tip
    Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla et euismod.
!!! example
    Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla et euismod.
!!! success
    Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla et euismod.
!!! info
    Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla et euismod.
!!! warning
    Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla et euismod.
!!! danger
    Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla et euismod.
```

!!! note
    Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla et euismod.
!!! abstract
    Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla et euismod.
!!! question
    Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla et euismod.
!!! tip
    Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla et euismod.
!!! example
    Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla et euismod.
!!! success
    Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla et euismod.
!!! info
    Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla et euismod.
!!! warning
    Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla et euismod.
!!! danger
    Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla et euismod.


<!-- ----------------------------------------------------------------------- -->
## PyMdown
https://squidfunk.github.io/mkdocs-material/extensions/pymdown/



### Arithmatex
Show LaTeX style math.

!!! example "Arithmatex Example"
    ```no-highlight
    $$
    \frac{n!}{k!(n-k)!} = \binom{n}{k}
    $$

    Lorem ipsum dolor sit amet: $p(x|y) = \frac{p(y|x)p(x)}{p(y)}$
    ```

    $$
    \frac{n!}{k!(n-k)!} = \binom{n}{k}
    $$

    Lorem ipsum dolor sit amet: $p(x|y) = \frac{p(y|x)p(x)}{p(y)}$



### Mark

!!! example "Mark Example"
    ```no-highlight
    Used to ==highlight== text.
    ```
    Used to ==highlight== text.



### Tasklist

!!! example "Tasklist Example"
    ```no-highlight
    * [x] Lorem ipsum dolor sit amet, consectetur adipiscing elit
    * [ ] Vestibulum convallis sit amet nisi a tincidunt
        * [x] Sed egestas felis quis elit dapibus, ac aliquet turpis mattis
        * [ ] Praesent sed risus massa
    * [ ] Nulla vel eros venenatis, imperdiet enim id, faucibus nisi
    ```

    * [x] Lorem ipsum dolor sit amet, consectetur adipiscing elit
    * [ ] Vestibulum convallis sit amet nisi a tincidunt
        * [x] Sed egestas felis quis elit dapibus, ac aliquet turpis mattis
        * [ ] Praesent sed risus massa
    * [ ] Nulla vel eros venenatis, imperdiet enim id, faucibus nisi



### ProgressBar

!!! example "Progress Bar Example"
    ```no-highlight
    [=0% "0%"]
    [=5% "5%"]
    [=25% "25%"]
    [=45% "45%"]
    [=65% "65%"]
    [=85% "85%"]
    [=100% "100%"]
    ```

    [=0% "0%"]
    [=5% "5%"]
    [=25% "25%"]
    [=45% "45%"]
    [=65% "65%"]
    [=85% "85%"]
    [=100% "100%"]


<!-- ----------------------------------------------------------------------- -->
## Font Awesome
- http://bwmarrin.github.io/MkDocsPlus/fontawesome/
- https://astronautweb.co/snippet/font-awesome/

<i class="fa fa-camera-retro fa-lg"></i> fa-lg
<i class="fa fa-external-link fa-2x"></i> fa-2x
<i class="fa fa-sign-in fa-3x"></i> fa-3x
<i class="fa fa-upload fa-4x"></i> fa-4x
<i class="fa fa-check-square fa-5x"></i> fa-5x

