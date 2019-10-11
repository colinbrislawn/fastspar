# FastSpar
[![Build Status](https://travis-ci.org/scwatts/fastspar.svg?branch=master)](https://travis-ci.org/scwatts/fastspar)
[![License](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0.en.html)
[![install with bioconda](https://img.shields.io/badge/install%20with-bioconda-brightgreen.svg?style=flat)](http://bioconda.github.io/recipes/fastspar/README.html)

Rapid and scalable correlation estimation for compositional data


## Table of contents
* [Table of contents](#table-of-contents)
* [Introduction](#introduction)
* [Citation](#citation)
* [Requirements](#requirements)
* [Installing](#installing)
* [Usage](#usage)
* [Contributors](#contributors)
* [License](#license)


## Introduction
`FastSpar` is a C++ implementation of the [SparCC algorithm](http://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1002687) which is up to several thousand times faster than the original Python2 release and uses much less memory. The `FastSpar` implementation provides threading support and a *p*-value estimator which accounts for the possibility of repetitious data permutations (see [this](https://arxiv.org/pdf/1603.05766.pdf) paper for further details).

An important step of correlation analysis is removal of noise and dimension reduction. A common method to perform this is distribution-based clustering of OTUs. The aim is to reunite OTUs derived from sequencing error with the parent OTU by clustering raw OTUs based on nucleotide edit distance and count distribution. `FastSpar` is paired with an [efficient implementation](https://github.com/scwatts/otudistclust) of the popular distribution-based clustering method dbOTU3.


## Citation
If you use this tool, please cite the `FastSpar` paper and original SparCC paper:
* [Watts, S. C., Ritchie, S. C., Inouye, M., & Holt, K. E. (2018). FastSpar: rapid and scalable correlation estimation for compositional data. Bioinformatics. doi: 10.1093/bioinformatics/bty734](https://doi.org/10.1093/bioinformatics/bty734)
* [Friedman, J. & Alm, E.J. (2017). Inferring correlation networks from genomic survey data. PLoS Comput. Biol. 8, e1002687.](http://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1002687)


## Requirements
There are no requirements for using the pre-compiled static binaries on 64-bit linux distributions. Otherwise, there are several libraries which are required for building and running dynamically linked binaries. For further information, see [Compiling from source](#compiling-from-source).


## Installing
`FastSpar` can be installed via pre-compiled binaries, Bioconda, or from source.


### GNU/Linux
For most 64-bit linux distributions (e.g. Ubuntu, Debian, RedHat, etc) the easiest way to obtain `FastSpar` is via statically compiled binaries on the releases page. These binaries can be downloaded and run immediately without any setup as they have no dependencies.


### Bioconda
`FastSpar` is available through Bioconda (thanks to [@epruesse](https://github.com/epruesse)):
```bash
conda install -c bioconda -c conda-forge fastspar
```


### Compiling from source
Compiling from source requires these libraries and software:
```
C++11 (gcc-4.9.0+, clang-4.9.0+, etc)
OpenMP 4.0+
Gfortran
Armadillo 6.7+
LAPACK
BLAS (OpenBLAS is recommended)
GNU Scientific Library 2.1+
GNU getopt
GNU make
GNU autoconf
GNU autoconf-archive
GNU m4
```

After meeting the above requirements, compiling and installing `FastSpar` from source can be done by:
```bash
git clone https://github.com/scwatts/fastspar.git
cd fastspar
./autogen.sh
./configure --prefix=/usr/
make
make install
```
Once completed, the `FastSpar` executables can be run from the command line.


## Usage
### Correlation inference
To run `FastSpar`, you must have absolute OTU counts in BIOM tsv format file (with no metadata). The `fake_data.tsv` (from the original SparCC implementation) will be used as an example:
```bash
fastspar --otu_table tests/data/fake_data.tsv --correlation median_correlation.tsv --covariance median_covariance.tsv
```

The number of iterations (rounds of SparCC correlation estimation) and exclusion iterations (the number of times highly correlation OTU pairs are discovered and excluded) can also be tweaked:
```bash
fastspar --iterations 50 --exclude_iterations 20 --otu_table tests/data/fake_data.tsv --correlation median_correlation.tsv --covariance median_covariance.tsv
```

Further, the minimum threshold to exclude correlated OTU pairs can be increased:
```bash
fastspar --threshold 0.2 --otu_table tests/data/fake_data.tsv --correlation median_correlation.tsv --covariance median_covariance.tsv
```


### Calculation of exact *p*-values
There are several methods to calculate *p*-values for inferred correlations. Here we have elected to use a robust permutation based approach. This process involves inferring correlation from random permutations of the original OTU count data. The magnitude of each *p*-value is related to how often a more extreme correlation is observed for randomly permutated data. In the below example, we calculate *p*-values from 1000 bootstrap correlations.

First we generate the 1000 bootstrap counts:

```bash
mkdir bootstrap_counts
fastspar_bootstrap --otu_table tests/data/fake_data.tsv --number 1000 --prefix bootstrap_counts/fake_data
```

And then infer correlations for each bootstrap count (running in parallel with all processes available):

```bash
mkdir bootstrap_correlation
parallel fastspar --otu_table {} --correlation bootstrap_correlation/cor_{/} --covariance bootstrap_correlation/cov_{/} -i 5 ::: bootstrap_counts/*
```

From these correlations, the *p*-values are then calculated:
```bash
fastspar_pvalues --otu_table tests/data/fake_data.tsv --correlation median_correlation.tsv --prefix bootstrap_correlation/cor_fake_data_ --permutations 1000 --outfile pvalues.tsv
```


### Threading
If `FastSpar` is compiled with OpenMP, threading can be used by invoking `--threads <thread_number>` at the command line for several tools:
```bash
fastspar --otu_table tests/data/fake_data.txt --correlation median_correlation.tsv --covariance median_covariance.tsv --iterations 50 --threads 10
```


## Contributors
* **[Scott Ritchie](https://github.com/sritchie73)**
  * Advised on use of permutation based statistical testing
  * Provided an example use of `statmod::permp`


## License
[GNU General Public License v3.0](https://www.gnu.org/licenses/gpl-3.0.en.html)
