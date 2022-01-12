---
output: html_document
---




# Importing and Visualizing One- and Two-Mode Social Network Data in **statnet**

In this lab we'll explore a variety of methods for importing social network data into R, manipulating one- and two-mode network data, and visualizing social networks. We'll be using a variety of social networks, some of which you'll recognize from other classes. We'll also illustrate a variety of ways to import network data, something that should be easy to do but often turns out to be challenging because a number of resources jump over this important step.

::: {.infobox data-latex=""}
**Note**: This lab has gone through many iterations and reflects the influence from a variety of individuals, including Phil Murphy, and Brendan Knapp.
:::

## Setup

Find and open your RStudio Project associated with this class. Begin by opening a new script. It's generally a good idea to place a header at the top of your scripts that tell you what the script does, its name, etc. 


```r
#######################################################################
# What: Importing and Visualizing One- and Two-Mode Social Network Data
# File: lab1_statnet.R
# Created: 02.28.14
# Revised: 01.05.22
#######################################################################
```

If you have not set up your RStudio Project to clear the workspace on exit, your environment contain the objects and functions from your prior session. To clear these before beginning use the following command.


```r
rm(list = ls())
```

Proceed to place the data required for this lab (`davis.csv`, `davis.net`, `davisedge.csv`, `Koschade Bali (Edge).csv`, `Koschade Bali (Matrix).csv`, and `Koschade Bali.net`) also inside your R Project folder. We have placed it in a sub folder titled `data` for organizational purposes; however, this is not necessary.

## Load Libraries

We need to load the libraries we plan to use. Here we will use **statnet**. Because **igraph** and **statnet** conflict with one another sometimes, we do not want to have them loaded at the same time, so you may want to detach it. Alternatively, you may choose to namespace functions using the `::` operator as needed (e.g., `igraph::betweenness()` vs. `sna::betweenness()`). Of course, this applies only if you had the `igraph` package loaded already. The `intergraph` package allows users to transform network data back and forth between **igraph** and **statnet**.


```r
# If you haven't done so, install the required packages:
# install.packages("statnet")
# install.packages("intergraph")

# Now load them:
library(statnet)
library(intergraph)
```

## One-mode Social Network Data in **statnet**: Koschade Network

Here, we will use data collected by Stuart Koschade of the 17 individuals who participated in the first Bali bombing. Koschade (2006) recorded both the ties between the individuals, as well as the strength of the tie between them.

### Importing One-Mode Social Network Data

#### Option 1: Importing One-Mode Social Network Data in Matrix Format

We can import the network data as a matrix, first using the `as.matrix()` function, and then transforming into a `network` object, which is the object class used by **statnet**, using the `as.network()` function. 



```r
# Here we are nesting functions, the inner functions are evaluated first.
koschade1_net <- as.network(
  as.matrix(
    read.csv("data/Koschade Bali (Matrix).csv", 
             header = TRUE,
             row.names = 1,
             check.names = FALSE)
  ),
  # Arguments for the as.network() function.
  directed = FALSE, 
  ignore.eval = FALSE
)
```

Here's another way to write the same command:


```r
koschade1_net <- as.network(
                     as.matrix(
                         read.csv("data/Koschade Bali (Matrix).csv", 
                                  header = TRUE,
                                  row.names = 1,
                                  check.names = FALSE)),
                  # Arguments for the as.network() function.
                            directed = FALSE, 
                            ignore.eval = FALSE)
```

Now that the data has been imported, let's examine the object. First, take a look at it's class:


```r
class(koschade1_net)
```

```
[1] "network"
```

What is it? The printout should read `network` which is a **statnet** graph object that works with the functions from this library. Many R objects have a class, which describes a type of object, describing the properties it possesses, how it behaves, and how it relates to other objects and functions [@Wickham2019].

By typing the name of the network object into the console, we can get basic information about it.


```r
koschade1_net
```

```
 Network attributes:
  vertices = 17 
  directed = FALSE 
  hyper = FALSE 
  loops = FALSE 
  multiple = FALSE 
  bipartite = FALSE 
  total edges= 63 
    missing edges= 0 
    non-missing edges= 63 

 Vertex attribute names: 
    vertex.names 

 Edge attribute names: 
    1 
```

Like with **igraph** we can retrieve and store attribute data for the graph, vertices (e.g., actor names) or edges (e.g., edge weight) on the graph object. Note that there are multiple ways of retrieving the vertex attributes, such as actor names.


```r
get.vertex.attribute(koschade1_net, "vertex.names")
```

