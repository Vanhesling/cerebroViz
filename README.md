# cerebroViz
cerebroViz is a data mapping tool for visualizing spatiotemporal
    data in the brain. The user inputs a matrix of data and the tool outputs
    publication quality SVG diagrams with color mapping reflective of the input
    data. cerebroViz supports 30 brain regions used by BrainSpan, GTEx, Roadmap
    Epigenomics, and more.

# cerebroViz Installation
cerebroViz can easily be downloaded directly through the repository on GitHub or within R with the following commands.
```
library(devtools)
install_github("ethanbahl/cerebroViz")
```
While this README provides on overview of package functionality and example usage, the full vignette may be accessed with the following command.
```
vignette(topic="intro_cerebroViz", package="cerebroViz")
```

# cerebroViz Basics
cerebroViz is a tool for visualizing spatiotemporal data in the brain. As input, it requires a matrix with rows corresponding to brain regions and columns corresponding to time points. You can learn more about the 30 regions which cerebroViz recognizes in the <a href="#regions">Brain Regions in cerebroViz</a> section.  

cerebroViz includes an example matrix, `cerebroEx`, which is gene expression data for the POGZ gene taken from the Brainspan developmental transcriptome[^1].
  
```{r, echo=TRUE, fig.show='hold', warning=FALSE, tidy=TRUE}
library('cerebroViz')
data(cerebroEx)
head(cerebroEx)[,c(1:7)]
```

Calling cerebroViz on this matrix produces two files in the current working directory: one with the suffix "\_outer.svg" and another with the suffix "\_slice.svg". The _outer_ file corresponds to a lobe view of the brain, while the _slice_ file corresponds to a sagittal view. 

```{r, echo = FALSE, message=FALSE, warning=FALSE}
cerebroViz(cerebroEx)
```

```{r, echo=TRUE, warning=FALSE}
cerebroViz(cerebroEx)

```
[^1]: BrainSpan: Atlas of the Developing Human Brain [Internet]. Funded by ARRA Awards 1RC2MH089921-01, 1RC2MH090047-01, and 1RC2MH089929-01. © 2011. Available from: http://developing human brain.org.



## Specifying a Timepoint

cerebroViz allows visualization of brain data temporally, as well as spatially. Multiple time points (corresponding to the columns of a properly formatted matrix) can be specified for rendering by passing an integer vector to the `timePoint = ` argument of `cerebroViz()`. This generates an _outer_ and _slice_ image for each time point, where the number of the corresponding time point is appended to the file name after _outer_ or _slice_. For example: `ex1_outer_1.svg` is the lobe view for the first time point and `ex1_slice_2.svg` is the sagittal view for the second time point. 

```{r, echo=TRUE, warning=FALSE}
library('cerebroViz')

cerebroViz(cerebroEx, timePoint = c(1, 5, 9))
```
## Changing the Ouput File {#filenames}

Users may specify a custom prefix for the rendered images with the `filePrefix = ` argument of `cerebroViz()`. The saved filenames with also contain a suffix indicating _outer_ or _slice_ and the requested time point before the file extension. By default, `cerebroViz()` save files to the working directory, but a file path may be passed to `filePrefix = ` so images may be saved elsewhere:

```{r, warning=FALSE}
library('cerebroViz')

dir.create('custom_directory/')

cerebroViz(cerebroEx, filePrefix = 'custom_directory/a_custom_filename', timePoint = c(1, 5, 9))

list.files('custom_directory/')

```

## Divergant Scales

By default, `cerebroViz()` maps input data to a linear color scale. When working with divergent data (e.g. gene expression), it is desirable to have a neutral "middle" color. 

A divergent scale can be specified by passing the `divData = TRUE` argument. In this case, `cerebroViz()` maps the **median** value of all data points in the matrix to a neutral yellow and the extreme values to a pleasant red and blue. 

