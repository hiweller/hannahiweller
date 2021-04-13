---
title: Modifying pixel plots
author: Hannah Weller
date: '2021-04-13'
slug: modifying-pixel-plots
categories: [colordistance, color, r packages]
tags: [colordistance, color, r packages, plotting]
subtitle: ''
summary: ''
authors: []
lastmod: '2021-04-13T12:06:09-04:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---

<script src="{{< blogdown/postref >}}index_files/htmlwidgets/htmlwidgets.js"></script>

<script src="{{< blogdown/postref >}}index_files/plotly-binding/plotly.js"></script>

<script src="{{< blogdown/postref >}}index_files/typedarray/typedarray.min.js"></script>

<script src="{{< blogdown/postref >}}index_files/jquery/jquery.min.js"></script>

<link href="{{< blogdown/postref >}}index_files/crosstalk/css/crosstalk.css" rel="stylesheet" />

<script src="{{< blogdown/postref >}}index_files/crosstalk/js/crosstalk.min.js"></script>

<link href="{{< blogdown/postref >}}index_files/plotly-htmlwidgets-css/plotly-htmlwidgets.css" rel="stylesheet" />

<script src="{{< blogdown/postref >}}index_files/plotly-main/plotly-latest.min.js"></script>

<script src="{{< blogdown/postref >}}index_files/htmlwidgets/htmlwidgets.js"></script>

<script src="{{< blogdown/postref >}}index_files/plotly-binding/plotly.js"></script>

<script src="{{< blogdown/postref >}}index_files/typedarray/typedarray.min.js"></script>

<script src="{{< blogdown/postref >}}index_files/jquery/jquery.min.js"></script>

<link href="{{< blogdown/postref >}}index_files/crosstalk/css/crosstalk.css" rel="stylesheet" />

<script src="{{< blogdown/postref >}}index_files/crosstalk/js/crosstalk.min.js"></script>

<link href="{{< blogdown/postref >}}index_files/plotly-htmlwidgets-css/plotly-htmlwidgets.css" rel="stylesheet" />

<script src="{{< blogdown/postref >}}index_files/plotly-main/plotly-latest.min.js"></script>

The `plotPixels` function in `colordistance` is pretty inflexible. It was originally meant as a diagnostic tool, and the plots it produces are not exactly beautiful:

``` r
library(colordistance)

# image from the 'recolorize' package (github.com/hiweller/recolorize)
img <- system.file("extdata/fulgidissima.png", package = "recolorize")

# load the image:
loaded_img <- loadImage(img)

# set the plot layout for opposing pixel plots
layout(matrix(1:3, nrow = 1), widths = c(0.46, 0.08, 0.46))

# plot the pixels in RGB color space from two angles:
plotPixels(loaded_img)

# plot the original image
par(mar = rep(0, 4)) # no margin
plotImage(loaded_img)

# and pixels from the opposite angle:
plotPixels(loaded_img, angle = -45)
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-1-1.png" width="672" />

These plots are certainly *fine* if you want to scope out the color distribution in the image, but I wouldn’t want to display them for communication: the axis text is too large and some of the tick marks overlap; the axis labels are oddly spaced; and depending on the intention of the graphic, I might not want the grid or the plot frame. The axis label thing in particular has always bothered me.

Some of those changes are possible to make by passing additional parameters to the `plotPixels` function itself, but in practice, I often want more flexibility than this provides. Luckily, the function itself has such simple building blocks that it’s pretty easy to unpack them to get more customized plots.

This is how `plotPixels` works:

1.  It takes a dataframe of RGB colors, where pixels are rows and color channels are columns.
2.  It creates a vector of hex codes from the RGB colors to tell R which color to make each point.
3.  It uses [`scatterplot3d`](https://www.econstor.eu/handle/10419/77160) to plot in the 3D color space indicated with the `color.space` argument.

I chose the `scatterplot3d` package because, of all the 3D plotting packages, it’s the most lightweight, and more or less just extends the base plotting syntax. It was also written in 2003, so there are a lot of newer packages that provide prettier output and more options, like [plot3D](https://cran.r-project.org/web/packages/plot3D/index.html) by Karline Soetaert, or the [plotly](https://cran.r-project.org/web/packages/plotly/) library.

``` r
# load the plot3D library
library(plot3D)

# get the RGB pixel matrix
pixels <- loaded_img$filtered.rgb.2d

# make the hex color vector using the rgb() function
color_vector <- rgb(pixels); head(color_vector) # just a bunch of hex codes!
```

    ## [1] "#247872" "#006862" "#006B62" "#00776A" "#00645C" "#007B71"

``` r
# use the scatter3D function
scatter3D(x = pixels[ , 1], 
          y = pixels[ , 2],
          z = pixels[ , 3], 
          colvar = 1:nrow(pixels), # <- note we have to make a fake 'variable' to assign each pixel a different color
          col = color_vector, 
          colkey = FALSE, # gets rid of the (in this case meaningless) legend
          xlab = "Red", ylab = "Green", zlab = "Blue")
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-2-1.png" width="672" />

Even the default `scatter3D` plot looks a lot better to me: the axis labels hug the axes, and the angle is nicer. We can get fancier with a lot of the options, too:

``` r
scatter3D(x = pixels[ , 1], 
          y = pixels[ , 2],
          z = pixels[ , 3], 
          colvar = 1:nrow(pixels), 
          col = color_vector, colkey = F,
          xlab = "Red", ylab = "Green", zlab = "Blue",
          xlim = 0:1, ylim = 0:1, zlim = 0:1, # RGB max and min
          pch = 19, # filled circles
          alpha = 0.5, # partially transparent
          theta = 115, phi = 25, # change viewing angle
          bty = "bl2") # black grid background looks sort of cool
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-3-1.png" width="672" />

What if you want to plot in another color space besides RGB? The only difference is that you have to first convert your pixel matrix to a given color space, for which you have several options.

``` r
# convert pixels to CIE Lab coordinates
pixels_lab <- convertColor(pixels, from = "sRGB", to = "Lab")