```
 [1] "Muklas"   "Amrozi"   "Imron"    "Samudra"  "Dulmatin" "Idris"   
 [7] "Mubarok"  "Azahari"  "Ghoni"    "Arnasan"  "Rauf"     "Octavia" 
[13] "Hidayat"  "Junaedi"  "Patek"    "Feri"     "Sarijo"  
```

```r
network.vertex.names(koschade1_net)
```

```
 [1] "Muklas"   "Amrozi"   "Imron"    "Samudra"  "Dulmatin" "Idris"   
 [7] "Mubarok"  "Azahari"  "Ghoni"    "Arnasan"  "Rauf"     "Octavia" 
[13] "Hidayat"  "Junaedi"  "Patek"    "Feri"     "Sarijo"  
```

There could be more vertex attributes, which can be called using the attribute name and the `get.vertex.attribute()` function. If you are not certain what the attribute is named, use the `list.vertex.attributes()` function to get a printout of the possible variable names.


```r
list.vertex.attributes(koschade1_net)
```

```
[1] "na"           "vertex.names"
```

Similarly, we can access edge attribute data using **statnet** functions.


```r
# The edge weights are stored in a variable named '1'
get.edge.attribute(koschade1_net, "1")
```

```
 [1] 2 2 1 1 5 1 1 1 1 2 4 5 3 5 3 5 5 5 1 5 2 5 2 2 2 2 2 2 2 2 2 2 2 5 5 5 1 5
[39] 2 2 2 2 2 5 2 1 2 5 1 5 2 2 2 2 2 2 2 2 2 2 1 5 1
```

Once again, you can always get a list of potential edge attribute variable names.


```r
network::list.edge.attributes(koschade1_net)
```

```
[1] "1"  "na"
```

#### Option 2: Importing One-Mode Social Network Data as an Edge List

We can also begin by importing an edge list. Like before, we will begin by reading the data into R using the base function `read.csv()` and examining the data. Then, we will pass the data to the **statnet** function `as.network()`, which constructs a `network` object.


```r
koschade_el <- read.csv("data/Koschade Bali (Edge).csv",
                        header = TRUE)
# Examine top 5 rows
head(koschade_el, 5)
```

```
  Source   Target Weight
1 Muklas   Amrozi      2
2 Muklas    Imron      2
3 Muklas  Samudra      1
4 Muklas Dulmatin      1
5 Muklas    Idris      5
```

Next, we convert it to a `network` object.


```r
koschade2_net <- as.network(koschade_el,
                            matrix.type = "edgelist",
                            directed = FALSE,
                            ignore.eval = FALSE)
```

Type the object name to get a printout with basic information about the network. Note that here the edge weight attribute is called `Weight`.


```r
koschade2_net
```

```
 Network attributes:
  vertices = 17 
  directed = FALSE 
  hyper = FALSE 
  loops = FALSE 
  multiple = FALSE 
  bipartite = FALSE 
  total edges= 63 
    missing edges= 0 
    non-missing edges= 63 

 Vertex attribute names: 
    vertex.names 

 Edge attribute names: 
    Weight 
```

#### Option 3: Importing One-Mode Social Network Data in Pajek Format

We can also read network data in from a Pajek file (*.net extension) and retrieve/check basic information about the network. 


```r
koschade3_net <- read.paj("data/Koschade Bali.net")
koschade3_net
```

```
 Network attributes:
  vertices = 17 
  directed = FALSE 
  hyper = FALSE 
  loops = FALSE 
  multiple = FALSE 
  bipartite = FALSE 
  title = Koschade Bali 
  total edges= 63 
    missing edges= 0 
    non-missing edges= 63 

 Vertex attribute names: 
    vertex.names x y z 

 Edge attribute names: 
    Koschade Bali 
```

Here, the edge weight attribute is imported by default as `Koschade Bali`, which may be misleading. Luckily, the `read.paj()` function has an optional argument to provide the name for the edge variable read from the file.


```r
koschade3_net <- read.paj("data/Koschade Bali.net",
                          edge.name = "Weight")
koschade3_net
```

```
 Network attributes:
  vertices = 17 
  directed = FALSE 
  hyper = FALSE 
  loops = FALSE 
  multiple = FALSE 
  bipartite = FALSE 
  title = Koschade Bali 
  total edges= 63 
    missing edges= 0 
    non-missing edges= 63 

 Vertex attribute names: 
    vertex.names x y z 

 Edge attribute names: 
    Weight 
```


Note that in addition to edge attributes, importing Pajek files includes coordinates of the Pajek layout, stored as `x`, `y`, and `z`. Once again, you can access these attributes using **statnet**'s `get.vertex.attribute()`function. 


