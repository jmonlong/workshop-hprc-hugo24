Instructions to run the workshop locally on a machine. 
For example, if the instances/cluster doesn't work or scale, participants could follow on their machine. 

Ideally, steps 1-3 would be done beforehand to avoid spending time during the workshop installing and downloading.

## Install Docker

Find information at [https://docs.docker.com/engine/install/](https://docs.docker.com/engine/install/)

## Pull the JupyterHub image

```
docker pull jupyterhub quay.io/jmonlong/hprc-hugo2024-jupyterhub
```

## Download data

```
docker run -v `pwd`:/app -w /app -u `id -u $USER` quay.io/jmonlong/hprc-hugo2024-jupyterhub sh download-data.sh
```

Or prepare a zenodo with everything? That way no need to use the image

## Launch the server

```
docker run --privileged -v `pwd`/data:/data:ro -v `pwd`/bigdata:/bigdata:ro -p 80:8000 --name jupyterhub quay.io/jmonlong/hprc-hugo2024-jupyterhub jupyterhub
```

Then, in your browser, navigate to `localhost`. The password to access the workspace is `hugo24pangenome`.
