# biac-analysis-resting_pipeline
Python/FSL Resting State Pipeline

This pipeline is a collection of steps that can be used to process a single subject's resting state data from raw into a node based correlation matrix representing connectivity between different regions of the brain.


# Install

Source files assume you have a working install of Python2.7, FSL, and all imported python modules.
Python modules are numpy, nibabel, scipy, and networkx.

Needs a working install of the [BIRN BXH/Xcede tools](https://www.nitrc.org/projects/bxh_xcede_tools/).

# Usage

```
Usage: 
resting_pipeline.py --func /path/to/run4.bxh --steps all --outpath /here/ -p func

Program to run through Nan-kuei Chen's resting state analysis pipeline:
    steps:
    0 - convert data to nii in LAS orientation ( we suggest LAS if you are skipping this step )
    1 - slice time correction
    2 - motion correction, then regress out motion parameter
    3 - skull stripping
    4 - normalize data
    5 - regress out WM/CSF
    6 - bandpass filter
    7 - do parcellation and produce correlation matrix from label file
      * or split it up:
         7a - do parcellation from label file
         7b - produce correlation matrix [--func option is ignored if step 7b
              is run by itself unless --dvarsthreshold is specified, and
              --corrts overrides default location for input parcellation
              results (outputpath/corrlabel_ts.txt)]
    8 - functional connectivity density mapping



Options:
  -h, --help            show this help message and exit
  -f /path/to/BXH, --func=/path/to/BXH
                        bxh ( or nifti ) file for functional run
  --throwaway=4         number of timepoints to dis-regard from beginning of
                        run
  --t1=/path/to/BXH     bxh ( or nifti ) file for the anatomical T1
  -p func, --prefix=func
                        prefix for all resulting images, defaults to name of
                        input
  -s 0,1,2,3, --steps=0,1,2,3
                        comma seperated string of steps. 'all' will run
                        everything, default is all
  -o PATH, --outpath=PATH
                        location to store output files
  --sliceorder=string   sliceorder if slicetime correction ( odd=interleaved
                        (1,3,5,2,4,6), up=ascending, down=descending,
                        even=interleaved (2,4,6,1,3,5) ).  Default is to read
                        this from input image, if available.
  --tr=MSEC             TR of functional data in MSEC
  --ref=FILE            pointer to FLIRT reference image if not using standard
                        brain
  --flirtmat=FILE       a pre-defined flirt matrix to apply to your functional
                        data. (ie: func2standard.mat)
  --refwm=FILE          pointer to WM mask of reference image if not using
                        standard brain
  --refcsf=FILE         pointer to CSF mask of reference image if not using
                        standard brain
  --refgm=FILE          pointer to GM mask of reference image if not using
                        standard brain
  --refbrainmask=FILE   pointer to brain mask of reference image if not using
                        standard brain
  --refacpoint=45,63,36
                        AC point of reference image if not using standard MNI
                        brain
  --betfval=0.4         f value to use while skull stripping. default is 0.4
  --anatbetfval=0.5     f value to use while skull stripping ANAT. default is
                        0.5
  --lpfreq=0.08         frequency cutoff for lowpass filtering in HZ.  default
                        is .08hz.  highpass is fixed at .001hz.
  --corrlabel=FILE      pointer to 3D label containing ROIs for the
                        correlation search. default is the 116 region AAL
                        label file
  --corrtext=FILE       pointer to text file containing names/indices for ROIs
                        for the correlation search. default is the 116 region
                        AAL label txt file
  --corrts=FILE         If using step 7b by itself, this is the path to
                        parcellation output (default is to use
                        OUTPATH/corrlabel_ts.txt), which will be used as input
                        to the correlation.
  --dvarsthreshold=THRESH
                        If specified, this reprsents a DVARS threshold either
                        in BOLD units, or if ending in a '%' character, as a
                        percentage of mean global signal intensity (over the
                        brain mask).  Any volume contributing to a DVARS value
                        greater than this threshold will be excluded
                        ("scrubbed") from the (final) correlation step.  DVARS
                        calculation is performed on the results of the last
                        pre-processing step, and is calculated as described by
                        Power, J.D., et al., "Spurious but systematic
                        correlations in functional connectivity MRI networks
                        arise from subject motion", NeuroImage(2011).  Note:
                        data is only excluded during the final correlation,
                        and so will never affect any operations that require
                        the full signal, like regression, etc.
  --dvarsnumneighbors=NUMNEIGHBORS
                        If --dvarsthreshold is specified, then
                        --dvarsnumnumneighbors specifies how many neighboring
                        volumes, before and after the initially excluded
                        volumes, should also be excluded.  Default is 0.
  --fdthreshold=THRESH  If specified, this reprsents a FD threshold in mm.
                        Any volume contributing to a FD value greater than
                        this threshold will be excluded ("scrubbed") from the
                        (final) correlation step.  FD calculation is performed
                        on the results of the last pre-processing step, and is
                        calculated as described by Power, J.D., et al.,
                        "Spurious but systematic correlations in functional
                        connectivity MRI networks arise from subject motion",
                        NeuroImage(2011).  Note: data is only excluded during
                        the final correlation, and so will never affect any
                        operations that require the full signal, like
                        regression, etc.
  --fdnumneighbors=NUMNEIGHBORS
                        If --fdthreshold is specified, then
                        --fdnumnumneighbors specifies how many neighboring
                        volumes, before and after the initially excluded
                        volumes, should also be excluded.  Default is 0.
  --motionthreshold=THRESH
                        If specified, any volume whose motion parameters
                        indicate a movement greater than this threshold (in
                        mm) will be excluded ("scrubbed") from the (final)
                        correlation step.  Volume-to-volume movement is
                        calculated per pair of neighboring volumes from the
                        three rotational and three translational parameters
                        generated by mcflirt.  Motion for a pair of
                        neighboring volumes is calculated as the maximum
                        displacement (due to the combined rotation and
                        translation) of any voxel on the 50mm-radius sphere
                        surrounding the center of rotation.  Note: data is
                        only excluded during the final correlation, and so
                        will never affect any operations that require the full
                        signal, like regression, etc.
  --motionnumneighbors=NUMNEIGHBORS
                        If --motionthreshold is specified, then
                        --motionnumnumneighbors specifies how many neighboring
                        volumes, before and after the initially excluded
                        volumes, should also be excluded.  Default is 1.
  --motionpar=FILE.par  If --motionthreshold is specified, then --motionpar
                        specifies the .par file from which the motion
                        parameters are extracted.  If you allow this script to
                        perform motion correction, then this option is
                        ignored.
  --scrubop=SCRUBOP     If --motionthreshold, --dvarsthreshold, or
                        --fdthreshold are specified, then --scrubop specifies
                        the aggregation operator used to determine the final
                        list of excluded volumes.  Default is 'or', which
                        means a volume will be excluded if *any* of its
                        thresholds are exceeded, whereas 'and' means all the
                        thresholds must be exceeded to be excluded.
  --powerscrub          Equivalent to specifying --fdthreshold=0.5
                        --fdnumneighbors=0 --dvarsthreshold=0.5%
                        --dvarsnumneigbhors=0 --scrubop='and', to mimic the
                        method used in the Power et al. article.  Any
                        conflicting options specified before or after this
                        will override these.
  --scrubkeepminvols=NUMVOLS
                        If --motionthreshold, --dvarsthreshold, or
                        --fdthreshold are specified, then --scrubminvols
                        specifies the minimum number of volumes that should
                        pass the threshold before doing any correlation.  If
                        the minimum is not met, then the script exits with an
                        error.  Default is to have no minimum.
  --fcdmthresh=THRESH   R-value threshold to be used in functional
                        connectivity density mapping ( step8 ). Default is set
                        to 0.6. Algorithm from Tomasi et al, PNAS(2010), vol.
                        107, no. 21. Calculates the fcdm of functional data
                        from last completed step, inside a dilated gray matter
                        mask
  --cleanup             delete files from intermediate steps?


```

# Pipeline Details

**Step 0**
Reorient into LAS, convert into niigz.
if –throwaway is defined, this number of timepoints will be disregarded during this step.

**Step 1**
FSL's slice time correction.
If starting with a BXH header, you'll likely have the sliceorder field which will be used to create a custom sliceorder file to be used by FSL.
If this isn't present, then you'll need to define –sliceorder so that we can generate the file for you:
“odd” is interleaved with odd, then even slice ordering: 1,3,5,2,4,6,etc
“even” is also interleaved, but when even slice ordering: 2,4,6,1,3,5,etc 
“up” is ascending data, from the bottom up: 1,2,3,4,5,6
“down” is descending data, from the top down: 6,5,4,3,2,1

**Step 2**
Motion correct data using FSL's mcflirt.
Once mcflirt is completed, the 6 motion parameters are then regressed out of each individual voxel usings a linear regression.

**Step 3**
The functional run is meaned across time with fslmaths, then brain extraction (BET) is applied. Yhe resulting mask is then applied to the entire run of data.

**Step 4**
Normalize the data using flirt.
If no options are specified, then the default is the standard MNI152_T1_2mm_brain used in feat.
If you have a specific template, you can define it with –ref (ie: a kid brain, or study specific template).
if your subject has already been normalized during standard pre-processing of other runs, please provide the flirt matrix from pre-stats 
If you've provided at T1 anatomical image then this sequence is followed:
func-2-t1
t1-2-standard

**Step 5**
Regress out CSF and WM signal using masks generated by FAST.

**Step 6**
Band-pass filter data to remove high-frequency noise using custom python code:
The default lowpass is 0.08 HZ
Highpass is fixed at .001 HZ

**Step 7**
if defaults are used, then the aal_MNI_V4 label file is used to extract the average timeseries for each of the 116 regions of the Automated Anatomical Labelling Atlas. If you want to provide your own label file, the –corrlabel option can be used. The input to this step would be a 3D image of ROIs the same size as your normalized data. Each individual ROI needs to have a unique intensity value for the timecourse extraction.

This step produces 4 files:
“subject.graphml” : graphml format with regions timecourse, zr_vals, r_vals
“corrlabel_ts.txt”: extracted time series for each region
“r_matrix.nii.gz”: the correlation coefficients
“zr_matrix.nii.gz”: normalized correlation coefficients
“mask_matrix.nii.gz”: an inclusion mask for everything below the intersect of the regions.

**Step 8**
Functional connectivity density mapping
Takes functional data from last step and calculates how connected they are to the voxels around them. Adapted from Dardo Tomasi, PNAS(2010), vol. 107, no. 21. 9885–9890

# License
Originaly obtained from https://wiki.biac.duke.edu/biac:analysis:resting_pipeline, under a CC Attribution-Share Alike 4.0 International license. Created by Chou et al. AJNAR(2012), May; 33(5): 833–838.
fcdm algorithm adapted from Dardo Tomasi, PNAS(2010), vol. 107, no. 21. 9885–9890
