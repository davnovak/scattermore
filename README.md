
# scattermore

Scatterplots with more datapoints. If you want to plot bazillions of points without much waiting, use this.

## Example

```r
# create 10 million 2D datapoints
data <- cbind(rnorm(1e7),rnorm(1e7))

# prepare empty plot
par(mar=rep(0,4))

# plot the datapoints and see how long it takes
system.time(plot(scattermore(data, rgba=c(64,128,192,10), xlim=c(-3,3), ylim=c(-3,3))))

   user  system elapsed 
  0.413   0.044   0.461 
```

The result:

![Resulting scatterplot](media/result.png "Scatterplot")

Now, how fast would the standard `plot()` do?

```r
# compare with the usual plot function on x11/cairo
system.time(plot(data, pch='.', xlim=c(-3,3), ylim=c(-3,3), col=rgb(0.25,0.5,0.75,0.04)))

   user  system elapsed 
  9.752   0.023   9.794 
```

That's roughly a nice 20x speedup on my laptop. Moreover, if you are using different plotting setups (basically any non-Cairo, windows- or quartz-based grDevices backend), you will very possibly see much greater speedups. Cairo is itself sometimes more than 10x faster than the other backends. That's 200x faster in total.

## How does it work

1. Points and colors get converted to vectors and passed to C
2. C code rasterizes the whole thing to a prepared bitmap. This is already quite fast, but some low-level optimization can probably speed it up several more times. Volunteers/pull requests welcome. (Is there a way to push a raw `uint8_t` array into C from R?)
3. The resulting array gets converted to R raster using `as.raster`, which can get plotted. (Fun fact: When plotting less than roughly 1 million points, most computational time is spent only by this conversion!)

## How fast is it

Let us measure the same example as above, with points limited to different sizes (i.e. in the first case, scattermore receives `data[1:1e4,]`):

```
points  .  average time (s)
--------+------------------
1e4     .  0.037
3e4     .  0.039
1e5     .  0.042
3e5     .  0.051
1e6     .  0.076     -- ~50% of the time is R raster conversion overhead
3e6     .  0.170     -- caches start to overflow here
1e7     .  0.460
```

(Multicolor plotting is slightly slower (usually 2x), because the reading and transporting of the relatively large color matrix eats quite a lot of cache.)

## Caveats and future work

- Y axis is flipped in R rasters! (but you can easily flip `ylim` parameter to fix that)
- You can plot the raster into standard axes (so that it looks like normal `points` output) but it requires some careful manipulation of the R plot functions (and quite a lot of reading TFM).
- Guessing the best raster size for plotting (ideally for pixel-on-pixel output) in R is tricky at best.
- Doing the above 2 tricks in ggplot would be cool, right?
