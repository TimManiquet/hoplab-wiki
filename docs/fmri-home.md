# Functional MRI

This page is a work in progress and it is based on what I do in my fMRI pipeline. This info may change once we agree on shared practices.

IMPORTANT: talk to Joan and Elahe to check on what they do, and standardize.

TODO: Add info and links about fmri tasks, preprocessing, GLM, ROIs, MVPA/RSA.

TODO: info on how to install main tools used for the fmri workflow for different OSs.

TODO: info about instruments and procedures at the hospital.

A template folder structure, along with the code to reproduce these analyses, can be found at [PLACEHOLDER]

# General Notes

### How to store raw data

In order to avoid error while converting into BIDS format, the raw data (i.e., the data collected from the scanner, behavioural measure, eye-tracking) should be stored in a folder with the following structure:

```
sourcedata
└── sub-01
    ├── bh
    │   ├── 20240503104938_log_41-1-2_exp.tsv
    │   ├── 20240503105558_41_1_exp.mat
    │   ├── 20240503105640_log_41-2-1_exp.tsv
    │   ├── 20240503110226_41_2_exp.mat
    │   ├── 20240503110241_log_41-3-2_exp.tsv
    └── nifti
        ├── SUBJECTNAME_WIP_CS_3DTFE_8_1.nii
        ├── SUBJECTNAME_WIP_Functional_run1_13_1.nii
        └── SUBJECTNAME_WIP_Functional_run2_12_1.nii
```

### How to get images from the scanner

For optimal BIDS conversion of fMRI data, it is recommended to initially collect DICOM files (not NIfTI or PAR/REC) at the scanner. Although this adds an extra conversion step and takes longer, it ensures proper conversion into BIDS format. Here is the recommended process:

1. **Initial DICOM Collection**:
    - Collect DICOM files for each modality (e.g., T1 and BOLD) for one subject.
    - Convert these DICOM files to NIfTI format, which will generate JSON sidecar files.

2. **Template Creation**:
    - Rename the JSON files for T1 and BOLD image to `sub-xx_T1w.json` and `sub-xx_task-exp_run-x_bold.json`
    - Move the JSON files into `misc/`.

3. **Subsequent Data Collection**:
    - After creating the template JSON files, collect future data in NIfTI format to save time. The `01_nifti-to-BIDS.m` script will use the JSON templates to populate the BIDS folders, provided that the fMRI sequence remained unchaged (in that case you need to generate new templates from the DICOM files).

### Missing fields in JSON files

Despite these steps, some BIDS fields in the sidecar JSON files may remain empty due to limitations of the Philips scanner, not the conversion tools. The most relevant fields that are left empty due to these limitations are `SliceTiming` and `PhaseEncodingDirection`.

- **SliceTiming**:
    - This field is used by fMRIPrep during slice timing correction.
    - It can be populated using the `/utils/get_philips_MB_slicetiming.py` script, assuming you have access to a DICOM file and know the multiband factor (default is 2, as used in our lab).
    - Note: The script assumes an interleaved, foot-to-head acquisition, and will not work for other types of acquisitions.

- **PhaseEncodingDirection**:
    - This BIDS tag allows tools to undistort images.
    - While the Philips DICOM header distinguishes the phase encoding axis (e.g., anterior-posterior vs. left-right), it does not encode the polarity (A->P vs. P->A).
    - You will need to check at the scanner or consult with Ron whether the polarity is AP or PA, and correct the `?` in the JSON file to `+` or `-`.

For more details on Philips DICOM conversion, refer to the following resources:

