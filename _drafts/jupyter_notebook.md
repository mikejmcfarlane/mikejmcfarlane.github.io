# Jupyter Notebook

## Useful links

[Jupyter documentation](https://jupyter.org/documentation)
[Selecting an Image](https://jupyter-docker-stacks.readthedocs.io/en/latest/using/selecting.html#core-stacks)
[Jupyter Docker Stacks](https://jupyter-docker-stacks.readthedocs.io/en/latest/index.html)
[Three-Dimensional Plotting in Matplotlib](https://jakevdp.github.io/PythonDataScienceHandbook/04.12-three-dimensional-plotting.html)

## How to use

```
docker run -p 8888:8888 jupyter/scipy-notebook:latest
```

With a local mount

```
docker run -v /Users/mikemcfarlane/Dropbox/jupyter_notebook:/home/jovyan/work -p 8888:8888 jupyter/scipy-notebook:latest
```