# Jupyter Notebook

- [Jupyter Notebook](#jupyter-notebook)
  - [Useful links](#useful-links)
    - [Setup](#setup)
    - [Examples](#examples)
    - [Details](#details)
  - [How to use](#how-to-use)

## Useful links

### Setup

[Jupyter documentation](https://jupyter.org/documentation)
[Selecting an Image](https://jupyter-docker-stacks.readthedocs.io/en/latest/using/selecting.html#core-stacks)
[Jupyter Docker Stacks](https://jupyter-docker-stacks.readthedocs.io/en/latest/index.html)
[Three-Dimensional Plotting in Matplotlib](https://jakevdp.github.io/PythonDataScienceHandbook/04.12-three-dimensional-plotting.html)

### Examples

[Matplotlib examples 2D](https://matplotlib.org/3.1.0/gallery/index.html)
[mplot3d Examples](https://matplotlib.org/2.0.0/examples/mplot3d/index.html)
[The python graph gallery](http://python-graph-gallery.com/barplot/)


### Details

[matplotlib.pyplot.colors](https://matplotlib.org/api/_as_gen/matplotlib.pyplot.colors.html)

## How to use

```
docker run -p 8888:8888 jupyter/scipy-notebook:latest
```

With a local mount

```
docker run -v /Users/mikemcfarlane/Dropbox/jupyter_notebook:/home/jovyan/work -p 8888:8888 jupyter/scipy-notebook:latest
```