```r
get.vertex.attribute(koschade3.net, "x")
get.vertex.attribute(koschade3.net, "y")
get.vertex.attribute(koschade3.net, "z")
```

#### Option 4: Importing One-Mode Social Network Data in **igraph** Format using **intergraph**

You may find yourself working with data in **statnet** and have to convert it to **igraph**. Luckily, the **intergraph** library let's you jump pretty smoothly between the data classes required by each library. 

Here we will take an `network` object and convert it to a `igraph` class object required by the **igraph** library. Then, we will return that object from `igraph` to `network` class.


```r
# Transform an igraph object to network class
koschade1_ig <- asIgraph(koschade1_net)
# Print it
koschade1_ig
```

```
IGRAPH 97d4b60 U--- 17 63 -- 
+ attr: na (v/l), vertex.names (v/c), X1 (e/n), na (e/l)
+ edges from 97d4b60:
 [1]  1-- 2  1-- 3  1-- 4  1-- 5  1-- 6  1-- 8  1-- 9  1--15  1--17  2-- 4
[11]  2-- 6  2-- 7  3-- 4  3-- 5  3-- 6  3-- 8  3-- 9  3--15  3--16  3--17
[21]  4-- 5  4-- 6  4-- 7  4-- 8  4-- 9  4--10  4--11  4--12  4--13  4--14
[31]  4--15  4--17  5-- 6  5-- 8  5-- 9  5--15  5--16  5--17  6-- 7  6-- 8
[41]  6-- 9  6--15  6--17  8-- 9  8--15  8--16  8--17  9--15  9--16  9--17
[51] 10--11 10--12 10--13 10--14 11--12 11--13 11--14 12--13 12--14 13--14
[61] 15--16 15--17 16--17
```

Note the different printout. Also, you can verify the class change using the `class()` function.


```r
class(koschade1_ig)
```

```
[1] "igraph"
```

Now, return the `igraph` object back into `network`, extract the edge list and print the top 5 rows.


```r
koschade_network <- asNetwork(koschade1_ig)

# 1. Extract the edge list with as.data.frame.network()
# 2. Print only top rows with head()
head(as.data.frame.network(koschade_network))
```

```
   .tail    .head X1
1 Muklas   Amrozi  2
2 Muklas    Imron  2
3 Muklas  Samudra  1
4 Muklas Dulmatin  1
5 Muklas    Idris  5
6 Muklas  Azahari  1
```

What changed? Note that some variables and entries may have changed in the transition.

### Plotting (Visualizing) the Koschade Network

Plotting in **statnet** is fairly straight forward. The primary function is `gplot()`, which produces a two dimensional network visualization and allows you to control vertex placements, edge characteristics, colors, etc. We suggest that you take a quick look at the documentation using the command `?gplot`.

To get us started with let's compare a base visualization against a much more refined graph. The first uses **statnet**'s defaults, for the second we will modify many of the arguments. In particular, the second tells R that the network is a one-mode network (`gmode = graph` rather than `digraph`, which is the default), adds labels using the `network.vertex.names()` function , colors the labels black, places them in the center of the nodes (`label.pos = 5`), and changes their size (`label.cex = 1.6`). The next series of arguments set the size and color of the vertices, hides the arrows, and colors the ties (edges) gray. Note that we saved the coordinates so both plots would have the same layout    


```r
# Set graph parameters to 1 row and 2 columns
par(mfrow = c(1, 2))

# Save coordinates in an object
coords <- network.layout.kamadakawai(koschade1_net,
                                     # The function expects a list of parameters
                                     # pass a NULL to use defaults
                                     layout.par = NULL)

# Plot base graph
gplot(koschade1_net,
      coord = coords)

# Plot graph using vertex coordinates and additional arugments
gplot(koschade1_net,
      gmode = "graph",
      coord = coords,
      label = network.vertex.names(koschade1_net),
      label.col = "black",
      label.pos = 5,
      label.cex = 0.5,
      vertex.cex = 1.6,
      vertex.col = "light blue",
      usearrows = FALSE,
      edge.col = "gray")
```

<img src="1-Importing_and_Visualizing_in_Statnet_files/figure-html/unnamed-chunk-21-1.png" width="70%" style="display: block; margin: auto;" />

In the previous visualizations, we used the Kamada and Kawai algorithm to layout the nodes. By default, `gplot()` uses the Fruchterman and Reingold algorithm to determine the positions of nodes. Let's compare the visual output of three layout algorithms: Kamada and Kawai, Fruchterman and Reigold, and circle. Please note that many other layouts exist, for a more indepth list look at the documenation `?gplot.layout`.


