
# Three Dimensional Optimal Spectral Extraction (TDOSE)

[//]: # "![TDOSE logo](TDOSElogo.png =250x)"

README for the optimal spectral extraction software TDOSE.

[//]: # "presented by Schmidt et al. (some day)"

TDOSE will be presented in a forthcoming publication ([Schmidt et al. in prep.](#references)), until this publication has appeared, please reference the TDOSE GitHub repository (https://github.com/kasperschmidt/TDOSE) if you find TDOSE useful. 

## Table of Content
<a href="TDOSElogo.png"><img src="TDOSElogo.png" align="right" height="300" ></a>

- [Description](#description)
- [Installing TDOSE](#installing-tdose)
- [Dependencies and Requirements](#dependencies-and-requirements)
  - [Standard Packages](#standard-packages)
  - [Special Packages](#special-packages)
- [Script Overview](#script-overview)
- [Running TDOSE](#running-tdose)
  - [Default Run](#default-run-of-tdose)
  - [Individual TDOSE Function Runs](#individual-tdose-function-runs)
- [TDOSE Output Overview](#tdose-output-overview) 
- [References](#references)
- [Schematic Overview of TDOSE](#schematic-overview-of-tdose)

## Description

The software for Three Dimensional Optimal Spectra Extraction (TDOSE) is build in Python to extract spectra of both point sources and extended sources from integral field data cubes, i.e., three dimensional data with spatial (x,y) and wavelength dimensions (λ). TDOSE was build for spectral extraction from MUSE data cubes. However, TDOSE was also build to be as broadly appliable as possible. Therefore, TDOSE should be able to extract 1D spectra from any 3D FITS data cube.

TDOSE broadly follows the point source extraction software described by [Kamann, Wisotzki and Roth (2013)](#references) adding the capability of modeling sources as non-point sources, e.g., via (multiple) multivariate gaussian modeling or GALFIT modeling (in which case the GALFIT FITS model of the reference image is passed directly to TDOSE), and using these non-point source models to guide the three dimensional extractions to approach optimal spectral extraction of extended objects.

As each sources in the field-of-view are modeled and the fluxes are optimized simultaneously in each individual wavelength layer, TDOSE also offers natural de-blending of neighboring sources.

Below is a quick run-through of TDOSE and how to use it. Comments and/or suggestions for improvement, new features etc. are more than welcome. Please send these to Kasper B. Schmidt (kbschmidt at aip.de) or add an 'issue' via GitHub.

## Installing TDOSE

TDOSE does not require any installation. Simply cloning the TDOSE GitHub repository (or downloading one of the releses from https://github.com/kasperschmidt/TDOSE/releases) and importing the scripts should be enough.
Hence, TDOSE is "installed" by doing:
```
git clone https://github.com/kasperschmidt/TDOSE.git
```
After adding the TDOSE directory to the `PYTHONPATH` or changing location to the TDOSE directory, TDOSE can be imported in `python` with:
```python
import tdose
```
If the import does not generate any errors, i.e., if the TDOSE dependencies and requirements listed in the following section are met, TDOSE is ready for use.

## Dependencies and Requirements

TDOSE is written in Python and uses a range of default packages included in standard installations of Python. As TDOSE was developed using STScI's [`ureka`](http://ssb.stsci.edu/ureka/) Python instalation, "standard" here referes to packages available from a `ureka` (or [`AstroConda`](#http://astroconda.readthedocs.io/en/latest/)) STScI Python installation. A list of 'special packages' that needs to be installed on top of that to get TDOSE running are also given below.

### Standard Packages

The following standard Python packages are imported in one or more of the TDOSE scripts and therefore needs to be available to run TDOSE successfully: 
`datetime`,
`collections`, 
`glob`,
`matplotlib`,
`multiprocessing`,
`numpy`,
`pdb`,
`pyfits`,
`scipy`,
`shutil`,
`subprocess`,
`sys`,
`time`, and
`warnings`.

### Special Packages

The follwoing 'special packages' also needs to be accessible to Python to run TDOSE:

`astropy`: http://www.astropy.org. Can be installed with pip. Used extensively throughout TDOSE. 

`reproject`: https://reproject.readthedocs.io. Can be installed with pip. Used to project model of reference image to data cube WCS, when the model is not a set of simple gaussians that can be build from scratch (i.e., when `source_model = modelimg` in the setup file).

## Script Overview

The following gives and overview of the scripts provided with the TDOSE code. For further details please refer to the headers of the scripts and subroutines themselves. 
A schematic overview is provided in the [Schematic Overview of TDOSE](#schematic-overview-of-tdose).

- `tdose.py`
  - The main wrapper for performing the spectral TDOSE extraction. The main command to run is `tdose.perform_extraction()`. Extractions from multiuple data cubes can be run in parallel on multiple cores by using `tdose.perform_extractions_in_parallel()`. See [Running TDOSE](#running-tdose) for details.
- `tdose_model_FoV.py`
  - Routines and functions for modeling the morphology of the sources in the field-of-view of the reference image (which are used for the optimal extraction). 
- `tdose_model_cube.py`
  - Rountines and functions for generating a 3D model cube based on the refernce image model and a model of the instrument PSF. 
- `tdose_extract_spectra.py`
  - Functions to extract and store the 1D spectra of individual objects produced by the TDOSE modeling in binary FITS tables. A single object spectrum can combine the flux models from multiple sources if desired. 
- `tdose_modify_cube.py`
  - Routines and functions for modifying the input 3D data cube, bu subtracting the 3D models generated by TDOSE of individual (or multiple) sources from the original input data cube.
- `tdose_utilities.py`
  - Collection of useful functions used by TDOSE including functions to generate template setup files, build 2D gaussians, perform convolutions, run GALFIT, and extract sub-images and sub-cubes.
- `tdose_setup_template.txt`
  - Example template text file containig the setup for running the TDOSE functions and routines. This template can be generated with `tdose_utilities.generate_setup_template()`.
- `tdose_setup_template_modify.txt`
  - Example template text file containig the setup for modifying the input 3D data cube using the TDOSE source models. This template can be generated with `tdose_utilities.generate_setup_template_modify()`.
- `tdose_build_mock_cube.py`
  - Routines and functions used to build mock data cubes which can be useful for testing and trouble shooting TDOSE.

## Running TDOSE

In the following a few useful examples of how to produce outputs using the TDOSE scripts are given. For examples on how to run individual pieces of code, please refer to the individual code headers.

### Default Run of TDOSE

A full TDOSE run (with minimal output) is performed by simply excecuting 
```python
import tdose
tdose.perform_extraction(setupfile='path/to/setup/tdose_setup.txt', verbose=True, verbosefull=False)
```
in a Python environment. Here `tdose_setup.txt` is a completed TDOSE setup file. A new template TDOSE setup file can be genererated with `tdose_utilities.generate_setup_template()`. Keywords to `tdose.perform_extraction()` can be used to ignore individual steps of the TDOSE run, which can be useful for repeated runs with minor changes to the setup. See the header of `tdose.perform_extraction()` for details on these keywords.

### Parallel run of multiple data cubes

TDOSE can be run in parallel on multiple data cubes. This is done by launching multiple default TDOSE runs in paralell with the `multiprocessing` package. To run in parallel simply provide a list of setup files to `tdose.perform_extractions_in_parallel()` with a calling sequence similar to:
```python
import tdose, glob
setupfiles           = glob.glob('/path/to/TDOSE/setups/tdose_setup_*[0-99].txt')
bundles, paralleldic = tdose.perform_extractions_in_parallel(setupfiles,Nsessions=8,clobber=True,generateOverviewPlots=True,skipextractedobjects=True,logterminaloutput=True)
```
Here `Nsessions` decides how many processes to launch (the setup files will be bundled up in this number of sub-lists), which essentially corresponds to the nunber of cores to occupy.

### Individual TDOSE Function Runs

Below a few examples of how to run individual pieces of TDOSE code to perform individual tasks are given, as these might be useful as stand-alone functions.

Details of the individual steps of a full TDOSE run can be found in `tdose.py`. 

#### Model Reference Image with Multiple Multivariate Gaussians

More details comning soon... in the meantime take a look at `tdose.model_refimage()`.
```python
import tdose_utilities as tu
import tdose_model_FoV as tmf

pinit, fit    = tmf.gen_fullmodel(img_data, sourcecat, modeltype=setupdic['source_model'], verbose=verbosefull, xpos_col=setupdic['sourcecat_xposcol'], ypos_col=setupdic['sourcecat_yposcol'], datanoise=None, sigysigxangle=sigysigxangle, fluxscale=fluxscale, generateimage=modelimg, generateresidualimage=True, clobber=clobber, outputhdr=img_hdr, param_initguess=param_initguess)

tu.model_ds9region(modelparam, regionfile, img_wcs, color='cyan', width=2, Nsigma=2, textlist=names, fontsize=12, clobber=clobber)
```

#### Model Data Cube 

More details comning soon...
```python
import tdose_model_cube as tmc
```

#### Model Reference Image with GALFIT

More details comning soon...
```python
import tdose_utilities as tu
image           = 'path/to/refimage/referenceimage.fits'
sigmaimg        = 'path/to/sigmaimage/sigmaimage.fits'
psfimg          = 'path/to/PSFimage/PSFimage.fits'
galfitinputfile = 'path/to/inputfile/galfit_inputfile.txt'

modelparam      = path+'datacube_modelimage_objparam.fits'
paramHSTinit    = tu.build_paramarray(modelparam,returninit=True,verbose=True)
paramHSTfitted  = tu.build_paramarray(modelparam,returninit=False,verbose=True)

tu.galfit_buildinput_fromparamlist(galfitinputfile, paramHSTfitted, image, objecttype='gaussian', sigmaimg=None, psfimg=psfimg, platescale=[0.03,0.03], magzeropoint=25.947, convolvebox=[500,500], verbose=True)

galfitoutput = tu.galfit_run(galfitinputfile,noskyest=False)
```

#### Remove Sources (Models) from Initial Data Cube

More details comning soon...
```python
import tdose_modify_cube as tmoc

sourcemodelcube = path+'tdose_source_model_cube.fits'
datacube        = path+'datacube.fits'

modified_cube   = tmoc.remove_object(datacube, sourcemodelcube, objects=[1,2,10], remove=True, dataext=0, sourcemodelext=0, savecube='removesource1and2and10', verbose=True)
```

#### Build Mock Data Cube

The following will build a mock data cube based on the source catalog provided of spatial dimensions (200,150) and 100 wavelength slices. A Moffat PSF will be applied and Gaussian noise will be added.
```python
import tdose_build_mock_cube as tbmc

sourcecat       = 'mock_cube_sourcecat161213_all.fits'
cube_dim        = [100,200,150]
noisetype       = 'gauss'
noise_gauss_std = 0.03
psf             = 'moffat'
psf_param       = [10.0,[1.1,1.3,1.5]]

outputcube      = tbmc.build_cube(sourcecat, cube_dim=cube_dim, clobber=False, noisetype=noise, noise_gauss_std=noise_gauss_std, psf=psf, psf_param=psf_param)
```
Here, the `sourcecat` is a FITS catalog containing x and y pixel positions, a flux scale, indicators of the source (model) types and spectral (model) types. For further details see header of `tbmc.build_cube()`. The `psf*` paramters define the psf model to convolve the cube with using `tdose_utilities.gen_psfed_cube()`.

## TDOSE Output Overview

The following presents and overview and description of the main outputs generated with TDOSE. The generation of a number of these can be constrolled from the TDOSE setup file. In the list below the [`name`] refers to the TDOSE setup parameter defining the name extension/generation of the output.

- Reference Image Model [`model_image_ext`]; 2D FITS Image
  - Model of the reference image (cutout). The high resolution reference image defines the morphology of the source before convolution with the data cube PSF. This model is generated by fitting multi-variate Guassians to the sources if `source_model=gauss` in the TDOSE setup file.
- Source Model Parameters [`model_param_reg`]; FITS Table
  - The Gaussian model parameters describing the source models making up the reference image model are stored in this FITS table.
- Image Model in Cube "WCS frame"  [`model_image_cube_ext`]; 2D Fits Image
  - Image showing the reference image model in the "WCS frame" of the data cube to extract spectra from, i.e., converting the pixel scales of the high-resolution reference image model.
- PSF Cube [`psf_savecube`]; 3D FITS Cube
  - Cube with the instrument model PSF spatially centered capturing the wavelength evolution as defined in the TDOSE setup file. The extension of this cube will be `*_psfcube_[source_model].fits` by default.
- Data Cube Model [`model_cube_ext`]; 3D FITS Cube
  - The model data cube after convolution with the instrument PSF including the wavelength dependent flux-optimized source profile(s).
- Residual Data Cube Model  [`residual_cube_ext`]; 3D FITS Cube
  - Data cube containing the residual from subtracting the data cube model from the input data cube (i.e., data cube - data cube model).
- Source Model Cube [`source_model_cube_ext`]; 4D FITS Cube
  - Data cube containing the de-blended source models for all sources in the field-of-view modeled, i.e. the dimensions are [source number, wavelength, y-axis, x-axis]. This cube can be used to modify the input data cube by removing individual sources (i.e. source models) with `tdose_modify_cube.py`.
- Spectrum [`spec1D_name`]; FITS Table
  - Stored 1D spectrum of individual objects. An object spectrum can be an arbitrariy combination of N sources. However, by default each object corresponds to a single source and the number of sources therefore equals the number of extracted object spectra.


## References 

- [Kamann, Wisotzki and Roth (2013)](http://adsabs.harvard.edu/abs/2013A%26A...549A..71K)
- Schmidt et al. in prep.

## Schematic Overview of TDOSE
`TDOSE_illustration_v2.png` (displayed below) presents a schemative overview of the different elements making up TDOSE and the workflow for the available extractions. The outputs shown are described in [TDOSE Output Overview](#tdose-output-overview) above.

<a href="TDOSE_illustration.png"><img src="TDOSE_illustration_v2.png" align="center" height="1200" ></a>


