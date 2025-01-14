# Reproducing FSL's fMRI Data Analysis via Nipype: Relevance, Challenges, and Solutions

This is the MNL repository for the study _Reproducing FSL's fMRI Data Analysis via Nipype: Relevance, Challenges, and Solutions_
You can also find Google Slides walking through the process here [https://drive.google.com/drive/folders/1isZCz_YIFPb6u75GOcCM814hZi7kI_gB]

## Set-ups

### FSL 6.0.4
We install FSL 6.0.4. on our workstation environment by exectuing the following commands as root:  
`curl -fsSL --retry 5 https://fsl.fmrib.ox.ac.uk/fsldownloads/fsl-6.0.4-centos6_64.tar.gz | tar -xz -C /usr/local/fsl --strip-components 1` 

Thereafter, make sure each/your user has the following paths defined in their .bashrc:
```
FSLDIR=/usr/local/fsl
. ${FSLDIR}/etc/fslconf/fsl.sh
PATH=${FSLDIR}/bin:${PATH}
export FSLDIR PATH
```

### Docker & Nipype

Our docker container ([version: 20.10.12](https://docs.docker.com/engine/release-notes/#201012)) is based on the [Neurodocker](https://github.com/ReproNim/neurodocker). Please install the _Neurodocker_ first, then use it to generate a dockerfile via the following command:
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

Finally, we add our lab specific user and group IDs to avoid [file permission problems](https://vsupalov.com/docker-shared-permissions/). 

Insert everything after the initial chmod 777 until mkdir -p. 

```
&& chmod 777 /opt && chmod a+s /opt \
&& addgroup # add your group ID if needed
&& adduser # add user ID if needed, one line per user
&& mkdir -p /neurodocker \
```

Build the container: 

```
docker build - < Dockerfile
```
## Download Notebooks

You can download or clone our repository via:

```
git clone https://github.com/medianeuroscience/nipype_repro.git
```

## Run!

### GLM

To use our three GLM notebook, please use the following docker command:

```
docker run -it --rm -v {data path}:/home/{user}/data -v {output path}:/home/{user}/out -v .../nipype_repro:/home/{user}/nipype_repro -p 8888:8888 medianeuro/niflow:2.0 jupyter-lab --ip=0.0.0.0 --port=8888
```

### Comparison

To compare outcomes, please run `nipype_fsl_comp.ipynb` and `shell_script.ipynb` outside of the docker above and within a python environment where [nltools](https://nltools.org/) is installed
