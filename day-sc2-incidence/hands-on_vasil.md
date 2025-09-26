# Hands-on session Day 4: Cross-immunization & Relative Fitness

## Introduction
In this hands-on session, we will compute the cross-immunization between several variants, the expected number of susceptibles to a given variant, and the relative fitness of a given variant. We will use Jupyter Notebook. 

## Input files

All input files can be found [here](https://github.com/AlexiaNomena/SC2_VASIL). Please download the whole repository.
The input file for this hands-on is `Hands_On.ipynb`

## 1. Mutation profiles
Mutation profiles can be found and extracted from [outbreak.info](https://outbreak.info/)
Please visit [here](https://outbreak.info/compare-lineages?pango=JN.1&pango=EG.5&gene=S&threshold=75&nthresh=1&dark=false) to check the differences in the S protein between the two variants e.g.: JN.1 and EG.5.

## 2. Getting the mutation profiles
There are two methods

### 1st method
Get a GISAID user account: [GISAID](https://gisaid.org/register/) (This might take a few days!)
Use the R code `Hands_On.R` to extract the mutation profiles of any variant of your choice from outbreak.info
For this, you will need to install R and RStudio on the command line (info [here](https://anaconda.org/conda-forge/r-base) and [here](https://anaconda.org/r/rstudio))

### 2nd method
Just copy and paste the mutation profiles displayed on the outbreak.info page into a .txt file (we will show you in class)

## 3a. Necessary software and packages

- Please install the Anaconda Distribution on your computer https://www.anaconda.com/docs/getting-started/anaconda/install
- Within the Anaconda Navigator, you can install Jupyter Notebook
- Then you can launch Jupyter Notebook, and it will show in your browser

## 3b. Alternative software and packages

- You can copy and paste all the cells in `Hands_On.ipynb`into a new python file e.g.: `Hands_On.py` (within the same folder)
- Everything should run fine with a pre-installed python 3 version

## NB:
If there is time, we will show you how to create environments in Anaconda and use it on your Jupyter Notebook
If you have any installation problems, feel free to come to Arnimallee 3 Room 112 with your computer between 11 am and 4 pm