# color vector remains the same!
color_vector <- rgb(pixels)

scatter3D(x = pixels_lab[ , 1], 
          y = pixels_lab[ , 2],
          z = pixels_lab[ , 3], 
          colvar = 1:nrow(pixels_lab), 
          col = color_vector, colkey = F,
          xlab = "Luminance", ylab = "a (red-green)", zlab = "b (yellow-blue)",
          theta = 120, phi = -5,
          xlim = c(0, 100), 
          pch = 19, # filled circles
          alpha = 0.5, # partially transparent
          bty = "b2")
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-4-1.png" width="672" />

As an aside, it’s good practice to set the axis limits thoughtfully. This is easy with RGB: all three channels have a 0-1 range. With CIE Lab, this depends on your reference white. The L channel will always be 0-100, and the outer limits for the a and b channels are -127 to 128 each, but for a given reference white converting from sRGB it will be a subset within that range. The axis limits will be set to the range of the data by default, which could be misleading if you’re comparing plots of multiple images.

If you’d rather have an interactive plot (especially helpful for data exploration), you can use the `plotly` package. I find I have to implement more workarounds to get these plots to behave how I’d expect, but once you get out an interactive plot, it’s pretty slick:

``` r
library(plotly, quietly = TRUE)

# let's subsample down to 100 pixels just for this example
pixel_sub <- as.data.frame(pixels[sample(1:nrow(pixels), 100), ])
plotly_colors <- rgb(pixel_sub)

# and plot!
plot_ly(data = pixel_sub, 
        x = ~r, y = ~g, z = ~b, 
        type = "scatter3d", mode = "markers", 
        color = I(plotly_colors), # this is a bit of a hack and you'll get a warning...
        colors = plotly_colors)
```