- [Philips DICOM Missing Information - dcm2niix](https://github.com/rordenlab/dcm2niix/tree/master/Philips#missing-information)
- [PARREC Conversion - dcm2niix](https://github.com/rordenlab/dcm2niix/tree/master/PARREC)

### Where to find additional info on the fMRI sequence

Additional information on the sequence can be found at the scanner by following these steps:
	
	- TODO: document the correct steps to get info on the geometry etc. we need to start new examination, load our sequence, click on one run/T1, and go in the geometry tab. here we have info about polarity, direction etc.
	
### BIDS standards

TODO: add info about the BIDS standard, and how we use it (from raw to BIDS + derivatives)
	
# Workflow

## Behavioral Data

TODO: here Tim should describe name and format of the logfiles

TODO: add ref to fMRI task repo

The fMRI task script should give two files per run as output. Here is a description of the naming structure and file content:

1. `<timestamp>_log_<subID>-<run>-<buttonMapping>_<taskName>.tsv`: [PLACEHOLDER]
2. `<timestamp>_log_<subID>_<run>_<taskName>.mat`: [PLACEHOLDER]

If the behavioural data is stored in a `sourcedata/sub-xx/bh/` folder consistent to the one described [above](#how-to-store-raw-data), you can run the `02_behavioural-to-BIDS.m` script, after editing the parameters at the top of the script. This script iterates through subject-specific directories, targeting behavioral .mat files, then processes and exports trial-related data into BIDS-compliant TSV event files. 

TODO: above, we need to phrase better and add more info about what these parameters are, possibly with a screenshot of the code. Also, in the code make more clear where the parameters are.
TODO: perhaps wrap nifti and bh 2 BIDS in a single script that takes some input arguments? 
TODO: we need to add more info about how these files are saved, and more in general about the BIDS structure

Ensure that each resulting tsv file has at least three columns representing: `onset`, `duration`, `trial_type`.

## Eye-Tracking Data

**Pre-requisite:** To convert EDF files into more easy-to-read ASC files, you need to install the EyeLink Developers Kit / API. More info and how-to:

- [EyeLink Developers Kit / API](https://www.sr-research.com/support/thread-13.html)

1. Run the ET script to convert to ASC in BIDS format.

**Note:** BEP020 has not been approved yet. Not sure if the events MSG should be included here or not.

## fMRI Data

### BIDS conversion

1. **Convert DICOM to BIDS (NIfTI):**
    - **Prerequisites:**
        - `dcm2nii`
        - Raw data should be organized as: `/sourcedata/sub-<xx>/dicom`
        - MATLAB

    1.1. Download `dicm2nii` from [dicm2nii](https://github.com/xiangruili/dicm2nii), unzip, and add to the MATLAB path.

    1.2. Open MATLAB and type `anonymize_dicm` in the console. Enter and it will ask you to select the folder where the files are and the folder in which you want to save it: `/sourcedata/sub-<xx>/dicom_Anon`.

    1.3. Run the `dicm2nii` MATLAB function from the unzipped file. Just write `dicm2nii` in the command window. A pop-up will appear: select DICOM folder (Dicom_Anon. files) and result folder (Dicom_Converted). Untick the compress box and make sure you select save JSON file. Start conversion. A pop-up appears: subject (only number of the subject, e.g., 01). Type: `func` (T2 scans) and `anat` (T1 scans). Under modality, we need `task-{name of the task}_run-{number of run}_bold`, e.g., `task-exp_run-2_bold`. Modality for anatomical scans is `T1W`. This pop-up will appear for each run of the fMRI sequence.

    At the end of this process, we have a new folder called `dicom_converted` structured as follows:
    ```
    dicom_converted
    ├── sub-01
    │   ├── anat
    │   │   ├── sub-01_T1w.json
    │   │   └── sub-01_T1w.nii.gz
    │   ├── func
    │   │   ├── sub-01_task-exp_run-1_bold.json
    │   │   └── sub-01_task-exp_run-1_bold.nii.gz
    │   └── dcmHeaders.mat
    └── participants.tsv
    ```

    Copy your `sub-01` folder from `dicom_converted` into the BIDS folder (make a new one if you don't have one).

    **Note:** The pop-up only appeared with the first participant and then does it automatically. This is quite annoying as it never gets the participant number right so you have to manually go in and change it. It also only worked when providing it with (enhanced) DICOM files.

2. **Validate the BIDS directory (and solve errors):**
    - [BIDS Validator](https://bids-standard.github.io/bids-validator/)

### Surface preprocessing (FastSurfer)


### Minimal preprocessing (fMRIprep)

TODO: explain how to install docker and fmriprep-docker

```sh
fmriprep-docker \
    /data/BIDS \
    /data/BIDS/derivatives/fmriprep \
    participant \
    --work-dir /chess/temp \ 
    --mem-mb 13000 \
    --n-cpus 16 \
    --resource-monitor \
    -vv \
    --output-spaces MNI152NLin2009cAsym:res-2 anat fsnative \
    --fs-license-file /misc/.license \
    --bold2t1w-dof 9 \
    --task exp \
    --dummy-scans 0 \
    --fs-subjects-dir /data/BIDS/derivatives/fastsurfer \
    --notrack \
    --participant-label 37 38 39 40
```

#### How to interpret fMRIprep visual reports

### Quality check (mriqc)

TODO: explain how to save and run this below
TODO: explain that the one below may fail, in that case run the single commands separately

```sh
#!/bin/bash

# Iterate over subject numbers
for i in {37..40}; do
    # Skip subject 0, 5, 14, 31
    if [ $i -eq 0 ] || [ $i -eq 5 ] || [ $i -eq 14 ] || [ $i -eq 31 ]; then
        continue
    fi

    # Format subject number with leading zeros
    subID=$(printf "sub-%02d" $i)
    echo "Processing $subID"

    # Docker command with dynamic subject ID using sudo
    docker run -it --rm \
    -v /data/BIDS/derivatives/fmriprep:/data:ro \
    -v /data/BIDS/derivatives/mriqc:/out \
    -v /temp_mriqc:/scratch \
    nipreps/mriqc:latest /data /out participant \
    --participant-label ${subID} \
    --nprocs 16 --mem-gb 40 --float32 \
     --work-dir /scratch \
     --verbose-reports --resource-monitor -vv

    # Wait or perform other actions between runs if needed
    sleep 0.5
done

echo "Running group analysis"

docker run -it --rm \
-v /data/BIDS/derivatives/fmriprep:/data:ro \
-v /data/BIDS/derivatives/mriqc:/out \
-v /temp_mriqc:/scratch \
nipreps/mriqc:latest /data /out group \
--nprocs 16 --mem-gb 40 --float32 \
 --work-dir /scratch \
 --verbose-reports --resource-monitor -vv

sleep 0.5

echo "Running classifier"
docker run \
-v /temp_mriqc:/scratch \
-v /data/BIDS/derivatives/mriqc:/resdir \
-w /scratch --entrypoint=mriqc_clf poldracklab/mriqc:latest \
 --load-classifier -X /resdir/group_T1w.tsv
```

### Running a GLM in SPM

TODO: show code snippets perhaps? or just reference to the code

### Regions of Interest

#### ROIs from localizers

#### HCP Glasser parcellation

#### Other parcels/atlases

### MVPA/RSA

### Plotting and reporting



