了解了数学才知其强大，目前正在摸索中

环境是conda搭建的。步骤如下：

## docker安装anaconda

Dockerfile:

```shell

FROM continuumio/anaconda3

#  $ docker build . -t continuumio/anaconda3:latest -t continuumio/anaconda3:5.3.0
#  $ docker run --rm -it continuumio/anaconda3:latest /bin/bash
#  $ docker push continuumio/anaconda3:latest
#  $ docker push continuumio/anaconda3:5.3.0
RUN /opt/conda/bin/conda update conda -y && /opt/conda/bin/conda update anaconda -y && /opt/conda/bin/conda update python -y

# install jupyter and jupyterlab
RUN /opt/conda/bin/conda install jupyter -y --quiet && /opt/conda/bin/conda install -c conda-forge jupyterlab
CMD [ "/bin/bash" ]

```

编译为自己的镜像

```shell
docker build -t oldsyang/conda .
```

启动容器jupyterlab

```shell
docker run -it -d -p 8896:8888 --name conda-s -v /Users/oldsyang/vscode/conda:/opt/notebooks oldsyang/conda /opt/conda/bin/jupyter lab --notebook-dir=/opt/notebooks  --ip='*' --port=8888 --allow-root --no-browser
```

访问[localhost:8896](localhost:8896)，即可

## 登录容器服务器DIY

```shell
docker exec -it conda-s bash
```

### 设置更新的源

```shell
# 使用国内清华的源
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/ 
conda config --set show_channel_urls yes
```

### 创建环境

```shell
# 创建环境Python版本为3.7.5，名字为opencv的虚拟环境，顺便为其安装ipykernel
conda create -n opencv python=3.7.5 ipykernel
# 创建环境Python版本为3.7.5，名字为demo的虚拟环境，顺便为其安装numpy
conda create -n demo python=3.7.5 numpy
# 激活环境
conda activate opencv
# 通过conda为环境安装包(比如在上边创建环境的时候没有安装ipykernel)
conda install -n opencv ipykernel
# 通过ipykernel为jupyter添加opencv环境（--display-name指在notebook中的显示名）
python -m ipykernel install --user --name opencv --display-name "opencv"
# 使用pip 安装 所需要的包
```

可能需要安装gcc等编译环境：

```shell
apt install build-essential
```

## conda相关命令

查看当前虚拟环境详情：conda info

```shell
(pyspider)
# root @ e55d5cd992fc in ~ [2:27:32]
$ conda info

     active environment : pyspider
    active env location : /opt/conda/envs/pyspider
            shell level : 1
       user config file : /root/.condarc
 populated config files : /root/.condarc
          conda version : 4.7.12
    conda-build version : 3.18.9
         python version : 3.7.5.final.0
       virtual packages :
       base environment : /opt/conda  (writable)
           channel URLs : https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/linux-64
                          https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/noarch
                          https://repo.anaconda.com/pkgs/main/linux-64
                          https://repo.anaconda.com/pkgs/main/noarch
                          https://repo.anaconda.com/pkgs/r/linux-64
                          https://repo.anaconda.com/pkgs/r/noarch
          package cache : /opt/conda/pkgs
                          /root/.conda/pkgs
       envs directories : /opt/conda/envs
                          /root/.conda/envs
               platform : linux-64
             user-agent : conda/4.7.12 requests/2.22.0 CPython/3.7.5 Linux/4.9.184-linuxkit debian/10 glibc/2.28
                UID:GID : 0:0
             netrc file : None
           offline mode : False
```

显示所有的虚拟环境：conda env list（或者conda info --envs）

```shell
$ conda env list
# conda environments:
#
base                     /opt/conda
opencv                   /opt/conda/envs/opencv
pyspider              *  /opt/conda/envs/pyspider
python2                  /opt/conda/envs/python2
```

更新包：conda update conda

```shell
conda update conda -y 
conda update anaconda -y 
conda update python -y
```

删除虚拟环境：conda remove

```shell
conda remove -n your_env_name --all
```

删除虚拟环境python的包：conda remove --name your_env_name  package_name

```shell
conda remove --name your_env_name  package_name
```

## 设置conda-s服务

重启conda-s服务

```shell
docker stop conda-s
docker start conda-s

#或者
docker restart conda-s
```

查看日志

```shell
$ docker logs -f conda-s

[I 09:30:00.752 LabApp] Writing notebook server cookie secret to /root/.local/share/jupyter/runtime/notebook_cookie_secret
[W 09:30:00.932 LabApp] WARNING: The notebook server is listening on all IP addresses and not using encryption. This is not recommended.
[I 09:30:00.940 LabApp] JupyterLab extension loaded from /opt/conda/lib/python3.7/site-packages/jupyterlab
[I 09:30:00.940 LabApp] JupyterLab application directory is /opt/conda/share/jupyter/lab
[I 09:30:00.944 LabApp] Serving notebooks from local directory: /opt/notebooks
[I 09:30:00.944 LabApp] The Jupyter Notebook is running at:
[I 09:30:00.944 LabApp] http://e55d5cd992fc:8888/?token=d1d9447a2667223233be995a8423512279b3669d98e85199a57ff463
[I 09:30:00.944 LabApp]  or http://127.0.0.1:8888/?token=d1d9447a2667223233be995a8423512279b3669d98e85199a57ff463
[I 09:30:00.944 LabApp] Use Control-C to stop this server and shut down all kernels (twice to skip confirmation).
[C 09:30:00.948 LabApp]

    To access the notebook, open this file in a browser:
        file:///root/.local/share/jupyter/runtime/nbserver-1-open.html
    Or copy and paste one of these URLs:
        http://e55d5cd992fc:8888/?token=d1d9447a2667223233be995a8423512279b3669d98e85199a57ff463
     or http://127.0.0.1:8888/?token=d1d9447a2667223233be995a8423512279b3669d98e85199a57ff463

```

本地访问[http://localhost:8896/?token=d1d9447a2667223233be995a8423512279b3669d98e85199a57ff463](http://localhost:8896/?token=d1d9447a2667223233be995a8423512279b3669d98e85199a57ff463)，即可

设置jupyter密码登录:

在浏览器里打开一个terminal或者一个console，按照提示操作，最后重启conda-s服务即可
```python
from notebook.auth import passwd
passwd()
Enter password: 
```