<div id="htmlwidget-1" style="width:672px;height:480px;" class="plotly html-widget"></div>
<script type="application/json" data-for="htmlwidget-1">{"x":{"visdat":{"67c16a7081ae":["function () ","plotlyVisDat"]},"cur_data":"67c16a7081ae","attrs":{"67c16a7081ae":{"x":{},"y":{},"z":{},"mode":"markers","color":["#00766C","#33A000","#008B54","#5D9926","#616028","#9A1E26","#88AD00","#4E3737","#AF5D04","#911B2F","#60A000","#6D222D","#3D712D","#008F42","#00816F","#00614E","#9A9B00","#C44B44","#6A2728","#64762C","#736B68","#007A87","#479F40","#818000","#5D4B35","#00CDAA","#6F1E27","#478500","#9C2529","#7F322D","#6D2128","#D25F17","#57590E","#72692E","#999000","#E13E1D","#00A64D","#10529E","#009D3E","#3E9000","#592128","#CE5D0F","#526437","#4C3F39","#2B7523","#24CF9E","#A37008","#1EADBF","#369632","#E14231","#82222A","#62232F","#932E23","#32931F","#653031","#77202C","#8C5712","#007C84","#B7521C","#3D5342","#008355","#A27900","#9A9500","#008860","#5D7006","#8D1A21","#00604F","#005986","#A62B2D","#628B00","#008572","#29317B","#296D36","#008E54","#285D1D","#565F34","#A61835","#677606","#DA4428","#53824B","#5E8E0B","#00894A","#006B43","#008276","#007C78","#B5681E","#00885E","#7A8100","#E0512F","#382737","#009449","#719200","#408C2D","#005B70","#00A384","#614244","#2B9737","#07982F","#578E00","#61BF37"],"colors":["#00766C","#33A000","#008B54","#5D9926","#616028","#9A1E26","#88AD00","#4E3737","#AF5D04","#911B2F","#60A000","#6D222D","#3D712D","#008F42","#00816F","#00614E","#9A9B00","#C44B44","#6A2728","#64762C","#736B68","#007A87","#479F40","#818000","#5D4B35","#00CDAA","#6F1E27","#478500","#9C2529","#7F322D","#6D2128","#D25F17","#57590E","#72692E","#999000","#E13E1D","#00A64D","#10529E","#009D3E","#3E9000","#592128","#CE5D0F","#526437","#4C3F39","#2B7523","#24CF9E","#A37008","#1EADBF","#369632","#E14231","#82222A","#62232F","#932E23","#32931F","#653031","#77202C","#8C5712","#007C84","#B7521C","#3D5342","#008355","#A27900","#9A9500","#008860","#5D7006","#8D1A21","#00604F","#005986","#A62B2D","#628B00","#008572","#29317B","#296D36","#008E54","#285D1D","#565F34","#A61835","#677606","#DA4428","#53824B","#5E8E0B","#00894A","#006B43","#008276","#007C78","#B5681E","#00885E","#7A8100","#E0512F","#382737","#009449","#719200","#408C2D","#005B70","#00A384","#614244","#2B9737","#07982F","#578E00","#61BF37"],"alpha_stroke":1,"sizes":[10,100],"spans":[1,20],"type":"scatter3d"}},"layout":{"margin":{"b":40,"l":60,"t":25,"r":10},"scene":{"xaxis":{"title":"r"},"yaxis":{"title":"g"},"zaxis":{"title":"b"}},"hovermode":"closest","showlegend":false},"source":"A","config":{"showSendToCloud":false},"data":[{"x":[0,0.2,0,0.364705882352941,0.380392156862745,0.603921568627451,0.533333333333333,0.305882352941176,0.686274509803922,0.568627450980392,0.376470588235294,0.427450980392157,0.23921568627451,0,0,0,0.603921568627451,0.768627450980392,0.415686274509804,0.392156862745098,0.450980392156863,0,0.27843137254902,0.505882352941176,0.364705882352941,0,0.435294117647059,0.27843137254902,0.611764705882353,0.498039215686275,0.427450980392157,0.823529411764706,0.341176470588235,0.447058823529412,0.6,0.882352941176471,0,0.0627450980392157,0,0.243137254901961,0.349019607843137,0.807843137254902,0.32156862745098,0.298039215686275,0.168627450980392,0.141176470588235,0.63921568627451,0.117647058823529,0.211764705882353,0.882352941176471,0.509803921568627,0.384313725490196,0.576470588235294,0.196078431372549,0.396078431372549,0.466666666666667,0.549019607843137,0,0.717647058823529,0.23921568627451,0,0.635294117647059,0.603921568627451,0,0.364705882352941,0.552941176470588,0,0,0.650980392156863,0.384313725490196,0,0.16078431372549,0.16078431372549,0,0.156862745098039,0.337254901960784,0.650980392156863,0.403921568627451,0.854901960784314,0.325490196078431,0.368627450980392,0,0,0,0,0.709803921568627,0,0.47843137254902,0.87843137254902,0.219607843137255,0,0.443137254901961,0.250980392156863,0,0,0.380392156862745,0.168627450980392,0.0274509803921569,0.341176470588235,0.380392156862745],"y":[0.462745098039216,0.627450980392157,0.545098039215686,0.6,0.376470588235294,0.117647058823529,0.67843137254902,0.215686274509804,0.364705882352941,0.105882352941176,0.627450980392157,0.133333333333333,0.443137254901961,0.56078431372549,0.505882352941176,0.380392156862745,0.607843137254902,0.294117647058824,0.152941176470588,0.462745098039216,0.419607843137255,0.47843137254902,0.623529411764706,0.501960784313725,0.294117647058824,0.803921568627451,0.117647058823529,0.52156862745098,0.145098039215686,0.196078431372549,0.129411764705882,0.372549019607843,0.349019607843137,0.411764705882353,0.564705882352941,0.243137254901961,0.650980392156863,0.32156862745098,0.615686274509804,0.564705882352941,0.129411764705882,0.364705882352941,0.392156862745098,0.247058823529412,0.458823529411765,0.811764705882353,0.43921568627451,0.67843137254902,0.588235294117647,0.258823529411765,0.133333333333333,0.137254901960784,0.180392156862745,0.576470588235294,0.188235294117647,0.125490196078431,0.341176470588235,0.486274509803922,0.32156862745098,0.325490196078431,0.513725490196078,0.474509803921569,0.584313725490196,0.533333333333333,0.43921568627451,0.101960784313725,0.376470588235294,0.349019607843137,0.168627450980392,0.545098039215686,0.52156862745098,0.192156862745098,0.427450980392157,0.556862745098039,0.364705882352941,0.372549019607843,0.0941176470588235,0.462745098039216,0.266666666666667,0.509803921568627,0.556862745098039,0.537254901960784,0.419607843137255,0.509803921568627,0.486274509803922,0.407843137254902,0.533333333333333,0.505882352941176,0.317647058823529,0.152941176470588,0.580392156862745,0.572549019607843,0.549019607843137,0.356862745098039,0.63921568627451,0.258823529411765,0.592156862745098,0.596078431372549,0.556862745098039,0.749019607843137],"z":[0.423529411764706,0,0.329411764705882,0.149019607843137,0.156862745098039,0.149019607843137,0,0.215686274509804,0.0156862745098039,0.184313725490196,0,0.176470588235294,0.176470588235294,0.258823529411765,0.435294117647059,0.305882352941176,0,0.266666666666667,0.156862745098039,0.172549019607843,0.407843137254902,0.529411764705882,0.250980392156863,0,0.207843137254902,0.666666666666667,0.152941176470588,0,0.16078431372549,0.176470588235294,0.156862745098039,0.0901960784313725,0.0549019607843137,0.180392156862745,0,0.113725490196078,0.301960784313725,0.619607843137255,0.243137254901961,0,0.156862745098039,0.0588235294117647,0.215686274509804,0.223529411764706,0.137254901960784,0.619607843137255,0.0313725490196078,0.749019607843137,0.196078431372549,0.192156862745098,0.164705882352941,0.184313725490196,0.137254901960784,0.12156862745098,0.192156862745098,0.172549019607843,0.0705882352941176,0.517647058823529,0.109803921568627,0.258823529411765,0.333333333333333,0,0,0.376470588235294,0.0235294117647059,0.129411764705882,0.309803921568627,0.525490196078431,0.176470588235294,0,0.447058823529412,0.482352941176471,0.211764705882353,0.329411764705882,0.113725490196078,0.203921568627451,0.207843137254902,0.0235294117647059,0.156862745098039,0.294117647058824,0.0431372549019608,0.290196078431373,0.262745098039216,0.462745098039216,0.470588235294118,0.117647058823529,0.368627450980392,0,0.184313725490196,0.215686274509804,0.286274509803922,0,0.176470588235294,0.43921568627451,0.517647058823529,0.266666666666667,0.215686274509804,0.184313725490196,0,0.215686274509804],"mode":"markers","type":"scatter3d","marker":{"color":["rgba(0,118,108,1)","rgba(51,160,0,1)","rgba(0,139,84,1)","rgba(93,153,38,1)","rgba(97,96,40,1)","rgba(154,30,38,1)","rgba(136,173,0,1)","rgba(78,55,55,1)","rgba(175,93,4,1)","rgba(145,27,47,1)","rgba(96,160,0,1)","rgba(109,34,45,1)","rgba(61,113,45,1)","rgba(0,143,66,1)","rgba(0,129,111,1)","rgba(0,97,78,1)","rgba(154,155,0,1)","rgba(196,75,68,1)","rgba(106,39,40,1)","rgba(100,118,44,1)","rgba(115,107,104,1)","rgba(0,122,135,1)","rgba(71,159,64,1)","rgba(129,128,0,1)","rgba(93,75,53,1)","rgba(0,205,170,1)","rgba(111,30,39,1)","rgba(71,133,0,1)","rgba(156,37,41,1)","rgba(127,50,45,1)","rgba(109,33,40,1)","rgba(210,95,23,1)","rgba(87,89,14,1)","rgba(114,105,46,1)","rgba(153,144,0,1)","rgba(225,62,29,1)","rgba(0,166,77,1)","rgba(16,82,158,1)","rgba(0,157,62,1)","rgba(62,144,0,1)","rgba(89,33,40,1)","rgba(206,93,15,1)","rgba(82,100,55,1)","rgba(76,63,57,1)","rgba(43,117,35,1)","rgba(36,207,158,1)","rgba(163,112,8,1)","rgba(30,173,191,1)","rgba(54,150,50,1)","rgba(225,66,49,1)","rgba(130,34,42,1)","rgba(98,35,47,1)","rgba(147,46,35,1)","rgba(50,147,31,1)","rgba(101,48,49,1)","rgba(119,32,44,1)","rgba(140,87,18,1)","rgba(0,124,132,1)","rgba(183,82,28,1)","rgba(61,83,66,1)","rgba(0,131,85,1)","rgba(162,121,0,1)","rgba(154,149,0,1)","rgba(0,136,96,1)","rgba(93,112,6,1)","rgba(141,26,33,1)","rgba(0,96,79,1)","rgba(0,89,134,1)","rgba(166,43,45,1)","rgba(98,139,0,1)","rgba(0,133,114,1)","rgba(41,49,123,1)","rgba(41,109,54,1)","rgba(0,142,84,1)","rgba(40,93,29,1)","rgba(86,95,52,1)","rgba(166,24,53,1)","rgba(103,118,6,1)","rgba(218,68,40,1)","rgba(83,130,75,1)","rgba(94,142,11,1)","rgba(0,137,74,1)","rgba(0,107,67,1)","rgba(0,130,118,1)","rgba(0,124,120,1)","rgba(181,104,30,1)","rgba(0,136,94,1)","rgba(122,129,0,1)","rgba(224,81,47,1)","rgba(56,39,55,1)","rgba(0,148,73,1)","rgba(113,146,0,1)","rgba(64,140,45,1)","rgba(0,91,112,1)","rgba(0,163,132,1)","rgba(97,66,68,1)","rgba(43,151,55,1)","rgba(7,152,47,1)","rgba(87,142,0,1)","rgba(97,191,55,1)"],"line":{"color":["rgba(0,118,108,1)","rgba(51,160,0,1)","rgba(0,139,84,1)","rgba(93,153,38,1)","rgba(97,96,40,1)","rgba(154,30,38,1)","rgba(136,173,0,1)","rgba(78,55,55,1)","rgba(175,93,4,1)","rgba(145,27,47,1)","rgba(96,160,0,1)","rgba(109,34,45,1)","rgba(61,113,45,1)","rgba(0,143,66,1)","rgba(0,129,111,1)","rgba(0,97,78,1)","rgba(154,155,0,1)","rgba(196,75,68,1)","rgba(106,39,40,1)","rgba(100,118,44,1)","rgba(115,107,104,1)","rgba(0,122,135,1)","rgba(71,159,64,1)","rgba(129,128,0,1)","rgba(93,75,53,1)","rgba(0,205,170,1)","rgba(111,30,39,1)","rgba(71,133,0,1)","rgba(156,37,41,1)","rgba(127,50,45,1)","rgba(109,33,40,1)","rgba(210,95,23,1)","rgba(87,89,14,1)","rgba(114,105,46,1)","rgba(153,144,0,1)","rgba(225,62,29,1)","rgba(0,166,77,1)","rgba(16,82,158,1)","rgba(0,157,62,1)","rgba(62,144,0,1)","rgba(89,33,40,1)","rgba(206,93,15,1)","rgba(82,100,55,1)","rgba(76,63,57,1)","rgba(43,117,35,1)","rgba(36,207,158,1)","rgba(163,112,8,1)","rgba(30,173,191,1)","rgba(54,150,50,1)","rgba(225,66,49,1)","rgba(130,34,42,1)","rgba(98,35,47,1)","rgba(147,46,35,1)","rgba(50,147,31,1)","rgba(101,48,49,1)","rgba(119,32,44,1)","rgba(140,87,18,1)","rgba(0,124,132,1)","rgba(183,82,28,1)","rgba(61,83,66,1)","rgba(0,131,85,1)","rgba(162,121,0,1)","rgba(154,149,0,1)","rgba(0,136,96,1)","rgba(93,112,6,1)","rgba(141,26,33,1)","rgba(0,96,79,1)","rgba(0,89,134,1)","rgba(166,43,45,1)","rgba(98,139,0,1)","rgba(0,133,114,1)","rgba(41,49,123,1)","rgba(41,109,54,1)","rgba(0,142,84,1)","rgba(40,93,29,1)","rgba(86,95,52,1)","rgba(166,24,53,1)","rgba(103,118,6,1)","rgba(218,68,40,1)","rgba(83,130,75,1)","rgba(94,142,11,1)","rgba(0,137,74,1)","rgba(0,107,67,1)","rgba(0,130,118,1)","rgba(0,124,120,1)","rgba(181,104,30,1)","rgba(0,136,94,1)","rgba(122,129,0,1)","rgba(224,81,47,1)","rgba(56,39,55,1)","rgba(0,148,73,1)","rgba(113,146,0,1)","rgba(64,140,45,1)","rgba(0,91,112,1)","rgba(0,163,132,1)","rgba(97,66,68,1)","rgba(43,151,55,1)","rgba(7,152,47,1)","rgba(87,142,0,1)","rgba(97,191,55,1)"]}},"textfont":{"color":["rgba(0,118,108,1)","rgba(51,160,0,1)","rgba(0,139,84,1)","rgba(93,153,38,1)","rgba(97,96,40,1)","rgba(154,30,38,1)","rgba(136,173,0,1)","rgba(78,55,55,1)","rgba(175,93,4,1)","rgba(145,27,47,1)","rgba(96,160,0,1)","rgba(109,34,45,1)","rgba(61,113,45,1)","rgba(0,143,66,1)","rgba(0,129,111,1)","rgba(0,97,78,1)","rgba(154,155,0,1)","rgba(196,75,68,1)","rgba(106,39,40,1)","rgba(100,118,44,1)","rgba(115,107,104,1)","rgba(0,122,135,1)","rgba(71,159,64,1)","rgba(129,128,0,1)","rgba(93,75,53,1)","rgba(0,205,170,1)","rgba(111,30,39,1)","rgba(71,133,0,1)","rgba(156,37,41,1)","rgba(127,50,45,1)","rgba(109,33,40,1)","rgba(210,95,23,1)","rgba(87,89,14,1)","rgba(114,105,46,1)","rgba(153,144,0,1)","rgba(225,62,29,1)","rgba(0,166,77,1)","rgba(16,82,158,1)","rgba(0,157,62,1)","rgba(62,144,0,1)","rgba(89,33,40,1)","rgba(206,93,15,1)","rgba(82,100,55,1)","rgba(76,63,57,1)","rgba(43,117,35,1)","rgba(36,207,158,1)","rgba(163,112,8,1)","rgba(30,173,191,1)","rgba(54,150,50,1)","rgba(225,66,49,1)","rgba(130,34,42,1)","rgba(98,35,47,1)","rgba(147,46,35,1)","rgba(50,147,31,1)","rgba(101,48,49,1)","rgba(119,32,44,1)","rgba(140,87,18,1)","rgba(0,124,132,1)","rgba(183,82,28,1)","rgba(61,83,66,1)","rgba(0,131,85,1)","rgba(162,121,0,1)","rgba(154,149,0,1)","rgba(0,136,96,1)","rgba(93,112,6,1)","rgba(141,26,33,1)","rgba(0,96,79,1)","rgba(0,89,134,1)","rgba(166,43,45,1)","rgba(98,139,0,1)","rgba(0,133,114,1)","rgba(41,49,123,1)","rgba(41,109,54,1)","rgba(0,142,84,1)","rgba(40,93,29,1)","rgba(86,95,52,1)","rgba(166,24,53,1)","rgba(103,118,6,1)","rgba(218,68,40,1)","rgba(83,130,75,1)","rgba(94,142,11,1)","rgba(0,137,74,1)","rgba(0,107,67,1)","rgba(0,130,118,1)","rgba(0,124,120,1)","rgba(181,104,30,1)","rgba(0,136,94,1)","rgba(122,129,0,1)","rgba(224,81,47,1)","rgba(56,39,55,1)","rgba(0,148,73,1)","rgba(113,146,0,1)","rgba(64,140,45,1)","rgba(0,91,112,1)","rgba(0,163,132,1)","rgba(97,66,68,1)","rgba(43,151,55,1)","rgba(7,152,47,1)","rgba(87,142,0,1)","rgba(97,191,55,1)"]},"line":{"color":["rgba(0,118,108,1)","rgba(51,160,0,1)","rgba(0,139,84,1)","rgba(93,153,38,1)","rgba(97,96,40,1)","rgba(154,30,38,1)","rgba(136,173,0,1)","rgba(78,55,55,1)","rgba(175,93,4,1)","rgba(145,27,47,1)","rgba(96,160,0,1)","rgba(109,34,45,1)","rgba(61,113,45,1)","rgba(0,143,66,1)","rgba(0,129,111,1)","rgba(0,97,78,1)","rgba(154,155,0,1)","rgba(196,75,68,1)","rgba(106,39,40,1)","rgba(100,118,44,1)","rgba(115,107,104,1)","rgba(0,122,135,1)","rgba(71,159,64,1)","rgba(129,128,0,1)","rgba(93,75,53,1)","rgba(0,205,170,1)","rgba(111,30,39,1)","rgba(71,133,0,1)","rgba(156,37,41,1)","rgba(127,50,45,1)","rgba(109,33,40,1)","rgba(210,95,23,1)","rgba(87,89,14,1)","rgba(114,105,46,1)","rgba(153,144,0,1)","rgba(225,62,29,1)","rgba(0,166,77,1)","rgba(16,82,158,1)","rgba(0,157,62,1)","rgba(62,144,0,1)","rgba(89,33,40,1)","rgba(206,93,15,1)","rgba(82,100,55,1)","rgba(76,63,57,1)","rgba(43,117,35,1)","rgba(36,207,158,1)","rgba(163,112,8,1)","rgba(30,173,191,1)","rgba(54,150,50,1)","rgba(225,66,49,1)","rgba(130,34,42,1)","rgba(98,35,47,1)","rgba(147,46,35,1)","rgba(50,147,31,1)","rgba(101,48,49,1)","rgba(119,32,44,1)","rgba(140,87,18,1)","rgba(0,124,132,1)","rgba(183,82,28,1)","rgba(61,83,66,1)","rgba(0,131,85,1)","rgba(162,121,0,1)","rgba(154,149,0,1)","rgba(0,136,96,1)","rgba(93,112,6,1)","rgba(141,26,33,1)","rgba(0,96,79,1)","rgba(0,89,134,1)","rgba(166,43,45,1)","rgba(98,139,0,1)","rgba(0,133,114,1)","rgba(41,49,123,1)","rgba(41,109,54,1)","rgba(0,142,84,1)","rgba(40,93,29,1)","rgba(86,95,52,1)","rgba(166,24,53,1)","rgba(103,118,6,1)","rgba(218,68,40,1)","rgba(83,130,75,1)","rgba(94,142,11,1)","rgba(0,137,74,1)","rgba(0,107,67,1)","rgba(0,130,118,1)","rgba(0,124,120,1)","rgba(181,104,30,1)","rgba(0,136,94,1)","rgba(122,129,0,1)","rgba(224,81,47,1)","rgba(56,39,55,1)","rgba(0,148,73,1)","rgba(113,146,0,1)","rgba(64,140,45,1)","rgba(0,91,112,1)","rgba(0,163,132,1)","rgba(97,66,68,1)","rgba(43,151,55,1)","rgba(7,152,47,1)","rgba(87,142,0,1)","rgba(97,191,55,1)"]},"frame":null}],"highlight":{"on":"plotly_click","persistent":false,"dynamic":false,"selectize":false,"opacityDim":0.2,"selected":{"opacity":1},"debounce":0},"shinyEvents":["plotly_hover","plotly_click","plotly_selected","plotly_relayout","plotly_brushed","plotly_brushing","plotly_clickannotation","plotly_doubleclick","plotly_deselect","plotly_afterplot","plotly_sunburstclick"],"base_url":"https://plot.ly"},"evals":[],"jsHooks":[]}</script>

