---
title: Moesaic Part I
date: 2017-10-09 01:57:09
tags: ["numpy","nearest neighbors","python","scikit-learn"]
---
# Introduction
Almost a year ago, I created a 116sq ft mosaic of the Taj Mahal out of anime girls. It was formed in pieces: 15x12 pages, where each page had 14x12 tiles. In other words, I had to match 30240 tiles to the best anime girl out of 2498 other ones (giving us about 12.11 occurrences per girl).

Sadly, when I built it, I only programmed the part that downloaded the images. I let someone else's program do the mosaic building part for me. While it turned out great, I missed out on the truly fun part of the puzzle.

You see, mosaics are actually a quite complex array of optimization problems. Given an large input matrix (the image), and large database of features (your image database), the following problems must be solved to create an optimal output image:

1. Find the closest image in your database to match a submatrix (tile) of your input matrix. You want the output image to resemble your input image.
2. Minimize the number of repeats for a given classification. If half of your output photo is made of the same photo, it will ruin the aesthetic.
3. If repeats must be made, maximize the distance at which they occur from one another. Multiple repeats are bad, but if they are far enough away from each other, they don't look as bad.

Thus, with these challenges in mind, I have taken it upon myself to find the best way to create a mosaic. One that looks good, isn't too computationally expensive to create, and employs the full extent of its resources (our image database).

I figured I'd do this in 3 parts, and write about the journey of completing each part. Part I will be a simplistic approach to solving the problem–– primarily focused on completing problem 1 without worrying too much about 2 and 3.

Part II would be to solve 2 and 3 using more complex (yet still classical) statistical methods–– I am currently considering genetic algorithms, but I may find better... I might also improve upon 1 by adding texture dimensions through grayscale.

Finally, Part III would be to attempt to optimize all 3 tasks at once with a neural network. I figured II would be a nice stepping point towards this, as I would have to create loss functions for the genetic algorithm anyway.

Before I get into how I did Part I, a little background...

# History
In December of 2016, instead of studying for finals, I started an art project. My roommate and I had surplus printing credits, and we thought it would be neat to print out something big and put it on our ceiling. After brainstorming for a while, we decided we wanted to make a mosaic, printed out on 180 different sheets of printer paper. We weren't sure what we wanted the picture to be, but we thought it would be funny to make it out of thousands of photos of anime girls.  

The first 13 iterations were Whoopi Goldberg peering over a mountain. Here are some of them.  

