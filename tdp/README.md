# Jupyterhub Build

For building Jupyterhub, you need to recover the build container here :
https://github.com/TOSIT-IO/TDP/build-env-python

## Start the container

Next, start the container with specific values regarding your environment but with the official build image above.

For example :

```sh
DCT_GIT="$HOME/git"
DCT_PIP="$HOME/.cache/pip"
DCT_BUILD="$HOME/build"
```

The git directory matchs to the local directory that contains the git repositories.
The build directory matchs to the local directory where the archive is created.

And the command :

```sh
docker run --rm=true -it --env "USER_UID=$(id -u)" --env "USER_GID=$(id -g)" --volume "$DCT_PIP:/home/builder/.cache/pip/" --volume "$DCT_GIT:/home/builder/git/" --volume "$DCT_BUILD:/home/builder/build" --workdir /home/builder/ tdp_builder_python
```

## Sourcing the NodeJS environment

```sh
source /usr/local/nvm/nvm.sh
source /usr/local/nvm/bash_completion
```

## Configuration

If you haven't cloned the repository before, do it, otherwise, it should be mounted with a volume. Verify that you're using the default branch of the repository.

In addition to the git repository, create the requirement file to generate wheel files. The requirement file must contain :

```sh
echo "pip>=21.3.1
jupyterhub-traefik-proxy==0.3.0
jupyterhub_ldapauthenticator==1.3.2
oauthenticator==15.1.0
jupyterhub_yarnspawner==0.4.0
psycopg2-binary==2.8.6" > $HOME/build/requirements.txt
```

## Generate wheels

```sh
python3.6 -m pip wheel --wheel-dir $HOME/build/dependencies/ --requirement $HOME/build/requirements.txt $HOME/git/jupyterhub/
```

## Traeffik

Get the binary for traeffik and change its permissions

```sh
mkdir $HOME/build/bin
curl -L --output $HOME/build/bin/traefik "https://github.com/traefik/traefik/releases/download/v1.7.29/traefik_linux-amd64"
chmod 755 $HOME/build/bin/traefik
```

## Modify requirement files for installation

Add the next line :

```sh
echo "jupyterhub==2.3.1" >> $HOME/build/requirements.txt
```

## Create archive

Create the archive with wheel files, the requirement file and the traeffik binary, then create the checksum

```sh
cd $HOME
tar cf jupyterhub-2.3.1-0.0.tar.gz build/
sha256sum jupyterhub-2.3.1-0.0.tar.gz | sed 's#.*/# #' > jupyterhub-2.3.1-0.0.tar.gz.sha256
```

