Notes to prepare or use the JupyterHub server for the HPRC Pangenome workshop at HUGO 2024.

# Launch an prepared instance

## Launch on EC2

- Start from an image
    - in left panel: *Images* -> *AMIs* 
    - select latest image, e.g. *hprc-workshop-hugo-2024*
    - *Launch instance from AMI*
- Ubuntu Server 22.04, for example:
    - t2.micro for testing
    - c5.9xlarge for testing with a few users
    - u-6tb1.112xlarge for the workshop
- pick your personal keypair
- Allow HTTPS and HTTP traffic from the internet

## Save the IP

Once the instance is running, save the instance's public IP locally as an ENV variable

```
IP=34.215.133.12
```

## Connect to instance 

Using your keypair:

```
ssh -i ~/.ssh/jmonlong-hprc-training.pem ubuntu@$IP
```

## Optional: update data

To be used when starting an instance from scratch, or if starting from an image/snapshot but the data has changed.
See below at [Download/update the data](#Download-update-the-data)

## Launch JupyterHub from a screen

```
screen -S hub

docker run --privileged -v `pwd`/data:/data:ro -v `pwd`/bigdata:/bigdata:ro -v `pwd`/singularity_cache:/singularity_cache:ro -p 80:8000 --name jupyterhub quay.io/jmonlong/hprc-hugo2024-jupyterhub jupyterhub
```

The JupyterHub should be accessible at the public IP through HTTP (https://<IP> won't work!).

# Prepare an instance from scratch

## Minimal installation

```
## install docker, maybe not the best way but works
sudo apt update
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository -y "deb [arch=amd64] https://download.docker.com/linux/ubuntu jammy stable"
sudo apt update
sudo apt install -y docker-ce
sudo usermod -aG docker ${USER}
sudo su - ${USER}

## install screen and AWS CLI
sudo apt install -y awscli screen
```

## Download/update the data

To download the files from S3, I couldn't find a better way than copying my local credentials to the instance...

```
mkdir -p ~/.aws
## locally: scp -i ~/.ssh/jmonlong-hprc-training.pem ~/.aws/config ~/.aws/credentials ubuntu@$IP:.aws/
```

Then

```
aws s3 sync s3://hprc-training/hugo24/ .
```

**Don't forget to remove your AWS credentials after that!**

```
rm ~/.aws/config ~/.aws/credentials
```

## Download/update the notebooks

To update just the notebooks, assuming the big data or cached Singularity images have not changed, we can just pull the latest version of this repo.

```
git clone https://github.com/jmonlong/workshop-hprc-hugo24.git
rm -r data
cp -r workshop-hprc-hugo24/data .
```

# Issues

### Docker problem: `Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?`

Docker sometimes bug. One way to fix, quick reinstall:

```
sudo apt install -y docker-ce
```

# Prepare the Docker image

```
docker build -t jmonlong-jupyterhub .

## test locally
docker run --privileged -v `pwd`/data:/data:ro -v `pwd`/bigdata:/bigdata:ro -v `pwd`/singularity_cache:/singularity_cache:ro -p 80:8000 --name jupyterhub jmonlong-jupyterhub jupyterhub
docker rm jupyterhub

## push to quay.io
docker tag jmonlong-jupyterhub quay.io/jmonlong/hprc-hugo2024-jupyterhub
docker push quay.io/jmonlong/hprc-hugo2024-jupyterhub
```

# Prepare the Singularity cache

*Soon*