```r
# Set graph parameters to 1 row and 3 columns
par(mfrow = c(1, 3))

# Kamada and Kawai
gplot(koschade1_net,
      gmode = "graph",
      mode = "kamadakawai",
      vertex.cex = 1.6,
      vertex.col = "light blue",
      usearrows = FALSE,
      edge.col = "gray")

# Kamada and Kawai
gplot(koschade1_net,
      gmode = "graph",
      mode = "fruchtermanreingold",
      vertex.cex = 1.6,
      vertex.col = "light blue",
      usearrows = FALSE,
      edge.col = "gray")

# Circle
gplot(koschade1_net,
      gmode = "graph",
      mode = "circle",
      vertex.cex = 1.6,
      vertex.col = "light blue",
      usearrows = FALSE,
      edge.col = "gray")
```

<img src="1-Importing_and_Visualizing_in_Statnet_files/figure-html/unnamed-chunk-22-1.png" width="70%" style="display: block; margin: auto;" />

Before we move forward, let's take a look at three more arguments that can grately improve the look of your graphs. First, the `jitter` argument insures that `gplot()` does not draw vertices on top of one another. Second, remember that the edge and vertex attributes can be called and used to aid the visuals. Here we use the `get.edge.attribute()` function to call the edge weight vector (`1`) and rescale the thickness of these. Finally, we can curve edges setting `usecurve` to `TRUE`.


```r
gplot(koschade1_net,
      gmode = "graph",
      coord = coords,
      label = network.vertex.names(koschade1_net),
      label.col = "black",
      label.pos = 5,
      label.cex = 0.5,
      vertex.cex = 1.6,
      vertex.col = "light blue",
      usearrows = FALSE,
      edge.col = "gray",
      # New arguments
      jitter = TRUE,
      edge.lwd = get.edge.attribute(koschade1_net, "1"),
      usecurve = TRUE,
      edge.curve = .1)
```

<img src="1-Importing_and_Visualizing_in_Statnet_files/figure-html/unnamed-chunk-23-1.png" width="70%" style="display: block; margin: auto;" />

### Saving Network Plots (e.g., pdf, jpeg, png, tiff)

Save final plot in various formats. Begin by saving the output in PDF format. To do such, use the `pdf()` function, which starts the graphics driver for producing PDFs. 


```r
# Start the graphic driver, name output file, and set size
pdf(file = "koschade1.pdf",
    width = 4, height = 4)
# Plot the output into the file
gplot(koschade1_net,
      gmode = "graph",
      coord = coords,
      jitter = TRUE,
      label = network.vertex.names(koschade1_net),
      label.col = "black",
      label.pos = 5,
      label.cex = 0.5,
      vertex.cex = 1.6,
      vertex.col = "light blue",
      usearrows = FALSE,
      edge.col = "gray")
# Turn off the graphics driver
dev.off()
```

To store the image as a JPEG, use the `jpeg()` function. The `bg = "transparent"` option saves the graphs with a transparent background (rather than white), which can be helpful when placing in slides or on non-white backgrounds.


```r
jpeg(file = "koschade1.jpg",
     width = 4, height = 4,
     units = 'in',
     res = 600,
     bg = "transparent")

gplot(koschade1_net,
      gmode = "graph",
      coord = coords,
      jitter = TRUE,
      label = network.vertex.names(koschade1_net),
      label.col = "black",
      label.pos = 5,
      label.cex = 0.5,
      vertex.cex = 1.6,
      vertex.col = "light blue",
      usearrows = FALSE,
      edge.col = "gray")

dev.off()
```

To store the image as a PNG, use the `png()` function.


```r
png(file = "koschade1.png",
    width = 4, height = 4,
    units = 'in',
    res = 300,
    bg = "transparent")

gplot(koschade1_net,
      gmode = "graph",
      coord = coords,
      jitter = TRUE,
      label = network.vertex.names(koschade1_net),
      label.col = "black",
      label.pos = 5,
      label.cex = 0.5,
      vertex.cex = 1.6,
      vertex.col = "light blue",
      usearrows = FALSE,
      edge.col = "gray")

dev.off()
```

To store the image as a TIFF, use the `tiff()` function.


```r
tiff(file = "koschade3.tif",
     width = 4, height = 4,
     units = 'in',
     res = 300,
     bg = "transparent")

gplot(koschade1_net,
      gmode = "graph",
      coord = coords,
      jitter = TRUE,
      label = network.vertex.names(koschade1_net),
      label.col = "black",
      label.pos = 5,
      label.cex = 0.5,
      vertex.cex = 1.6,
      vertex.col = "light blue",
      usearrows = FALSE,
      edge.col = "gray")

dev.off()
```

### Saving Network Data

