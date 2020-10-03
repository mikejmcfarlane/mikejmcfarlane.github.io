# Jupyter Notebook

- [Jupyter Notebook](#jupyter-notebook)
  - [Useful links](#useful-links)
    - [Setup docker](#setup-docker)
    - [Setup a notebook server on a Pi](#setup-a-notebook-server-on-a-pi)
    - [Examples](#examples)
    - [Details](#details)
  - [How to use docker](#how-to-use-docker)
  - [How to use pi server](#how-to-use-pi-server)

## Useful links

### Setup docker

[Jupyter documentation](https://jupyter.org/documentation)
[Selecting an Image](https://jupyter-docker-stacks.readthedocs.io/en/latest/using/selecting.html#core-stacks)
[Jupyter Docker Stacks](https://jupyter-docker-stacks.readthedocs.io/en/latest/index.html)
[Three-Dimensional Plotting in Matplotlib](https://jakevdp.github.io/PythonDataScienceHandbook/04.12-three-dimensional-plotting.html)

### Setup a notebook server on a Pi

[Running a notebook server](https://jupyter-notebook.readthedocs.io/en/stable/public_server.html)
[Config file and command line options](https://jupyter-notebook.readthedocs.io/en/stable/config.html)
[Jupyter Notebook and Nginx Setup](https://aptro.github.io/server/architecture/2016/06/21/Jupyter-Notebook-Nginx-Setup.html)

### Examples

[Matplotlib examples 2D](https://matplotlib.org/3.1.0/gallery/index.html)
[mplot3d Examples](https://matplotlib.org/2.0.0/examples/mplot3d/index.html)
[The python graph gallery](http://python-graph-gallery.com/barplot/)

### Details

[matplotlib.pyplot.colors](https://matplotlib.org/api/_as_gen/matplotlib.pyplot.colors.html)

## How to use docker

```
docker run -p 8888:8888 jupyter/scipy-notebook:latest
```

With a local mount

```
docker run -v /Users/mikemcfarlane/Dropbox/jupyter_notebook:/home/jovyan/work -p 8888:8888 jupyter/scipy-notebook:latest
```

## How to use pi server

Install stuff:

```bash
sudo apt install python3-pip nginx libatlas-base-dev python3-matplotlib python3-numpy python3-scipy python3-pandas libopenjp2-7
sudo apt autoremove
python3 -m pip install jupyter
```

Add to PATH:
```bash
vi ~/.bash_profile
export PATH="$PATH:/home/pi/.local/bin" 
```

Configure jupyter:

```bash
jupyter notebook --generate-config
vi ~/.jupyter/jupyter_notebook_config.py
c.NotebookApp.allow_remote_access = True
c.NotebookApp.password = '<HASHED PASSWORD>'
```
Plenty ways to generate the above hashed password eg:

```bash
python3
from notebook.auth import passwd; passwd()
```

Configure nginx:

```bash
sudo cp /etc/nginx/sites-enabled/default .
sudo vi /etc/nginx/sites-enabled/default
```

Replace content with:

```bash
upstream notebook {
    server localhost:8888;
}
server{
listen 80;
server_name xyz.abc.com;
location / {
        proxy_pass            http://notebook;
        proxy_set_header      Host $host;
}

location ~ /api/kernels/ {
        proxy_pass            http://notebook;
        proxy_set_header      Host $host;
        # websocket support
        proxy_http_version    1.1;
        proxy_set_header      Upgrade "websocket";
        proxy_set_header      Connection "Upgrade";
        proxy_read_timeout    86400;
    }
location ~ /terminals/ {
        proxy_pass            http://notebook;
        proxy_set_header      Host $host;
        # websocket support
        proxy_http_version    1.1;
        proxy_set_header      Upgrade "websocket";
        proxy_set_header      Connection "Upgrade";
        proxy_read_timeout    86400;
}
}
```

Restart nginx:

```bash
sudo systemctl restart nginx
sudo systemctl status nginx
```

Configure jupyter as a service:

```bash
sudo vi /etc/init/jupyter.conf
sudo vi /etc/systemd/system/jupyter.service
```

```bash
[Unit] 
Description=Jupyter Service 
After=multi-user.target

[Service] 
User=pi
ExecStart=/home/pi/.local/bin/jupyter notebook --no-browser --NotebookApp.allow_origin='*' --notebook-dir='/home/pi/repos' --config=/home/pi/.jupyter/jupyter_notebook_config.py  
Restart=on-failure

[Install] 
WantedBy=multi-user.target
```

Then run the Jupyter service:

```bash
sudo systemctl daemon-reload
sudo systemctl start jupyter
sudo systemctl status jupyter
sudo systemctl enable jupyter
```