![Whoopi1](https://imgur.com/ivwBDQY.jpg)  
![Whoopi2](https://imgur.com/3PDIAte.jpg)  
![Whoopi3](https://imgur.com/zE46EpP.jpg)  
This one is different in that it focuses on matching texture, and then re-colors the image to match.

After testing out photos of Whoopi Goldberg, I decided that a better subject would be the Taj Mahal. It had to be tweaked though, as it didn't offer a large enough array of colors.

![Taj1](https://imgur.com/03udMbt.jpg)  
![Taj2](https://imgur.com/9SC4xxL.jpg)  
![Taj3](https://imgur.com/FzHCXQ3.jpg)  
![Taj4](https://imgur.com/vlKGPFt.jpg)  

## Scraping
The scraping part of this isn't all that neat, so I'll just leave you [my code](https://gist.github.com/Manwholikespie/695efa6ab7a156f4b4493dca9557c4d8) if you want to get the images yourself. I recommend using [aria2](https://aria2.github.io) to download the photos in parallel.  

## Early Prototype
![](https://imgur.com/736BBtt.jpg)
One of my first attempts at doing this, was to create a NearestNeighbors classifier on the average RGB value between tiles and database images. In this plot, the blue points are DB images, and orange are tiles. Needless to say (*yet I shall*), this was not affected by the [curse of dimensionality](https://en.wikipedia.org/wiki/Curse_of_dimensionality), but rather the lack of it. I needed more dimensionality to differentiate the gradual transition of color and texture across tiles.

## Assembly
We first want to load all of the images we downloaded. Here, I put them in a folder named p2, where each file is named 1.jpg, 2.jpg, and so on all the way to 1312.jpg. As open each one, I resize it to a 5x5 image, and store its array as a tuple inside my images list. Once we have all of the images, sort their placement in the list by filename, and store them in a pandas DataFrame.

```python
import pandas as pd
import numpy as np
from scipy.misc import toimage, imresize, imsave
from PIL import Image
import os

images = []
for i in os.listdir('p2'):
    name = i.split('.')
    fileName = '%04d.%s' % (int(name[0]),name[1])
    tile_matrix = imresize(np.array(Image.open('p2/'+i)), (5,5))
    
    images.append((fileName,tile_matrix))

images = sorted(images, key=lambda x: x[0])
df = pd.DataFrame(images, columns=['name','vals'])
```
Now, we will take our list of (5,5,3) shaped color matrixes of the form:  

```python  
array([[[217, 204, 197],
        [225, 220, 199],
        [243, 245, 221],
        [244, 246, 225],
        [252, 252, 248]],

       [[172, 166, 155],
        [214, 213, 199],
        [240, 242, 225],
        [247, 247, 237],
        [252, 252, 248]],

       [[176, 172, 158],
        [223, 220, 212],
        [227, 224, 216],
        [234, 228, 226],
        [238, 231, 230]],

       [[234, 231, 218],
        [224, 221, 212],
        [189, 184, 174],
        [209, 194, 189],
        [212, 196, 190]],

       [[243, 240, 232],
        [240, 236, 228],
        [234, 231, 222],
        [219, 206, 200],
        [215, 200, 195]]], dtype=uint8)
```
and flatten each of them down to a (75,) shaped matrix of the form:  

```python
array([217, 204, 197, 225, 220, 199, 243, 245, 221, 244, 246, 225, 252,
       252, 248, 172, 166, 155, 214, 213, 199, 240, 242, 225, 247, 247,
       237, 252, 252, 248, 176, 172, 158, 223, 220, 212, 227, 224, 216,
       234, 228, 226, 238, 231, 230, 234, 231, 218, 224, 221, 212, 189,
       184, 174, 209, 194, 189, 212, 196, 190, 243, 240, 232, 240, 236,
       228, 234, 231, 222, 219, 206, 200, 215, 200, 195], dtype=uint8)
```

To do this:

```python
X = map(lambda x: x.flatten(), df['vals'])
```
and that's that.

## Model - Fitting

![](https://imgur.com/9vp1Vf1.jpg)  
(*Example, credit to [Erik Bernhardsson](https://twitter.com/fulhack)*)

From here, the classification is fairly simple. We have 1312 different data points in 75-dimensional space, so we merely need to fit them to our NearestNeighbors model. We have specified k to be 5 in this, so for each tile in our input photo, we will be getting 5 possibilities as to potential matches.

```python
from sklearn.neighbors import NearestNeighbors
nbrs = NearestNeighbors(n_neighbors=5, algorithm='ball_tree').fit(X)
```

Before I load the input image, I want to be able to split it into 5x5 tiles that I can match to my database images. So, for images that are 500x313 for example, the 500 is divisible by 5 so it can stay, however I will need to resize the 313 to be 315:

```python
def resizeFive(imgArray):
    y = len(imgArray)
    x = len(imgArray[0])
    
    f = lambda z: z if (z%5 == 0) else 5*(z/5 + 1)
    
    return imresize(imgArray, (f(y),f(x)))
```

Now we can load and resize the photo.

```python
base = np.array(Image.open('taj.jpg'))[:,:,:3]
base = resizeFive(base)
```

In order for me to classify each tile, they need to be of the same 75-dimensional form that my DB images are in:

```python
def flattenBase(imgArray):
    tiles = []
    y = len(imgArray)
    x = len(imgArray[0])
    
    for i in range(0,y,5):
        for j in range(0,x,5):
            tiles.append(imgArray[i:i+5,j:j+5].flatten())
    
    return tiles
```
## Model - Classifying
![](https://imgur.com/o6boMC7.jpg)  
(*Example, credit to [Erik Bernhardsson](https://twitter.com/fulhack)*)

Now, I can classify my tile features. As each tile is classified, 5 potential candidates are given, along with how far away each candidate it from matching it exactly. I can take this information, and form a weighted distribution to pick an image from.

Say I have 5 candidates that are each equally far away from my point. Then, they will each have `1.0/5 = 0.2` chance of being picked. As I generate a random float from 0-1, I can say 0-0.2 results in candidate 1, 0.2-0.4 results in candidate 2, and so on.

```python
flat_base = flattenBase(base)
distances, indices = nbrs.kneighbors(flat_base)

def predict(imgNum):
    i = indices[imgNum][1:]
    d = distances[imgNum][1:]

    tot_d = d.sum()
    cur_sum = 0
    choice = np.random.random()
    for x in range(0, len(d)):
        cur_sum += (d[x] / tot_d)
        if choice <= cur_sum:
            return i[x]
    
    return i[x]

# make our predictions
p_tiles = []
for x in range(0,len(indices)):
    p_tiles.append(predict(x))
```

Now that we have a list of what we want to change each tile to, we can re-assemble the photo with our new tiles in place.

```python
def makeImage(origImg,imgDB,tileLabels):
    yDim = len(origImg)/5
    xDim = len(origImg[0])/5
    
    tiles = [imgDB[i] for i in tileLabels]
    columns = []
    for i in range(0,len(tiles),xDim):
        columns.append(np.concatenate(tiles[i:i+xDim], axis=1))
    
    return np.concatenate(columns, axis=0)

finalImg = makeImage(base,df.vals,p_tiles)
imsave('basefinal.jpg', finalImg)
```

Thus, our input photo:

![](https://imgur.com/0jDzXWL.jpg)

becomes this:

![](https://imgur.com/oA5KRon.jpg)

## Conclusion
As you can see, there are a number of repeats, and they are quite close to each other. Out of the 1312 photos I started with, I believe only around 600 of them were used in the final image. So, some work is definitely to be done in Part II for optimizing problems 2 and 3.

With that being said, I'm quite proud of how it turned out. I hadn't done much with matrices before this, and I'm eager to learn more of NumPY's useful features.

Thanks for reading. I'll see you in Part II.