Finally, it doesn't hurt to save the data that you've imported and created. Perhaps not all (e.g., coordinates) but it is helpful to save those that you may want to use in another setting.


```r
save(koschade_el,  
     koschade1_net, 
     koschade2_net, 
     koschade3_net, 
     file = "koschade_statnet.RData")
```

## Two-Mode Social Network Data in **statnet**: Davis Southern Women

We will now switch to another data set to import, manipulate, and visualize two-mode network data in **statnet**. The data that we will use here is what is known as Davis' Southern Club Women. Davis and her colleagues recorded the observed attendance of 18 Southern women at 14 different social events.

Recall that in two-mode network ties only exist between modes. That means that ties are only possible between women and events, not between women and women or between events and events. Any direct ties between nodes within a mode may be derived (projected), as we will do below. But they should not appear within the original network.

### Importing Two-Mode Network Data

#### Option 1: Importing Two-Mode Network Data in Matrix Format

Let's begin my importing two-mode network data that's recorded in matrix format (i.e., an incidence matrix). 


```r
davis_mat <- as.matrix(
  read.csv("data/davis.csv",
           header = TRUE,
           row.names   = 1,
           check.names = FALSE)
  )
```

Convert the matrix into a `network` object with the `as.network()` function, specifying that the network is bipartite and directed through the appropriate arguments. 


```r
davis1_net  <- as.network(davis_mat,
                          # Should the network be interpreted as bipartite?
                          bipartite = TRUE,
                          # Should the edges be interpreted as directed?
                          directed = FALSE,
                          # Ignore edge values?
                          ignore.eval = FALSE,
                          # Optional edgeset constructor argument:
                          matrix.type = "incidence")
davis1_net
```

```
 Network attributes:
  vertices = 32 
  directed = FALSE 
  hyper = FALSE 
  loops = FALSE 
  multiple = FALSE 
  bipartite = 18 
  total edges= 89 
    missing edges= 0 
    non-missing edges= 89 

 Vertex attribute names: 
    vertex.names 

 Edge attribute names: 
    NULL 
```
  
Note that you can use the `is.bipartite()` function to make sure the object is indeed a bipartite (two-mode) network.


```r
is.bipartite(davis1_net)
```

```
[1] TRUE
```


#### Option 2: Importing Two-Mode Network Data as an Edge List

Let's begin by importing an edge list and then check the first few rows with the `head()` command.


```r
davis_el <- read.csv("data/davisedge.csv",
                      header = TRUE)
head(davis_el)
```

```
  Source Target Weight
1 EVELYN     E1      1
2 EVELYN     E2      1
3 EVELYN     E3      1
4 EVELYN     E4      1
5 EVELYN     E5      1
6 EVELYN     E6      1
```

As you can see, the first column is the women and the second is the events they attended. This is how a two-mode edge list should be organized: the first mode will be whatever is represented in the first column and the second mode represented in the second.

To read a bipartite edge list to **statnet** use the `as.network()` function like before. Specify the `matrix.type` as `edgelist`, `directed = TRUE`, and `bipartite = TRUE`.


```r
davis2_net <- as.network(davis_el,
                         matrix.type = "edgelist",
                         directed = FALSE,
                         bipartite = TRUE)
davis2_net
```

```
 Network attributes:
  vertices = 32 
  directed = FALSE 
  hyper = FALSE 
  loops = FALSE 
  multiple = FALSE 
  bipartite = 18 
  total edges= 89 
    missing edges= 0 
    non-missing edges= 89 

 Vertex attribute names: 
    vertex.names 

 Edge attribute names: 
    Weight 
```

Let's check the graph object by plotting it. Once again, you will have to specify the type of graph being evaluated by `gplot()`; to do so, set the `gmode` argument to `twomode`. Note that the women are colored red and events are colored red.


```r
gplot(davis2_net,
      gmode = "twomode", 
      usearrows = FALSE, 
      displaylabels = TRUE,
      label.pos = 5,
      label.cex = .6)
```

<img src="1-Importing_and_Visualizing_in_Statnet_files/figure-html/unnamed-chunk-34-1.png" width="70%" style="display: block; margin: auto;" />


#### Option 3: Importing Two-Mode Social Network Data in Pajek Format

We can also read two-mode network data into R from a Pajek network file. To do so, we will use the `read.paj()` function, then look at the printout to ensure the import worked.


```r
davis3_net <- read.paj("data/davis.net")
davis3_net
```

```
 Network attributes:
  vertices = 32 
  directed = FALSE 
  hyper = FALSE 
  loops = FALSE 
  multiple = FALSE 
  bipartite = 18 
  title = davis 
  total edges= 89 
    missing edges= 0 
    non-missing edges= 89 

 Vertex attribute names: 
    vertex.names x y z 

 Edge attribute names: 
    davis 
```