```{r, echo=TRUE, warning=FALSE}
library('cerebroViz')

summary(as.vector(cerebroEx))

cerebroViz(cerebroEx, divData = TRUE, timePoint = 11, filePrefix = 'divergent')
```

## Clamping Outliers {#clamp}

The removal of outliers from data is crucial to producing interpretable visualizations. For example, if we set the data for the cerebellum in our example data `cerebroEx` to an extremely high value: 

```{r, echo=TRUE, warning=FALSE}
cerebroEx -> ex1_out

ex1_out["CB",11] <- 55
```
The cerebellum has an extremely high signal, and all other values are indistinguishable. 
```{r, echo=TRUE, warning=FALSE}
cerebroViz(ex1_out, timePoint = 11, filePrefix = 'outlier')
```

To compensate for outliers, a value can be passed to the `clamp = ` argument of `cerebroViz()`. The *clamp* value is used as a coefficient with the Median Absolute Deviation (MAD) to calculate a range of 'acceptable' values.   

$$median \pm clamp \times MAD $$

Values greater than this range are reduced (or "clamped") to its maximum value, while values less than the range are increased to its minimum. The ideal *clamp* value depends on the shape of the data, but values near 3 are often used as a rule of thumb.  Applying this process to the `ex1_out` matrix, which has been edited to contain an outlier:

```{r, echo=TRUE, warning=FALSE}
cerebroViz(ex1_out, clamp = 3, timePoint = 11, filePrefix = 'outlier_clamped')
```

# Customizing cerebroViz's Appearance

### Labeling regions

To label the regions of the brain for which you have data, pass the `regLabel = TRUE` argument to `cerebroViz()`. The labels generated by `regLabel` always display the cerebroViz default, but are fully editable in any vector graphics manipulation program, or even a text editor. 

```{r, echo=TRUE, warning=FALSE}
cerebroEx -> ex1

cerebroViz(ex1, regLabel = TRUE, filePrefix = 'regLabel')
```


### Labeling time points

Viewers find it easier to follow a series when every image in the series is labled. Passing the `figLable = TRUE` argument to `cerebroViz()` prints the column name for each figure on the lower left corner of the vizualization. 

```{r, echo=TRUE, warning=FALSE}
cerebroEx -> ex1

cerebroViz(ex1, figLabel = TRUE, timePoint = c(5, 9), filePrefix = 'figLabel')
```

### Crosshatching Missing Regions

Depending on specified color schemes, it may be difficult to distinguish a color (likely a neutral color in divergent data) from the background color of the brain. When some data points are missing (NA), corresponding regions may appear to be near the median, when they are actually missing. The argument `naHatch = TRUE` may be passed to `cerebroViz()` to apply a cross-hatch to these regions to help distinguish them from the background or the neutral color.

```{r, echo=TRUE, warning=FALSE}
cerebroEx -> ex1

cerebroViz(ex1, naHatch = TRUE, filePrefix = 'naHatch')
```

### Removing the legend

The color scale bar can be toggled off with the `legend = FALSE` argument of `cerebroViz()`, as may be desired when generating images to be used in multi-panel figures. 

```{r, echo=TRUE, warning=FALSE}
cerebroEx -> ex1

cerebroViz(ex1, legend = FALSE, filePrefix = 'legend')
```

### Custom color palettes 
The `palette = ` argument of `cerebroViz()` allows a custom color palette to be set for the vizulaization. To define a new palette, we recommend passing the `RColorBrewer::brewer.pal()` function to `palette = `.  In fact, the default palettes for `cerebroViz()` use `brewer.pal()`! 

