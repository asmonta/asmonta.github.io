---
title: Useful Anaconda Commands
categories: [Python]
tags: [Python]
render_with_liquid: false
---

[Anaconda <span class="fa-solid fa-arrow-up-right-from-square"/>](www.anaconda.com/download) is a python distribution 

# Useful commands
Here are several useful commands for conda.
## Environments
Environments are python installations that use a specific collection of packages.
```bash
conda create --name ENVNAME python=VER.NUMBER
```

### Print all environments  
```bash
conda env list
```
### Print information of all packages in the active environment
```bash
conda list
```

### Save list of packages in active environment
```bash

```

## Package Installation
To install a package, use
```bash
conda install [-y] PACKAGE_NAME
```
### Package Version:

| Name       | Operand  | Example              | Result                                            |
| :--------- | :------: | :------------------- | :------------------------------------------------ |
| Fuzzy      | =        | numpy=1.1            | 1.11.0, 1.11.1, 1.11.2, etc                       |
| Exact      | ==       | numpy==1.11          | 1.11.0                                            |
| Newer than | >=       | numpy>=1.11          | versions newer than 1.11.0, less than is similar  |
| Or         | \|       | numpy=1.11.1\|1.11.3 | 1.11.1 OR 1.11.3                                  |
| Range      | >=,<     | numpy>=1.8,<2        | 1.8.0,1.8.1,... 1.9.0, 1.9.1,....1.9.999999 NOT 2 |




## Copy Environments
### Local Machine Copy:
```bash
conda create --name CLONED_ENV_NAME --clone ENV_TO_CLONE
```

### Cross Machine/Operating Systems
On machine with master environment active:
```bash
conda env export environment.yml
```
On slave machine(s):
```bash
conda env create -f environment.yml
```
New enviroments will be named ____.yml.

For machines using the same OS, a plain-text file alternative would be:
Master machine:
```bash
conda list --explicit PACKAGE_LIST.txt
```
Slave machine(s):
```bash
conda create --name NEW_ENV_NAME --file PACKAGE_LIST.txt
```

### Conda-Pack
Rather than constructing environments from a list, an entire repo can be copied. Useful for complex installations.

Install conda-pack
```bash
conda install -c conda-forge conda-pack
```

Master machine:
```bash
conda pack -n MY_ENV # Pack environment MY_ENV into MY_ENV.tar.gz
```

On slave machine(s):
```bash
mkdir -p MY_ENV                   # or create manually
tar -xzf MY_ENV.tar.gz -C my_env  # unzip environment

source MY_ENV/bin/activate        # activate the environment

(MY_ENV) $ python                 # Run Python from in the environment


(MY_ENV) $ conda-unpack           # Cleanup prefixes from in the active environment.
```
Read more [here <span class="fa-solid fa-arrow-up-right-from-square"/>](www.anaconda.com/blog/moving-conda-environments).

## Set up Conda to PATH variable (Windows)
For automatons, it is useful to set up Conda to run from the command line

1. Get conda's path
In 'anaconda prompt' run 
```bash
where conda
```
Which should return something like:
```bash
C:\\Users\\USERNAME\\PATH_TO\\anaconda3\\Library\\bin\\conda.bat
C:\\Users\\USERNAME\\PATH_TO\\anaconda3\\Scripts\\conda.exe
C:\\Users\\USERNAME\\PATH_TO\\anaconda3\\condabin\\conda.bat
```
Make a note of these paths

2. Open up Environmental Variables
Under 'System Properties -> Advanced -> Environment Variables'

3. Under 'System Variables' (Second entry), edit 'Path'
Add three entries similar to the ones found in step one:
```bash
C:\\Users\\USERNAME\\PATH_TO\\anaconda3
C:\\Users\\USERNAME\\PATH_TO\\anaconda3\\Scripts
C:\\Users\\USERNAME\\PATH_TO\\anaconda3\\Library\\bin
```