Is it bipartite?


```r
is.bipartite(davis3_net)
```

```
[1] TRUE
```

Notice again that **statnet** has imported the coordinates from the Pajek layout. Additionally, the file was imported as bipartite, but not as directed. To solve this, use the `set.network.attribute()` function to overwrite the `directed` attribute from `FALSE` to `TRUE`.


```r
davis3_net <- set.network.attribute(davis3_net,
                                   attrname = "directed",
                                   value = TRUE)
davis3_net
```

```
 Network attributes:
  vertices = 32 
  directed = TRUE 
  hyper = FALSE 
  loops = FALSE 
  multiple = FALSE 
  bipartite = 18 
  title = davis 
  total edges= 89 
    missing edges= 0 
    non-missing edges= 89 

 Vertex attribute names: 
    vertex.names x y z 

 Edge attribute names: 
    davis 
```

Finally, remember you can list attributes and actor (vertex) names.


```r
list.vertex.attributes(davis3_net)
```

```
[1] "na"           "vertex.names" "x"            "y"            "z"           
```

```r
network.vertex.names(davis3_net)
```

```
 [1] "EVELYN"    "LAURA"     "THERESA"   "BRENDA"    "CHARLOTTE" "FRANCES"  
 [7] "ELEANOR"   "PEARL"     "RUTH"      "VERNE"     "MYRNA"     "KATHERINE"
[13] "SYLVIA"    "NORA"      "HELEN"     "DOROTHY"   "OLIVIA"    "FLORA"    
[19] "E1"        "E2"        "E3"        "E4"        "E5"        "E6"       
[25] "E7"        "E8"        "E9"        "E10"       "E11"       "E12"      
[31] "E13"       "E14"      
```

### Plotting Two-Mode Networks

At this point, we have already plotted one bipartite network. Here we will compare a few plots using only the `davis1_net` network. Note that we need to tell `gplot()` that it is a two-mode network (with the argument `gmode = "twomode"`). Like before, we will compare layout algorithms side-by-side on the same row.


```r
# Set graph parameters to 1 row and 3 columns
par(mfrow = c(1, 3))
# Store coordinates
coords_fr <- gplot.layout.fruchtermanreingold(davis1_net,
                                              layout.par = NULL)
coords_kk <- gplot.layout.kamadakawai(davis1_net,
                                      layout.par = NULL)
coords_cr <- gplot.layout.circle(davis1_net,
                               layout.par = NULL)
# Plot graphs
gplot(dat = davis1_net,
      gmode = "twomode",
      coord = coords_fr,
      label = network.vertex.names(davis1_net),
      label.col = "black",
      label.cex = 0.6,
      label.pos = 5,
      usearrows = FALSE)

gplot(dat = davis1_net,
      gmode = "twomode",
      coord = coords_kk,
      label = network.vertex.names(davis1_net),
      label.col = "black",
      label.cex = 0.6,
      label.pos = 5,
      usearrows = FALSE)

gplot(dat = davis1_net,
      gmode = "twomode",
      coord = coords_cr,
      label = network.vertex.names(davis1_net),
      label.col = "black",
      label.cex = 0.6,
      label.pos = 5,
      usearrows = FALSE)
```

<img src="1-Importing_and_Visualizing_in_Statnet_files/figure-html/unnamed-chunk-39-1.png" width="70%" style="display: block; margin: auto;" />

The default colors for **statnet** are blue and red, so if we want to assign different colors we can do so by creating a separate `color` vector.


```r
# First, create a vector of length 18 with the value "light blue"
women  <- rep("light blue", times = 18)

# Next, create a vector of length 14 with the value "yellow"
events <- rep("yellow", times = 14)

# Now, combine both into a single vector
color  <- c(women, events)
```

Now, replot the same networks as above. The only difference in the following commands from those above is that they use the stored coordinates and the color vector we just created.


```r
par(mfrow = c(1, 3))

gplot(dat = davis1_net,
      gmode = "twomode",
      coord = coords_fr,
      label = network.vertex.names(davis1_net),
      label.col = "black",
      label.cex = 0.6,
      label.pos = 5,
      vertex.col = color,
      usearrows = FALSE)

gplot(dat = davis1_net,
      gmode = "twomode",
      coord = coords_kk,
      label = network.vertex.names(davis1_net),
      label.col = "black",
      label.cex = 0.6,
      label.pos = 5,
      vertex.col = color,
      usearrows = FALSE)

gplot(dat = davis1_net,
      gmode = "twomode",
      coord = coords_cr,
      label = network.vertex.names(davis1_net),
      label.col = "black",
      label.cex = 0.6,
      label.pos = 5,
      vertex.col = color,
      usearrows = FALSE)
```

