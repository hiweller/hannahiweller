---
title: Scaling images to a uniform resolution
author: Hannah Weller
date: '2022-03-14'
slug: scaling-images-to-a-uniform-resolution
categories:
  - recolorize
tags:
  - tutorial
  - images
  - image processing
  - resolution
  - resizing
subtitle: ''
summary: ''
authors: []
lastmod: '2022-03-14T14:28:03-04:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---

Kei-Lin Ooi at the University of Melbourne has a very good question about batch resizing (excerpted from an email): 

> I am using images from online databases and thus the images are not standardized. I am using the anisotropic blur to smooth out the image and eliminate image "noise", but the anisotropic blur function is dependent on the image resolution. I was wondering if there was a way of batch processing images to scale and resize them to be, for example, 10 pixels / millimeter, so that the blur function smooths each image proportionally?

This is an important step of image processing, so I wanted to use Kei-Lin's email as motivation for writing out slightly more general instructions for this problem. 

If you're reading this post, you're most likely already familiar with image resolution, usually defined as pixels per some real-world unit, like millimeters or inches (*not* the number of pixels in the image). When we're taking photographs for analysis, we often try to take the highest resolution pictures we can of each subject, which might mean zooming in on smaller specimens and zooming out for large ones. The result is that we're sampling those smaller specimens at a much higher resolution than the larger ones, which if nothing else introduces a confounding variable to our measurements.

Luckily, this is pretty easy to fix a long as you know the resolution of each of your images: you just have to calculate the rescaling factor for the image from its current resolution and its target resolution, and provide that factor as an argument to an image resizing tool. We'll do it in R (for pretty obvious inertia reasons), but I'll list some alternatives at the end of the post.

## Generating example images

First, I'll generate some simple exmple images by resizing an original image. We'll do this with one of the images that comes with `recolorize`, reading it in at 1x, 2x, and 3x the original size:


```r
# get image path
image_path <- system.file("extdata/chongi.png", package = "recolorize")

# load recolorize
library(recolorize)

# read in image at 1x, 2x, and 3x
sample_images <- lapply(1:3, function(i) readImage(image_path, resize = i))
```

We can verify that the dimensions of the 2x and 3x images are indeed 2x and 3x the dimensions of the original:


```r
lapply(sample_images, dim)
```

```
## [[1]]
## [1] 226  90   4
## 
## [[2]]
## [1] 452 180   4
## 
## [[3]]
## [1] 678 270   4
```

I'm using this as sample data because the final solution is obvious: we want to downsample the 2x and 3x images to have the same dimensions as the 1x image again. For now, let's save them as their own set of images:


```r
# save as images
for (i in 1:3) {
  png::writePNG(sample_images[[i]], paste0("beetle_", i, ".png"))
}

# and make a list of image paths
image_paths <- dir(".", "beetle_")
```

Skip this step if you have your own images that you want to resize! 

## Calculating current resolution and rescaling factors

Let's say the beetle in this image is 35mm long, and I measure it in ImageJ on each image to find that the beetle is 200px long in image 1, 400px long in image 2, and 600px long in image 3. That means my image resolutions are 200/35 = 5.71 px/mm for image 1, 400/35 = 11.43 px/mm for image 2, and 600/35 = 17.14 px/mm in image 3. For a real dataset, I would probably make a metadata spreadsheet of image name in one column and resolution in another so that you can keep track of everything.

In our example, we'll try to resize everything so that it's the same as the original resolution of 5.71 px/mm. For an actual dataset, you could pick any target resolution so long as it is equal to or lower than the lowest resolution of any image in your dataset. 

In our case, we know our rescaling factors because we picked them, but let's pretend we need to find them. The simplest calculation is to just divide the target resolution by the current resolution: 

```r
current_resolutions <- c(200/35, 400/35, 600/35)
target_resolution <- 200/35

rescaling_factors <- target_resolution / current_resolutions
print(rescaling_factors)
```

```
## [1] 1.0000000 0.5000000 0.3333333
```

As expected, we end up with rescaling factors of 1 for the original image (no resizing), 0.5 for the 2x image (1/2), and 0.3333 for the 3x image (1/3). 

## Rescaling images

The easiest thing to do is to rescale the images using the `resize` argument in `recolorize`, since you have the option any time you read in an image for the package:


```r
# make an empty list for images
image_list <- vector("list", length = length(image_paths))
image_list <- setNames(image_list, basename(image_paths))

# read in images and rescale appropriately
for (i in 1:length(image_paths)) {
  image_list[[i]] <- recolorize::readImage(image_paths[i],
                                           resize = rescaling_factors[i])
}
```

However, you'll notice something if you look at the actual image dimensions:


```r
lapply(image_list, dim)
```

