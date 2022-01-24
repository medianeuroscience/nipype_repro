# nipype_repro

This is the MNL repository for the study: A re-executable GLM-based fMRI data analysis: replicating FSL through Nipype

## Analitical Requirments

### fMRIPrep

Information about fMRIPrep see [Here](https://fmriprep.org/en/stable/index.html). Using fMRIPrep within Docker see [here](https://www.nipreps.org/apps/docker/)

### FSL 6.0.4
We install FSL 6.0.4. into our workstation environment by exectuing the following commands as root:  
`curl -fsSL --retry 5 https://fsl.fmrib.ox.ac.uk/fsldownloads/fsl-6.0.4-centos6_64.tar.gz | tar -xz -C /usr/local/fsl --strip-components 1` 

Thereafter, make sure each/your user has the following paths defined in their .bashrc:
```
FSLDIR=/usr/local/fsl
. ${FSLDIR}/etc/fslconf/fsl.sh
PATH=${FSLDIR}/bin:${PATH}
export FSLDIR PATH
```

### Docker & 
For our docker container, we rely on neurodocker to generate a dockerfile via the following command:
```
neurodocker generate docker --base ubuntu:20.04 \
--pkg-manager apt \
--install vim datalad tree \
--afni version=latest \
--ants version=2.3.1 \
--convert3d version=1.0.0 \
--dcm2niix version=latest method=source \
--freesurfer version=6.0.1 \
--copy license.txt /opt/freesurfer-6.0.1 \
--fsl version=6.0.4 \
--user=neuro \
--miniconda \
create_env=neuro \
conda_install="python=3.7 graphviz jupyter jupyterlab jupyter_contrib_nbextensions matplotlib nbformat nilearn numpy pandas pytest scipy seaborn sphinx sphinxcontrib-napoleon traits" \
pip_install="nibabel atlasreader nipype=1.6.1 neurora pybids" \
activate=true > Dockerfile
```

In the resulting Dockerfile, we do the following edit before it is shared with the wider community:
```
LD_LIBRARY_PATH="$LD_LIBRARY_PATH:/usr/lib/" \
MATLABCMD="/opt/matlabmcr-2018a/v94/toolbox/matlab"
```

This dockerfile is ready to be shared with the wider community. 

Finally, we add our MNL/lab specific user and group IDs to avoid [file permission problems](https://vsupalov.com/docker-shared-permissions/). 
Insert everything after the initial chmod 777 until mkdir -p. 

```
&& chmod 777 /opt && chmod a+s /opt \
&& addgroup --gid 10000 lab \
&& adduser --disabled-password --gecos '' --gid 10000 --uid 2002 rw \
&& adduser --disabled-password --gecos '' --gid 10000 --uid 10004 fhopp \
&& adduser --disabled-password --gecos '' --gid 10000 --uid 10017 mm \
&& adduser --disabled-password --gecos '' --gid 10000 --uid 10018 yc \
&& adduser --disabled-password --gecos '' --gid 10000 --uid 10020 sy \
&& adduser --disabled-password --gecos '' --gid 10000 --uid 10021 pw \
&& adduser --disabled-password --gecos '' --gid 10000 --uid 10024 kw \
&& mkdir -p /neurodocker \
```

Build the container: 

```
docker build - < Dockerfile
```