<img src="1-Importing_and_Visualizing_in_Statnet_files/figure-html/unnamed-chunk-41-1.png" width="70%" style="display: block; margin: auto;" />

Let's calculate two-mode degree centrality and then assign the scores as actor attributes. First, let's take a look at how to calculate node degree.


```r
degree(davis1_net)
```

```
 [1] 16 14 16 14  8  8  8  6  8  8  8 12 14 16 10  4  4  4  6  6 12  8 16 16 20
[26] 28 24 10  8 12  6  6
```

Note that you can assign that vector of scores as a vertex attributes.


```r
davis1_net <- set.vertex.attribute(davis1_net,
                                   attrname = "degree",
                                   value = degree(davis1_net))
davis1_net
```

```
 Network attributes:
  vertices = 32 
  directed = FALSE 
  hyper = FALSE 
  loops = FALSE 
  multiple = FALSE 
  bipartite = 18 
  total edges= 89 
    missing edges= 0 
    non-missing edges= 89 

 Vertex attribute names: 
    degree vertex.names 

 Edge attribute names: 
    NULL 
```

You can always call this attribute back.


```r
get.vertex.attribute(davis1_net, "degree")
```

```
 [1] 16 14 16 14  8  8  8  6  8  8  8 12 14 16 10  4  4  4  6  6 12  8 16 16 20
[26] 28 24 10  8 12  6  6
```

Plot graph with node size reflecting two-mode degree centrality. The degree scores are rescaled so that the vertices don't overwhelm the graph.


```r
gplot(dat = davis1_net,
      gmode = "twomode",
      coord = coords_fr,
      label = network.vertex.names(davis1_net),
      label.col = "black",
      label.cex = 0.6,
      label.pos = 5,
      vertex.col = color,
      vertex.cex = get.vertex.attribute(davis1_net,
                                        attrname = "degree")/10,
      usearrows = FALSE)
```

<img src="1-Importing_and_Visualizing_in_Statnet_files/figure-html/unnamed-chunk-45-1.png" width="70%" style="display: block; margin: auto;" />

### Projecting (Folding) Two-Mode Networks into One-Mode Networks in **statnet**.

For this section, we will just use the `davis1_net` network ojbect.

#### Multiplying Matrices

We can transform the network into a one-mode network of the women by multiplying the matrix (not the graph) by its transpose in order to get a one-mode of the women-to-women.

First, let's take a look at how to extract an adjacency matrix from the graph.


```r
as.matrix.network.adjacency(davis1_net)
```

```
          E1 E2 E3 E4 E5 E6 E7 E8 E9 E10 E11 E12 E13 E14
EVELYN     1  1  1  1  1  1  0  1  1   0   0   0   0   0
LAURA      1  1  1  0  1  1  1  1  0   0   0   0   0   0
THERESA    0  1  1  1  1  1  1  1  1   0   0   0   0   0
BRENDA     1  0  1  1  1  1  1  1  0   0   0   0   0   0
CHARLOTTE  0  0  1  1  1  0  1  0  0   0   0   0   0   0
FRANCES    0  0  1  0  1  1  0  1  0   0   0   0   0   0
ELEANOR    0  0  0  0  1  1  1  1  0   0   0   0   0   0
PEARL      0  0  0  0  0  1  0  1  1   0   0   0   0   0
RUTH       0  0  0  0  1  0  1  1  1   0   0   0   0   0
VERNE      0  0  0  0  0  0  1  1  1   0   0   1   0   0
MYRNA      0  0  0  0  0  0  0  1  1   1   0   1   0   0
KATHERINE  0  0  0  0  0  0  0  1  1   1   0   1   1   1
SYLVIA     0  0  0  0  0  0  1  1  1   1   0   1   1   1
NORA       0  0  0  0  0  1  1  0  1   1   1   1   1   1
HELEN      0  0  0  0  0  0  1  1  0   1   1   1   0   0
DOROTHY    0  0  0  0  0  0  0  1  1   0   0   0   0   0
OLIVIA     0  0  0  0  0  0  0  0  1   0   1   0   0   0
FLORA      0  0  0  0  0  0  0  0  1   0   1   0   0   0
```

Now, let's generate a one-mode matrix of women-to-women relations based on shared participation in an event. To do so, we will multiply the adjacency matrix using the `%*%` operator times its transpose (`t()`).


```r
davis_women_mat <- as.matrix.network.adjacency(davis1_net) %*%
  t(as.matrix.network.adjacency(davis1_net))
```