```
## $beetle_1.png
## [1] 226  90   4
## 
## $beetle_2.png
## [1] 226  90   4
## 
## $beetle_3.png
## [1] 223  89   4
```

The first two images now have identical dimensions, but the last one—which we scaled up 3x—isn't quite right: it's 223x89 pixels, instead of 226x90 pixels, so we have a resolution of about 5.63 px/mm instead of 5.71 px/mm. As far as I can tell, this is an artifact of our particular rescaling factor (1/3) being a repeating decimal (0.333...) that gets rounded by the computer. One way around that would be to explicitly calculate the image dimensions you want, and use the `imager` library directly to specify those dimensions:


```r
library(imager)

# load the image as a cimg object
img <- load.image(image_paths[3])

# this doesn't work (this is the function recolorize calls):
img_imresize <- imresize(img, scale = 1/3)
dim(img_imresize) 
```

```
## [1]  89 223   1   4
```

```r
# cimg objects have x and y flipped, but this is still 223 pixels long x 89
# pixels wide

# we can explicitly calculate image dimensions:
new_dimensions <- dim(img)[1:2] * 1/3

# and specify those dimensions directly:
img_resize <- resize(img, 
                     size_x = new_dimensions[1], 
                     size_y = new_dimensions[2])
dim(img_resize)
```

```
## [1]  90 226   1   4
```

In practice, you'll most likely encounter these fractional rescaling artifacts a lot, and even doing it the more precise way in `imager` doesn't totally eliminate it, since images have to be represented as an integer number of pixels. In this case, our artifact translated to a 0.08 px/mm discrepancy, partly because the images were so small to begin with.

## Saving images

Once you rescale an image, you'll want to save it. I would use `writePNG()` if your images are arrays, but the other option is `imager`'s `save.image()` function:


```r
# we can save it with save.image:
save.image(img_resize, file = "beetle_3_resized.png", quality = 1)

# or we can use a non-exported recolorize function to turn it back into an array:
img_array <- recolorize:::cimg_to_array(img_resize)

# and then export it:
png::writePNG(img_array, "beetle_3_resized.png")
```

That's basically it! Calculate the rescaling factor for each image, apply it to that image, and save the downsampled result. 

## A slightly more realistic example

Here's some code for running through a folder of images (the beetles that come with the package). The easier way:


```r
# get vector of images:
images <- dir(system.file("extdata", package = "recolorize"), 
              "png", full.names = TRUE)

# best practice would be to have these stored in a CSV in one column with image
# name in the other column, but for this example we'll just define image
# resolution as a vector:
image_resolutions <- c(5.71, 6.63, 5.32, 5.91, 6.63) # px/mm

# target resolution is 5 px/mm, since our lowest resolution is 5.3 px/mm
target_resolution <- 5

# for every image...
for (i in 1:length(images)) {
  
  # read in new image, rescaling it appropriately
  new_image <- readImage(images[i], 
                         resize = target_resolution / image_resolutions[i])
  
  # get filename for renaming
  filename <- basename(tools::file_path_sans_ext(images[i]))
  
  # and store output in current working directory
  png::writePNG(new_image, target = paste0(filename, "_resized.png"))
}
```

And the slightly more complicated way:

```r
# or, you could use the more precise option:
for (i in 1:length(images)) {
  # load image
  img <- load.image(images[i])
  
  # define new dimensions
  new_dimensions <- dim(new_image)[1:2] * (target_resolution / image_resolutions[i])
  
  # resize
  new_image <- resize(img,
                      size_x = new_dimensions[1], 
                      size_y = new_dimensions[2])
  
  # and save
  filename <- basename(tools::file_path_sans_ext(images[i]))
  save.image(new_image, file = paste0(filename, "_resized.png"))
}
```

## Alternatives to rescaling in R

Almost any image processing tool will let you resize/rescale an image. The most tedious (and familiar) way is probably to use [Photoshop or Gimp](https://bobbingwidewebdesign.com/photoshop-tutorial/how-do-i-batch-resize-in-gimp.html), manually opening each image and resizing it to desired dimensions. 

Personally, I'm a big fan of [ffmpeg](https://ffmpeg.org/), a popular command line tool for video, audio, and image processing. For example, to resize the 3x beetle, we would just call the following line in the terminal:


```r
> ffmpeg -i beetle_3.png -vf "scale=iw/3:ih/3" beetle_3_resized.png
```

It's pretty easy to automate the renaming and rescaling. The only thing that would be difficult here is figuring out how to supply a different rescaling argument for each image (notice here I just divided the image width and height by 3). I'm sure you could do this by writing a short bash script, but I didn't bother to do that for this post. 

Hopefully this is helpful—I guess the takeaway is that you have to calculate a simple ratio and then use any of half a dozen tools to resize your image. Pretty straightforward!
