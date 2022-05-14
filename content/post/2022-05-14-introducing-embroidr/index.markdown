---
title: Introducing {embroidr}
author: Ben Vigreux
date: '2022-05-14'
slug: introducing-embroidr
categories: 
- package-description
- tutorial
- hobbies
tags: 
- embroidr
- brickr
- dmc
featuredImage: "style.svg"
---

## tl;dr 

[{embroidr}](https://github.com/BenjaminVigreux/embroidr) is an experimental R package that supports you in planning embroidery projects. Its main function, `create_pattern_from_image()`, allows you to create a cross-stitch embroidery pattern, complete with DMC floss colours needed, from any image.  

## Help with your embroidery needs

Move out of the way [{knitr}](https://github.com/yihui/knitr)! There's a new kid on the block: [{embroidr}](https://github.com/BenjaminVigreux/embroidr). (Ok... now that's out of the way, other than the similarity in names those two packages have very little in common!)

One of my hobbies irl is embroidery. I find it to be a very mindful task that allows me to produce something tangible and that reminds me of people I care about. But unless you're very good at improvising, embroidery always starts with sourcing a pattern and the floss colours needed to produce said pattern. This is where {embroidr} comes in. 

When embroidering [cross-stitch](https://en.wikipedia.org/wiki/Cross-stitch) (a very common and probably the most simple type of embroidery), I've often thought it would be neat to be able to turn any image I like into an embroidery pattern, complete with the DMC [embroidery floss](https://en.wikipedia.org/wiki/Embroidery_thread) colours needed. Naturally, I tried to work out how to do this in R. This was also a good opportunity for me to build my very first R package! (although note that {embroidr} isn't properly unit-tested, please flag any issues [here](https://github.com/BenjaminVigreux/embroidr/issues))

## The ~~knitting~~ embroidery circle

Before I delved into writing my own R package, I tried to see if others had already produced an R package that does what I need it too. I didn't find said package, but I found a few R packages that I could build upon and that I would recommend checking out! 

In particular:
- I was heavily inspired by Ryan Timpe's package [{brickr}](https://github.com/ryantimpe/brickr) which, among other things, allows users to plan building Lego mosaics. The way {brickr}'s code is laid out inspired {embroidr} and you will find many similarities between the Mosaics aspect of {brickr} and {embroidr}'s main function `create_pattern_from_image()`. 
- I came across Sharla Gelfand's package [{dmc}](https://github.com/sharlagelfand/dmc) which allows users to find similar DMC floss colours. The `floss` dataset used by {embroidr} comes directly from {dmc}. 
- Thomas Lin Pedersen's package [{farver}](https://github.com/thomasp85/farver) helps to easily compare (the mathematical distance between) colours and find nearest colours. {farver} is amazing and runs under the hood of {embroidr} (and also under the hoods of {brickr} and {dmc})  
- (Notable mention) I didn't actually use anything from Florian Privé's package [{pixelart}](https://github.com/privefl/pixelart), but it is definitely neat and well worth a look! 

## Tutorial: Turning an image into a cross-stitch pattern

The function `create_pattern_from_image()` allows you to create a cross-stitch embroidery pattern from an image. Its key input is a raster array from any image; and the function includes a number of arguments which enable different options. I run through these options and how to use them below.

First, let's load in the packages we'll want to use and an image of our choice. In this case, I load in the R logo, which I've downloaded in a .png format. I use the `readPNG()` function from {png} to load the image as a raster array. For .jpeg images, you'll want to use `jpeg::readJPEG` instead.


```r
# Load packages
library(png) # Used to load in our image raster array
library(embroidr) # The star of this show

image_array <- png::readPNG("R_logo.png") 
```

### Default pattern

Let's first use the `create_pattern_from_image()` function, using its default arguments. 


```r
embroidr::create_pattern_from_image(image_array)
```

```
## [1] "Cross-stitch pattern produced and saved as embroidr_pattern.svg"
```

```
## [1] "List of DMC colours produced and saved as embroidr_pattern_colour_list.svg"
```

Printed messages have let us know our files have been produced and saved - that's good news! The pattern file looks like this. (Note: this is a scaled-down version to fit the width of this page, the actual .svg file is larger so that the pattern can be printed out clearly!)

![](embroidr_pattern.svg) 
And the colour_list looks like this:

![](embroidr_pattern_colour_list.svg)

By default, the pattern produced is a square of 48 stitches x 48 stitches, picks out colours from all 454 DMC floss colours, and has no limits on the maximum number of DMC floss colours to use. Let's see how we can change these (and other) default settings below.  

### Resized pattern

Three arguments of `create_pattern_from_image()` allow us to resize the image to a pattern of any size we'd like. 

- __size_unit__ sets the unit that we want to use for resizing. This can be 'stitches' (the default), 'inches' or 'cm' (centimetres). 
- __img_size__ is the size that we set, in the unit determined by __size_unit__. If we want a rectangular pattern, we can pass a vector `c( _width_ , _height_ )` to this argument.
- __cloth_count__: When setting the size of the embroidery by inches or centimetres, it's important to know what fabric or [cloth count](https://stitchedmodern.com/blogs/news/what-does-cross-stitch-fabric-count-mean) we are working with. 

For example, say we'd like to produce the R logo in a size of 5cm x 3.75cm on a 28-count fabric, we could use the specification below. I've also gone ahead and used the __file_name__ argument to give the output file a different name.


```r
embroidr::create_pattern_from_image(image_array, 
                                    size_unit = "cm",
                                    img_size = c(5, 3.75),
                                    cloth_count = 28,
                                    file_name = "resized_pattern")
```

![](resized_pattern.svg)
### Playing around with colours

When it comes to colours, there are a few things we can do.

#### Reducing the pool of colours to choose from

First, we can reduce the number of DMC colours to choose from. By default, `create_pattern_from_image()` uses a pool of all 454 available DMC colours to choose from and match to our original image.  


```r
# The dataset floss contains 454 DMC floss colours
embroidr::floss
```

```
## # A tibble: 454 x 6
##    dmc   name                hex       red green  blue
##    <chr> <chr>               <chr>   <dbl> <dbl> <dbl>
##  1 3713  Salmon - Very Light #FFE2E2   255   226   226
##  2 761   Salmon - Light      #FFC9C9   255   201   201
##  3 760   Salmon              #F5ADAD   245   173   173
##  4 3712  Salmon - Medium     #F18787   241   135   135
##  5 3328  Salmon - Dark       #E36D6D   227   109   109
##  6 347   Salmon - Very Dark  #BF2D2D   191    45    45
##  7 353   Peach               #FED7CC   254   215   204
##  8 352   Coral - Light       #FD9C97   253   156   151
##  9 351   Coral               #E96A67   233   106   103
## 10 350   Coral - Medium      #E04848   224    72    72
## # ... with 444 more rows
```

If we already have many colours left over from previous projects (or would like to only use certain tones like pastels), we may want to specify a reduced pool of colours to choose from. For example, say, we have floss left over from previous projects in 25 random colours


```r
set.seed(123)

# tibble with the 25 random colours we own
my_colours_tibble <- dplyr::sample_n(floss, 25) 

# vector of dmc codes for the 25 random colours we own
my_colours_vector <- my_colours_tibble$dmc
```

If we wanted to restrict the pool of DMC colours to match from to these 25 colours instead of the broader pool of colours, we can do this in two ways. We can either pass our tibble of colours to the argument __colour_table__.


```r
embroidr::create_pattern_from_image(image = image_array, 
                                    size_unit = "cm",
                                    img_size = c(5, 3.75),
                                    cloth_count = 28,
                                    file_name = "lower_colour_pool",
                                    
                                    colour_table = my_colours_tibble
                                    
                                    )
```

or we could pass a vector of our 25 colours DMC codes to the argument __colour_palette__. 


```r
embroidr::create_pattern_from_image(image = image_array, 
                                    size_unit = "cm",
                                    img_size = c(5, 3.75),
                                    cloth_count = 28,
                                    file_name = "lower_colour_pool",
                                    
                                    colour_palette = my_colours_vector
                                    
                                    )
```

The result should be the same, and with our example, would result in a pattern that looks something like this:

![](lower_colour_pool.svg)
#### Reducing the number of colours of output

Say we're now less concerned about the pool of colours to use, but we just want to keep the number of DMC floss colours we'll need down so that we don't bankrupt ourselves the next time we head to our local haberdashery. Using the argument __n_colours__, we can set a maximum number of colours to use. For example, say we want to reduce the number of colours we'll need to buy to a maximum of 10, we can use the following.


```r
embroidr::create_pattern_from_image(image = image_array, 
                                    size_unit = "cm",
                                    img_size = c(5, 3.75),
                                    cloth_count = 28,
                                    file_name = "lower_colour_output",
                                    
                                    n_colours = 10
                                    
                                    )
```
The result is a colour list which does not exceed 10 colours. Grand!

![](lower_colour_output_colour_list.svg)

#### Going artsy

The final colour-related argument I'll showcase is something that comes from the {brickr} package and that I could not resist also including in {embroidr}. Using the __warhol__ argument, we can swap the rgb channels (defaulted at `c(1, 2, 3)`) around (e.g. gbr is set using `c(2, 3, 1)`, brg using `c(3, 1, 2)`) to create a bit of colour madness.


```r
embroidr::create_pattern_from_image(image = image_array, 
                                    size_unit = "cm",
                                    img_size = c(5, 3.75),
                                    cloth_count = 28,
                                    file_name = "warhol",
                                    
                                    warhol = c(2, 3, 1)
                                    
                                    )
```

![](warhol.svg)
Groovy.

### Changing how the pattern looks

We may also decide that we want a pattern that looks closer to what the end product will look like, with crosses and all. To do so, we can change the __style__ argument to `"crosses"` (the default is called `"numbers"`). To be honest, I don't really think I'd personally use this because I think the tiled version looks much clearer with the DMC codes written out. 


```r
embroidr::create_pattern_from_image(image = image_array, 
                                    size_unit = "cm",
                                    img_size = c(5, 3.75),
                                    cloth_count = 28,
                                    file_name = "style",
                                    
                                    style = "crosses"
                                    
                                    )
```

![](style.svg)
### Other options

There are other arguments to the `create_pattern_from_image()` function, which I will quickly explain below:

- __brightness__ can be used to increase/decrease the brightness of the image (and therefore of the pattern output). 
- __method__ specifies the specific {farver} algorithm to use to determine which DMC colour is closest to the original image.
- __trans_bg__: If your original image contains transparent background, you can use this argument to set the colour you want the background to have. 
- __colour_list__ can be set to `FALSE` if you do not want a list of DMC colours to be output. 

## Under the hood

If you'd like to know more about how `create_pattern_from_image()` works under the hood, here is a short summary of the pipeline behind it. In reality, `create_pattern_from_image()` is an umbrella function which calls a few different functions:

- `embroidr:image_to_scaled()` takes the image raster array and resizes it to the user's specification. It is also at this stage that options like __brightness__ and __warhol__ are defined. 
- `embroidr::scaled_to_colours()` takes the resized_output and matches the colours of the resized image to the nearest DMC floss colours using {farver}. It returns a dataframe of each cross-stitch position and its associated DMC colour match. 
- `embroidr::colours_to_pattern()` then uses this dataframe to produce the pattern and colour list outputs 

## To Improve

A few things that could be done to improve {embroidr} are:
- Creating an R Shiny app so that anyone (regardless of R ability) can transform an image to a pattern with an easy-to-use interface. Similar to {pixelart}'s Shiny app. 
- Adding more useful embroidery-related functions, such as {dmc}'s simple functions to find the nearest DMC floss colours given a specific colour. 
- The __n_colours__ argument is currently relatively crude and selects the n most used colours, instead of using clustering methods to project colours into a smaller set of colours, as is done in {pixelart}. It would be interesting to implement both options and see how much they differ. 