View the matrix:


```r
davis_women_mat
```

Repeat the process, this time switch the order of the transposed matrix to generate an events-to-events matrix.


```r
davis_events_mat <- t(as.matrix.network.adjacency(davis1_net)) %*%
  as.matrix.network.adjacency(davis1_net)
```

Finally, convert the new matrices into `network` objects.


```r
davis_women_net <- as.network(davis_women_mat)
davis_events_net <- as.network(davis_events_mat)
```

### Plotting Projected One-Mode Networks

Now that we have extracted the one-mode networks, plot the two new graphs using `gplot()` and the additional arguments used previously.


```r
par(mfrow = c(1, 2))
# Save coordinates
coords_women_kk <- gplot.layout.kamadakawai(davis_women_net,
                                            layout.par = NULL)
coords_events_kk <- gplot.layout.kamadakawai(davis_events_net,
                                            layout.par = NULL)
# Plot graphs
gplot(dat = davis_women_net,
      gmode = "onemode",
      coord = coords_women_kk,
      label = network.vertex.names(davis_women_net),
      label.col = "black",
      label.cex = 0.6,
      label.pos = 5,
      vertex.col = "light blue",
      usearrows = FALSE)
gplot(dat = davis_events_net,
      gmode = "onemode",
      coord = coords_events_kk,
      label = network.vertex.names(davis_events_net),
      label.col = "black",
      label.cex = 0.6,
      label.pos = 5,
      vertex.col = "yellow",
      usearrows  = FALSE)
```

<img src="1-Importing_and_Visualizing_in_Statnet_files/figure-html/unnamed-chunk-51-1.png" width="70%" style="display: block; margin: auto;" />

Resize the nodes by degree centrality. This time, we will not store the value as a vertex attribute.


```r
par(mfrow = c(1, 2))
# Plot graphs
gplot(dat = davis_women_net,
      gmode = "onemode",
      coord = coords_women_kk,
      label = network.vertex.names(davis_women_net),
      label.col = "black",
      label.cex = 0.6,
      label.pos = 5,
      vertex.col = "light blue",
      vertex.cex = degree(davis_women_net, gmode = "graph")/10,
      usearrows = FALSE)
gplot(dat = davis_events_net,
      gmode = "onemode",
      coord = coords_events_kk,
      label = network.vertex.names(davis_events_net),
      label.col = "black",
      label.cex = 0.6,
      label.pos = 5,
      vertex.col = "yellow",
      vertex.cex = degree(davis_events_net, gmode = "graph")/10,
      usearrows  = FALSE)
```

<img src="1-Importing_and_Visualizing_in_Statnet_files/figure-html/unnamed-chunk-52-1.png" width="70%" style="display: block; margin: auto;" />

### Saving Network Plots

Now, save plots of the two-mode network and the two one-mode networks.


```r
png(file = "davis1.png", width = 4, height = 4, units = 'in', res = 300,
    bg = "transparent")
gplot(dat = davis1_net,
      gmode = "twomode",
      coord = coords_fr,
      label = network.vertex.names(davis1_net),
      label.col = "black",
      label.cex = 0.6,
      label.pos = 5,
      vertex.col = color,
      vertex.cex = get.vertex.attribute(davis1_net, attrname = "degree")/10,
      usearrows = FALSE)
dev.off()

png(file = "daviswomen.png", width = 4, height = 4, units = 'in', res = 300,
    bg = "transparent")
gplot(dat = davis_women_net,
      gmode = "onemode",
      coord = coords_women_kk,
      label = network.vertex.names(davis_women_net),
      label.col = "black",
      label.cex = 0.6,
      label.pos = 5,
      vertex.col = "light blue",
      vertex.cex = degree(davis_women_net, gmode = "graph")/10,
      usearrows = FALSE)
dev.off()

png(file = "davisevents.png", width = 4, height = 4, units = 'in', res = 300,
    bg = "transparent")
gplot(dat = davis_events_net,
      gmode = "onemode",
      coord = coords_events_kk,
      label = network.vertex.names(davis_events_net),
      label.col = "black",
      label.cex = 0.6,
      label.pos = 5,
      vertex.col = "yellow",
      vertex.cex = degree(davis_events_net, gmode = "graph")/10,
      usearrows = FALSE)
dev.off()
```

### Saving Network Data

Once again, it doesn't hurt to save the data that you've imported and created.


```r
save(davis_el, 
     davis1_net, 
     davis2_net, 
     davis3_net, 
     davis_events_net,
     davis_women_net,
     file = "data/davis_statnet.RData")
```

That's all for **statnet** for now.