cerebroViz scale                 | RColorBrewer palette
---------------------------------| --------------------------------
`cerebroViz(x, divData = FALSE)` | `brewr.pal(9, "YlOrRd")`
`cerebroViz(x, divData = TRUE)`  | `rev(brewr.pal(11, "RdYlBu"))`[*](#note)

`RColorBrewer::brewer.pal()` takes 2 arguments: `n`, the number of colors in the palette; and a `name` corresponding to the palette to use.  An odd number of colors are reccomended whever `divData = TRUE`, ensuring the median is assigned to a known color. 

```{r, echo=TRUE, warning=FALSE}
library(cerebroViz)
library(RColorBrewer)

cerebroViz(cerebroEx, palette = rev(brewer.pal(11, "PiYG")), divData = TRUE, timePoint = 11, filePrefix = "palette")
```

To see a list of palettes available from RColorBrewer, use `RColorBrewer::display.brewer.all()`. More information about `brewer.pal()` can be read in the documentation for RColorBrewer.


##### {#note} 
\* Note that for `divData = TRUE`, the call to `brewer.pal()` is wrapped in `rev()` so the "cooler" color corresponds to the low end of the scale and the "warmer" to high values.


### Advanced custom colors

`cerebroViz()` allows users to fine-tune the color schemes used for both the visualization and the brain background.The color scale to which the data are mapped can be specified by passing a vector of color names, RGB values or hex values with a length of two or greater to the `palette = ` argument:

```{r, echo=TRUE, warning=FALSE}
library(cerebroViz)

cerebroViz(cerebroEx, palette = c("cornflowerblue", "antiquewhite", "coral"), timePoint = 11, filePrefix = "custom_palette")
```

Users are advised to specify an odd number of colors whenever using `divData = TRUE`, ensuring the median is assigned to a known color. 

The color of the brain background ("empty" regions of the brain), brain outline, and image background can be specified, in that order, to the `secPalette = ` argument. Like `palette`, this argument accepts a vector of color names, RGB values or hex values. Unlike `palette`, `secPalette = ` only accepts a vector of length three. 

```{r, echo=TRUE, warning=FALSE}
library(cerebroViz)

cerebroViz(cerebroEx, secPalette = c("darkgrey", "white", "lightgrey"), timePoint = 11, filePrefix = "secPalette")
```

# Generating a heatmap with cerebroScale
Because it is impractical to interpret many cerebroViz diagrams simultaneously, it often makes sense to also visualize the data as a heatmap. In order to account for the custom scaling `cerebroViz()` can perform thanks to `clamp =` and `divData = `, the convenience function `cerebroScale()` is included. 

When passed a matrix in proper cerebroViz format, `cerebroScale()` returns a new matrix of the same dimensions where all values have been rescaled with the minimum value set to 0.0, and the maximum set to 1.0. 

```{r, echo=TRUE,warning=FALSE}
head(cerebroEx)[,c(1:7)]

head(cerebroScale(cerebroEx, clamp = NULL, divData = FALSE))[,c(1:7)]
```

If the `divData = TRUE` argument is passed to `cerebroScale()`, the median of the dataset is also scaled to 0.5, so it will map to the neutral color of the palette The `clamp = #` argument clamps outliers, as described <a href="#clamp" >here</a>, before converting to the to the 0 to 1 range .

```{r, echo=TRUE, warning=FALSE}
summary(as.vector(cerebroEx))

summary(as.vector(cerebroScale(cerebroEx, clamp = NULL, divData = FALSE)))

summary(as.vector(cerebroScale(cerebroEx, clamp = 3, divData = TRUE)))
```

Note that both `divData = ` and `clamp = ` are required arguments for `cerebroScale()`. If `divData = FALSE` and `clamp = NULL` (the default values in `cerebroViz()`), there is no need to use `cerebroScale()` to rescale data before visualizing as a heatmap. 

Using the `heatmap()` function from the {stats} package is the quickest way to generate a heatmap corresponding to a dataset which had been visualized with cerebroViz. Color schemes for `heatmap()` are specified with the `col = ` argument, and can accept `RColorBrewer::brewer.pal()` as input. If using a default `cerebroViz()` color palette:

cerebroViz scale   | RColorBrewer palette
------------------ | --------------------
`cerebroViz(x, divData = FALSE)` | `brewr.pal(9, "YlOrRd"")`
`cerebroViz(x, divData = TRUE)`  | `rev(brewr.pal(11, "RdYlBu"))`

Note to pass the `scale = "none"` and `Colv = NA` arguments to `heatmap()` so that individual time-points are neither separately centered and scaled, nor rearranged into a dendrogram.

```{r, echo=TRUE, fig.height=7, fig.show='hold', fig.width=7, warning=FALSE}
cerebroScale(cerebroEx, clamp = NULL, divData = TRUE) -> ex1_scaled

heatmap(ex1_scaled, Colv = NA, scale = "none", col = rev(brewer.pal(11, "RdYlBu")))

cerebroViz(cerebroEx, clamp = NULL, divData = TRUE, filePrefix = "scaled")
```

# Brain Regions in cerebroViz {#regions}
Cerebroviz provides 30 regions to which brain data can be mapped. To view a data frame containing mapping of cerebroViz's regions to those of several popular databases:

```{r, echo = TRUE, results='hide', warning=FALSE}
  data(regionMap)
  regionMap
```
```{r, echo=FALSE, results='asis', warning=FALSE}
knitr::kable(regionMap)
```


Because many databases utilize different abbreviation for analogous regions, users are encouraged to name the rows of their data using the cerebroViz conventions described in the above table. If this is not possible,  a matrix with 2 columns, where [,1] contains cerebroViz convention names and [,2] contains the corresponding custom name can be passed to the `customNames = ` argument of cerebroViz. The names cerebroViz uses by convention are reserved, and my not be assigned to custom regions in this way. 

## Custom Regions Example
For example, Brainspan (from which `cerebroEx` was taken), uses the abbreviation CBC instead of CB, and MD instead of THA. Recreating this original nomenclature throws an error:
```{r, echo=TRUE, warning=TRUE}
rownames(cerebroEx)[c(3, 14)]
rownames(cerebroEx)[c(3, 14)] <- c('CBC', 'MD')
cerebroViz(cerebroEx, filePrefix = "missing_name", timePoint = 9)
```

***
To map these regions to the custom names: create a 2 column matrix where [,1] contains cerebroViz convention names and [,2] contains the corresponding custom name, then pass it to the `customNames = ` argument of `cerebroViz()`.
```{r, echo=TRUE, warning=FALSE}
matrix(c("CB", "THA", "CBC", "MD"), ncol=2 ) -> cnm
cnm
cerebroViz(cerebroEx, customNames = cnm, filePrefix = "custom_name", timePoint = 9)
```

***
The data are now properly mapped to the corresponding regions. Note that both the "missing_name" and "custom_name" images have the same scale. cerebroViz uses all data in the supplied matrix to calculate its color scale, even those which cannot be mapped to regions in the output images. 

```{r, include=FALSE}
data("cerebroEx")
cerebroEx
```


# Make an animated gif of cerebroViz diagrams

cerebroViz diagrams can be quickly and easily converted into animated gifs to highlight temporal changes with the **command-line** tool [Imagemagick](http://www.imagemagick.org/script/index.php). Imagemagick is a free image manipulation suite available for Windows, Mac and Linux distributions. 

When generating SVGs which will be animated with `cerebroViz()`, it is recommended to use the `figLabel = TRUE` argument to print the column name for the provided matrix on the corresponding SVG:

```{r, warning=FALSE}
library(cerebroViz)
dir.create("gif_directory")
cerebroViz(cerebroEx2, timePoint = c(1:50), divData = TRUE,  filePrefix = "gif_directory/gif", figLabel = TRUE)

```

To convert the SVGs generated by `cerebroViz()`, enter the directory containing them in the shell, and run the below command, where `-delay n` is the number of milliseconds between frames and `-loop n` the number of times the animation should loop (0 = infinite).

```{bash}
cd gif_directory
convert -delay 100 -loop 0 `ls -v *_slice_*` animated_slice.gif
convert -delay 100 -loop 0 `ls -v *_outer_*` animated_slice.gif
```