If you play around with this enough, you’ll realize that plotting all of your 3D data on a plot as individual points is kind of cumbersome when you have thousands of points; you can’t really tell which regions of your color space are more or less dense. It may better suit your purposes to cluster the data a bit first, and then plot the clusters:

``` r
clusters <- extractClusters(getKMeanColors(img, color.space = "Lab",
                                           ref.white = "D65",
                                           n = 50, plotting = F))
colnames(clusters) <- c("L", "a", "b", "Pct")

# We can do this with a colordistance function...
scatter3dclusters(clusters, color.space = "lab", scaling = 100)
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-6-1.png" width="672" />

``` r
# we can also use scatter3D, with a bit of a hack to get different point sizes
col_vector <- rgb(convertColor(clusters[ , 1:3], from = "Lab", to = "sRGB"))

# make blank plot
scatter3D(clusters$L, clusters$a, clusters$b, 
          cex = 0, colkey = F, phi = 35, theta = 60,
          xlab = "L", ylab = "a", zlab = "b")

# set scale multiplier for point sizes
scale <- 80

# add one point at a time, setting size with the cex argument
for (i in 1:nrow(clusters)) {
  scatter3D(x = clusters$L[i],
           y = clusters$a[i],
           z = clusters$b[i],
           cex = clusters$Pct[i] * scale, 
           pch = 19, alpha = 0.5,
           col = col_vector[i], add = TRUE)
}
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-6-2.png" width="672" />

