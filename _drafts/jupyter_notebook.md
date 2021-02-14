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
sudo apt install python3-pip nginx libatlas-base-dev python3-matplotlib python3-numpy python3-scipy python3-pandas libopenjp2-7 python3-jinja2
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

Then running on wlan at something like:

```bash
http://192.168.1.153/
```


## Jupyterlab on jetson

### Background

https://hub.docker.com/r/hikariai/jupyterlab
https://github.com/yqlbu/jetson_lab/blob/master/Dockerfile

Build docker image

```bash
sudo docker build -t jupy .
```

Run jupyterlab (then broswe to IP:8888)

```bash
sudo docker run --name jupyterlab -d -e TZ=Europe/London -p 8888:8888 -v /home/pi/nvdli-data:/opt/app/data jupy:latest
```

Kill it when done

```bash
sudo docker ps
sudo docker kill <hash>
sudo docker rm $(sudo docker ps -a -q)
```

Dockerfile

```bash
FROM ubuntu:20.04

RUN apt-get update && apt upgrade -y 

# update timezone
ENV TZ=Europe/London
RUN echo $TZ > /etc/timezone && apt-get install -y tzdata && \
    dpkg-reconfigure tzdata 

# install softwares
RUN apt install -y python3 python3-pip python3-dev python3-setuptools libffi-dev \
    gcc make cmake g++ \
    git nano curl wget
RUN apt install -y python3-matplotlib python3-numpy python3-scipy python3-pandas python3-sklearn python3-sklearn-pandas
RUN ln -s /usr/bin/python3 /usr/bin/python && ln -s /usr/bin/pip3 /usr/bin/pip 
RUN pip install pip -U
RUN pip install jupyterlab
RUN pip install --upgrade --force jupyter-console 
RUN echo 'export PATH=$PATH:~/.local/bin' >> ~/.bashrc

RUN curl -sL https://deb.nodesource.com/setup_12.x | bash - 
RUN apt install -y nodejs 
RUN jupyter lab build --minimize=False

# fix for scipy module load error, maybe should pip3 install modules
RUN apt install -y --reinstall python*-decorator

# set working dir
EXPOSE 8888
RUN mkdir -p /opt/app/data
WORKDIR /opt/app/data

RUN sh -c "$(wget -O- https://github.com/deluan/zsh-in-docker/releases/download/v1.1.1/zsh-in-docker.sh)" -- \
    -t robbyrussell 

CMD [ "/bin/bash", "-c", "SHELL=zsh jupyter lab --ip=* --port=8888 --no-browser \
    --notebook-dir=/opt/app/data \
    --allow-root --NotebookApp.token='' \
    --NotebookApp.password='' " ]

# jupyter lab --ip=* --port=8888 --no-browser --notebook-dir=/opt/app/data --allow-root --NotebookApp.token='' --NotebookApp.password='' --LabApp.terminado_settings='{"shell_command": ["/bin/zsh"]}'
```