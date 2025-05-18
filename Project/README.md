# cs185c_spring2025
Welcome to my ocean modeling repository! This repository is designed to host and store my notes on modeling the Earth's Ocean using Python and Jupyter Notebook, receiving data from MITgcm and GEBCO, which has Bathymetry Data. 

There are two key directories in this repository:

## Homework
The homework directory is for storing, managing, and turning in weekly homework assignments.

## Project
For this class's final project, I will research the effects of Typhoons on the Philippine Region. Specifically, I will be finding the answer to this question: 

_How does climate change and the increase of global temperatures affect Typhoon activity?_

To investigate, I will construct my model on the Philippines and surrounding regions and run the model for two separate years, focusing primarily on the years 1998 and 2011, when the Philippines had La Nina, bringing in more precipitation. I anticipate the model to have an increase in Typhoon activity over the years due to increase in global temperatures. These years were chosen from  [Columbia' Climate Prediction Center ENSO Forecast ](https://iri.columbia.edu/our-expertise/climate/forecasts/enso/current/?enso_tab=enso-sst_image). For my project, I will be primarily using data from the ECCO Version 5 model in the years provided. During those years, I will observe each month to month. I observe results through timeseries that analyzes the Ocean temperatures and investigate the differences over time. For visualization, I will create a movie of temperature differences between the years.

Before running this experiment, my initial hypothesis on the quesetion is that Typhoon activity and severity has been increasing over the years. The reasoning behind this is that La Nina becomes more intense over the year due to increases in the ranges of global temperatures, which will in turn, generate more typhoons that wreck the Phillipines. During these La Nina years in the Phillipines, rainfall increases. I will observe the Sea Surface height as when typhoons come, it brings colder waters up and lowers the Sea Surface height for that location. 

My results and conclusion can be observed in the 'Answering Science Question' notebook.


## Brief rundown of steps to run on the Spartan Cluster
clone the github repo onto Spartan
```git clone ```

create a file system in the configurations folder of MITgcm
- configurations
- - githubrepo
- - coastal_philippines
- - - input
- - - build
- - - code
- - - namelist
- - - run
to send data over to the Spartan cluster from my local computer to copy over the exf and obcs directories
```
scp 1998/exf/* cs185c10@spartan03.sjsu.edu:/scratch/cs185c10/MITgcm/configurations/coastal_philippines/input/exf
scp 1998/obcs/* cs185c10@spartan03.sjsu.edu:/scratch/cs185c10/MITgcm/configurations/coastal_philippines/input/obcs
scp Philippines_bathymetry.bin cs185c10@spartan03.sjsu.edu:/scratch/cs185c10/MITgcm/configurations/coastal_philippines/input
scp 1998/* cs185c10@spartan03.sjsu.edu:/scratch/cs185c10/MITgcm/configurations/coastal_philippines/input/
```

```
scp 2011/exf/* cs185c10@spartan03.sjsu.edu:/scratch/cs185c10/MITgcm/configurations/coastal_philippines_2/input/exf
scp 2011/obcs/* cs185c10@spartan03.sjsu.edu:/scratch/cs185c10/MITgcm/configurations/coastal_philippines_2/input/obcs
scp Philippines_bathymetry.bin cs185c10@spartan03.sjsu.edu:/scratch/cs185c10/MITgcm/configurations/coastal_philippines_2/input
scp 2011/* cs185c10@spartan03.sjsu.edu:/scratch/cs185c10/MITgcm/configurations/coastal_philippines_2/input/
```

copy over the namelist and code folder information from the github repo you cloned into coastal philippines
(command below used while in the coastal philippines namelist folder):
```
cp ../../cs185c_spring2025/Project/namelist/* .
```

activate the conda cs185c:
```
conda activate --stack cs185c
```

generate the build mitgcmuv:
```
cd build
export MPI_HOME=/home/cs185c10/.conda/envs/cs185c
../../../tools/genmake2 -of ../../../tools/build_options/linux_amd64_gfortran -mods ../code -mpi
make depend
make
```

link the files from namelist, build mitgcmuv, and input onto run
```
cd ../run
ln -s ../input/* .
ln -s ../build/mitgcmuv .
ln -s ../namelist/* .
```

create a diags folder for each thing mentioned in the data.diagnostics file
```
mkdir diags
mkdir EtaN_mon_mean
mkdir EtaN_day_snap
mkdir vel_3D_mon_snap
mkdir TS_3D_mon_mean
mkdir TS_surf_daily_mean
```

create a slurm file
```
nano cs185c10.slm
```

change the ntasks to the # of CPU's used mentioned in the SIZE.h file
```#!/bin/bash
#SBATCH --partition=nodes
#SBATCH --nodes=1
#SBATCH --ntasks=2
#SBATCH --time=10:00:00
export MPI_HOME=/home/[username]/.conda/envs/cs185c
mpirun -np 2 ./mitgcmuv
```

clear diags folder

```
rm diags/EtaN_day_snap/*
rm diags/EtaN_mon_mean/*
rm diags/TS_3D_mon_mean/*
rm diags/TS_surf_daily_mean/*
rm diags/vel_3D_mon_snap/*
```

Download all data files from the diags folder into your run/diags directory.

```
scp -r cs185c10@spartan03.sjsu.edu:/scratch/cs185c10/MITgcm/configurations/coastal_philippines/run/diags/ . 
``` 

Errors encountered:
- pot temp extreme
- Mismatched years when looking at data files (exf, obcs, etc.) and years being observed
- Mismatched number of processors
- Initial Conditions, External Conditions, and Boundary Conditions had a different bathymetry file due to not reuploading all files