``` r
# or, we can just use plotly again
plot_ly(data = clusters,
        x = ~L, y = ~a, z = ~b, 
        type = "scatter3d", mode = "markers", 
        color = I(col_vector), # this is a bit of a hack and you'll get a warning...
        colors = col_vector, size = ~Pct)
```

<div id="htmlwidget-2" style="width:672px;height:480px;" class="plotly html-widget"></div>
<script type="application/json" data-for="htmlwidget-2">{"x":{"visdat":{"67c12c3395a6":["function () ","plotlyVisDat"]},"cur_data":"67c12c3395a6","attrs":{"67c12c3395a6":{"x":{},"y":{},"z":{},"mode":"markers","color":["#2C9230","#4F9034","#65232B","#64512B","#BA7608","#047579","#118236","#7DAD0C","#9B222B","#C14323","#773A33","#572531","#18AEAC","#759006","#0C9849","#B92C2D","#7F1C32","#3D9810","#878408","#069157","#077C6C","#563F3B","#35C87C","#108DB0","#384D39","#5B790C","#22587E","#5B6B26","#99491C","#462F37","#788C47","#508B05","#B0A006","#10753C","#06667C","#722229","#232023","#1860A9","#796C29","#832128","#C66014","#0C8B68","#1BA53D","#545B2D","#8A8684","#3D7323","#9E7D0B","#0FCEA9","#54BD26","#266038"],"size":{},"colors":["#2C9230","#4F9034","#65232B","#64512B","#BA7608","#047579","#118236","#7DAD0C","#9B222B","#C14323","#773A33","#572531","#18AEAC","#759006","#0C9849","#B92C2D","#7F1C32","#3D9810","#878408","#069157","#077C6C","#563F3B","#35C87C","#108DB0","#384D39","#5B790C","#22587E","#5B6B26","#99491C","#462F37","#788C47","#508B05","#B0A006","#10753C","#06667C","#722229","#232023","#1860A9","#796C29","#832128","#C66014","#0C8B68","#1BA53D","#545B2D","#8A8684","#3D7323","#9E7D0B","#0FCEA9","#54BD26","#266038"],"alpha_stroke":1,"sizes":[10,100],"spans":[1,20],"type":"scatter3d"}},"layout":{"margin":{"b":40,"l":60,"t":25,"r":10},"scene":{"xaxis":{"title":"L"},"yaxis":{"title":"a"},"zaxis":{"title":"b"}},"hovermode":"closest","showlegend":false},"source":"A","config":{"showSendToCloud":false},"data":[{"x":[53.1946588835979,53.9596351095243,24.0976564584289,35.6036012488722,55.5373862751527,44.21651687503,47.4239445060195,65.0617731289884,34.7366402990177,46.3246684583419,32.2688263994665,22.3072555232524,64.5954107904765,55.6587946948685,54.9557114800121,41.8793760448375,28.4372845071126,55.6027567096673,53.7361972267739,52.7022094636126,46.3396917808826,28.9570681728065,71.8552991973423,54.3461739170174,30.5404554391342,46.8693842477265,35.5332956130365,42.624929276063,40.7316024142178,22.4432746780307,55.2473571923107,51.9350091118374,65.3565042379547,42.8796136944513,39.4002815788575,26.5751766715309,12.6447841819736,40.1338361320406,45.4889133027626,29.6856257044905,52.473964649942,51.2079509024561,59.4999358475635,37.127931684563,56.1685646806015,43.2088930516049,54.040194454177,74.1967967705915,68.1773193129727,35.8726842325365],"y":[-48.9319862304724,-38.4536482875821,30.4167385062592,2.3539057625659,19.9103869337368,-25.3929355968667,-46.8051639349089,-36.1673476399427,49.4346316271018,49.0187579887318,26.4242721745034,24.1441341490294,-36.0422089936324,-25.9423324479601,-51.7743159618286,55.5545907805274,42.7470331463492,-49.35652253922,-11.8548377214015,-46.8254248527074,-32.8473500621974,9.55580938915399,-55.4621746976972,-18.7144718129058,-12.9982029817216,-25.7483339052722,-4.37777177199426,-17.044849697663,30.6051979420308,11.6178774961548,-18.4576791086705,-38.5919380690018,-7.74897265314973,-41.0652676233888,-16.2512192026847,35.4153216994582,2.1525074433549,6.26556465863796,-3.10589658426307,41.7167895034872,36.8445519801008,-40.3948331265627,-56.7045114720343,-10.9180035725077,1.16887389353378,-33.40460128885,3.39884180009848,-51.0882885468845,-55.3979626888029,-28.9688473920853],"z":[42.0991071907099,41.5928558003257,10.3243909240605,25.2442662286597,60.7198716424919,-10.4896224844523,32.4818728664549,64.8746493593969,26.4265397751342,45.0519679923596,16.3889840689659,3.62900950231146,-9.46559838500863,58.0226451293352,31.9928079846149,34.6176261547054,12.7065401544778,55.0937166029178,56.8206774061723,21.3945487770611,0.258456447095425,6.31991316427493,27.0353200137217,-27.1244338341843,9.23917556208758,49.2046335367832,-26.5501909597621,35.88055124857,40.976202061052,-0.421345367822215,33.9912415917403,54.2427542170472,67.6467678886672,23.842303939043,-19.564988924393,15.6534462172701,-1.79597904549463,-45.4109435932668,38.0348617620099,21.0733611028549,56.4838942617785,10.0271807057811,43.2959492868798,25.7885320667752,1.55291947983258,37.2723855229067,57.4896097876359,6.41094893114716,61.1202159652607,17.2139594147985],"mode":"markers","type":"scatter3d","marker":{"color":["rgba(44,146,48,1)","rgba(79,144,52,1)","rgba(101,35,43,1)","rgba(100,81,43,1)","rgba(186,118,8,1)","rgba(4,117,121,1)","rgba(17,130,54,1)","rgba(125,173,12,1)","rgba(155,34,43,1)","rgba(193,67,35,1)","rgba(119,58,51,1)","rgba(87,37,49,1)","rgba(24,174,172,1)","rgba(117,144,6,1)","rgba(12,152,73,1)","rgba(185,44,45,1)","rgba(127,28,50,1)","rgba(61,152,16,1)","rgba(135,132,8,1)","rgba(6,145,87,1)","rgba(7,124,108,1)","rgba(86,63,59,1)","rgba(53,200,124,1)","rgba(16,141,176,1)","rgba(56,77,57,1)","rgba(91,121,12,1)","rgba(34,88,126,1)","rgba(91,107,38,1)","rgba(153,73,28,1)","rgba(70,47,55,1)","rgba(120,140,71,1)","rgba(80,139,5,1)","rgba(176,160,6,1)","rgba(16,117,60,1)","rgba(6,102,124,1)","rgba(114,34,41,1)","rgba(35,32,35,1)","rgba(24,96,169,1)","rgba(121,108,41,1)","rgba(131,33,40,1)","rgba(198,96,20,1)","rgba(12,139,104,1)","rgba(27,165,61,1)","rgba(84,91,45,1)","rgba(138,134,132,1)","rgba(61,115,35,1)","rgba(158,125,11,1)","rgba(15,206,169,1)","rgba(84,189,38,1)","rgba(38,96,56,1)"],"size":[48.2014388489209,40.2158273381295,80.7913669064748,30.2877697841727,34.1726618705036,59.4244604316547,56.6187050359712,32.6618705035971,62.0143884892086,45.6115107913669,13.6690647482014,43.4532374100719,17.7697841726619,62.2302158273381,70.431654676259,42.589928057554,31.5827338129496,73.0215827338129,55.5395683453237,61.5827338129496,54.8920863309352,50.3597122302158,12.3741007194245,10.6474820143885,31.5827338129496,58.5611510791367,28.1294964028777,45.8273381294964,25.3237410071942,64.8201438848921,24.2446043165468,100,19.9280575539568,40.863309352518,60.5035971223022,95.0359712230216,45.3956834532374,10,44.1007194244604,68.705035971223,31.7985611510791,60.0719424460432,52.9496402877698,49.0647482014388,16.9064748201439,44.1007194244604,44.1007194244604,18.8489208633094,23.8129496402878,28.3453237410072],"sizemode":"area","line":{"color":["rgba(44,146,48,1)","rgba(79,144,52,1)","rgba(101,35,43,1)","rgba(100,81,43,1)","rgba(186,118,8,1)","rgba(4,117,121,1)","rgba(17,130,54,1)","rgba(125,173,12,1)","rgba(155,34,43,1)","rgba(193,67,35,1)","rgba(119,58,51,1)","rgba(87,37,49,1)","rgba(24,174,172,1)","rgba(117,144,6,1)","rgba(12,152,73,1)","rgba(185,44,45,1)","rgba(127,28,50,1)","rgba(61,152,16,1)","rgba(135,132,8,1)","rgba(6,145,87,1)","rgba(7,124,108,1)","rgba(86,63,59,1)","rgba(53,200,124,1)","rgba(16,141,176,1)","rgba(56,77,57,1)","rgba(91,121,12,1)","rgba(34,88,126,1)","rgba(91,107,38,1)","rgba(153,73,28,1)","rgba(70,47,55,1)","rgba(120,140,71,1)","rgba(80,139,5,1)","rgba(176,160,6,1)","rgba(16,117,60,1)","rgba(6,102,124,1)","rgba(114,34,41,1)","rgba(35,32,35,1)","rgba(24,96,169,1)","rgba(121,108,41,1)","rgba(131,33,40,1)","rgba(198,96,20,1)","rgba(12,139,104,1)","rgba(27,165,61,1)","rgba(84,91,45,1)","rgba(138,134,132,1)","rgba(61,115,35,1)","rgba(158,125,11,1)","rgba(15,206,169,1)","rgba(84,189,38,1)","rgba(38,96,56,1)"]}},"textfont":{"color":["rgba(44,146,48,1)","rgba(79,144,52,1)","rgba(101,35,43,1)","rgba(100,81,43,1)","rgba(186,118,8,1)","rgba(4,117,121,1)","rgba(17,130,54,1)","rgba(125,173,12,1)","rgba(155,34,43,1)","rgba(193,67,35,1)","rgba(119,58,51,1)","rgba(87,37,49,1)","rgba(24,174,172,1)","rgba(117,144,6,1)","rgba(12,152,73,1)","rgba(185,44,45,1)","rgba(127,28,50,1)","rgba(61,152,16,1)","rgba(135,132,8,1)","rgba(6,145,87,1)","rgba(7,124,108,1)","rgba(86,63,59,1)","rgba(53,200,124,1)","rgba(16,141,176,1)","rgba(56,77,57,1)","rgba(91,121,12,1)","rgba(34,88,126,1)","rgba(91,107,38,1)","rgba(153,73,28,1)","rgba(70,47,55,1)","rgba(120,140,71,1)","rgba(80,139,5,1)","rgba(176,160,6,1)","rgba(16,117,60,1)","rgba(6,102,124,1)","rgba(114,34,41,1)","rgba(35,32,35,1)","rgba(24,96,169,1)","rgba(121,108,41,1)","rgba(131,33,40,1)","rgba(198,96,20,1)","rgba(12,139,104,1)","rgba(27,165,61,1)","rgba(84,91,45,1)","rgba(138,134,132,1)","rgba(61,115,35,1)","rgba(158,125,11,1)","rgba(15,206,169,1)","rgba(84,189,38,1)","rgba(38,96,56,1)"],"size":[48.2014388489209,40.2158273381295,80.7913669064748,30.2877697841727,34.1726618705036,59.4244604316547,56.6187050359712,32.6618705035971,62.0143884892086,45.6115107913669,13.6690647482014,43.4532374100719,17.7697841726619,62.2302158273381,70.431654676259,42.589928057554,31.5827338129496,73.0215827338129,55.5395683453237,61.5827338129496,54.8920863309352,50.3597122302158,12.3741007194245,10.6474820143885,31.5827338129496,58.5611510791367,28.1294964028777,45.8273381294964,25.3237410071942,64.8201438848921,24.2446043165468,100,19.9280575539568,40.863309352518,60.5035971223022,95.0359712230216,45.3956834532374,10,44.1007194244604,68.705035971223,31.7985611510791,60.0719424460432,52.9496402877698,49.0647482014388,16.9064748201439,44.1007194244604,44.1007194244604,18.8489208633094,23.8129496402878,28.3453237410072]},"error_y":{"width":[]},"error_x":{"width":[]},"line":{"color":["rgba(44,146,48,1)","rgba(79,144,52,1)","rgba(101,35,43,1)","rgba(100,81,43,1)","rgba(186,118,8,1)","rgba(4,117,121,1)","rgba(17,130,54,1)","rgba(125,173,12,1)","rgba(155,34,43,1)","rgba(193,67,35,1)","rgba(119,58,51,1)","rgba(87,37,49,1)","rgba(24,174,172,1)","rgba(117,144,6,1)","rgba(12,152,73,1)","rgba(185,44,45,1)","rgba(127,28,50,1)","rgba(61,152,16,1)","rgba(135,132,8,1)","rgba(6,145,87,1)","rgba(7,124,108,1)","rgba(86,63,59,1)","rgba(53,200,124,1)","rgba(16,141,176,1)","rgba(56,77,57,1)","rgba(91,121,12,1)","rgba(34,88,126,1)","rgba(91,107,38,1)","rgba(153,73,28,1)","rgba(70,47,55,1)","rgba(120,140,71,1)","rgba(80,139,5,1)","rgba(176,160,6,1)","rgba(16,117,60,1)","rgba(6,102,124,1)","rgba(114,34,41,1)","rgba(35,32,35,1)","rgba(24,96,169,1)","rgba(121,108,41,1)","rgba(131,33,40,1)","rgba(198,96,20,1)","rgba(12,139,104,1)","rgba(27,165,61,1)","rgba(84,91,45,1)","rgba(138,134,132,1)","rgba(61,115,35,1)","rgba(158,125,11,1)","rgba(15,206,169,1)","rgba(84,189,38,1)","rgba(38,96,56,1)"]},"frame":null}],"highlight":{"on":"plotly_click","persistent":false,"dynamic":false,"selectize":false,"opacityDim":0.2,"selected":{"opacity":1},"debounce":0},"shinyEvents":["plotly_hover","plotly_click","plotly_selected","plotly_relayout","plotly_brushed","plotly_brushing","plotly_clickannotation","plotly_doubleclick","plotly_deselect","plotly_afterplot","plotly_sunburstclick"],"base_url":"https://plot.ly"},"evals":[],"jsHooks":[]}</script>
