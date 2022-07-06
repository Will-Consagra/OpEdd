# OpEdd
Optimal experimental design and sparse estimation for diffusion MRI.

Code for implementing optimal design and sparse diffusion signal estimation from "Optimized Diffusion Imaging for Brain Structural Connectome Analysis", arXiv:2102.12526 (to appear in IEEE TMI).

## Installing Prerequisites

1. Using a managed system with Slurm + module management, ANTs should be accessible using

```
module load ants/2.3.1
```

2. Install Anaconda3 (https://repo.anaconda.com/archive/) and create a python 3.7.7 instance 'myCondEnv3'. 

3. Install the following python packages + versions: 

```
pip install numpy=='1.19.1'
pip install nibabel=='3.1.1'
pip install dipy=='1.1.1'
pip install scipy=='1.5.2'
pip install sklearn=='0.23.2'
```

## Build Prior

### Initial Data Required 

Building a new template space prior requires a training set of diffusion weighted images. The data should be organized as follows.

1. "${DaTaFolder}/DWI_unregistered" contains a subfolder for each training subject. The name of the subfolders 'subject_id' is linked to the training_ids.txt file. Each training_id folder should house the following files:

	bvals: space separated text file of the b-values 
	bvecs: space separated text file of b-vector coordinates: 3 lines with each line corresponding to a coordinate, (each 'column' indicates a unique b-vector)
	dwi.nii.gz: NIfTI file of diffusion weighted images
	mask.nii.gz: (optional) NIfTI file of binary mask (e.g., white matter mask)
	geom_field.nii.gz: NIfTI file of image to estimate the warp, e.g. FA or b0.  

2. "${OpEddFolder}/training_ids.txt" is a text file with one 'training_id' per row

3. 
	i) Under "${DaTaFolder}/priors", create a subfolder to house the prior objects.
	ii) Under this subfolder, create two directories: 'training' and 'template'. 
	iii) Finally, under the 'template' subfolder, put the 'template.nii.gz' NIfTI file defining the template space in which to build the prior, e.g. FA or b0 template image. 

### Running Code

0. Configure global paths in config.txt

1. To build the template space prior from the training data, run:

```
bash construct_prior.sh
```

This command will create "mean_signal.nii.gz" and "log_cov_signal.nii.gz" NIfTI file tensors in the ${DaTaFolder}/priors/${PRIOR_NAME}/template/ folder. 

2. To map the prior from the template space onto the geometry of an individual subjects brain, run: 

```
bash map_prior.sh
```
Note: Must configure SUBJECT_DIR (in config.txt) to point to a folder with:

	bvals: space separated text file of the b-values (can be only b0 images)
	bvecs: space separated text file of b-vector coordinates: 3 lines with each line corresponding to a coordinate, (each 'column' indicates a unique b-vector)
	dwi.nii.gz: NIfTI file of diffusion weighted images (need at least 2 b0 images)
	mask.nii.gz: NIfTI file of binary mask (e.g., white matter mask)
	geom_field.nii.gz: NIfTI file of FA image (to provide coordinate system for prior injection)

Writes "EB_object.pkl" to ${SUBJECT_DIR}$, which can then be loaded and used for design selection and estimation using the methods in opedd. See example.py for a sample usage.