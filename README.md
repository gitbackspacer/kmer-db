# Kmer-db

## REQUIREMENTS

Kmer-db requires a CPU supporting AVX extensions (AVX2 is also used, though it is not obligatory).

## INSTALLATION AND CONFIGURATION

Kmer-db comes with a set of precompiled binaries for Windows and Linux. 
The software can be also built from the sources distributed as:

* MAKE project (G++ 4.8 required) for Linux and OS X.
* Visual Studio 2015 solution for Windows,


### *zlib* linking

Kmer-db uses *zlib* for handling gzipped inputs. Under Linux, the software is by default linked against system-installed *zlib*. Due to issues with some library versions, precompiled *zlib* is also present the repository. In order to use it, one needs to modify variable INTERNAL_ZLIB at the top of the makefile. Under Windows, the repository library is always used.

### AVX2 support

Kmer-db by default takes advantage of AVX and AVX2 CPU extensions. Pre-built binary detetermines supported instructions at runtime, thus it is multiplatform. However, one may encounter a problem when building default Kmer-db version on a CPU without AVX2. To prevent from using AVX2, the program must be compiled with NO_AVX2 symbolic constant defined. When building under Linux or OS X, there is a NO_AVX2 switch at the top of the makefile which does the job.

## USAGE
`kmer-db <mode> [options] <positional arguments>`

Kmer-db operates in one of the following modes:

* `build` - building a database from genomes,
* `build-kmers` - building a database from k-mers,
* `build-mh` - building a database from minhashed k-mers,
* `minhash` - minhashing k-mers,
* `all2all` - calculating number of common k-mers between all samples in the database,
* `one2all` - calculating number of common kmers between single sample and all the samples in the database,
* `distance` - calculating similarities/distances.
    
Options:

* `-t <threads>` - number of threads (default: number of available cores),
* `-buffer <size_mb>` - size of cache buffer in megabytes, applies to `all2all` mode (use L3 size for Intel CPUs and L2 for AMD to maximize performance; default: 8).
    
The meaning of the positional arguments depends on the selected mode.
    
### Building the database
Construction of k-mers database is an obligatory step for further analyses. The procedure accepts several input types:
* compressed or uncompressed genomes:

    ```kmer-db build [-f <filter> -k <kmer-length>] <sample_list> <database>```
* [KMC-generated](https://github.com/refresh-bio/KMC) k-mers: 

    ```kmer-db build-kmers [-f <filter>] <sample_list> <database>```
  
* minhashed k-mers produced by `minhash` mode:

    ```kmer-db build-mh <sample_list> <database>```

Parameters:
* `sample_list` (input) - file containing list of samples in the following format:
    ```
    sample1
    sample2
    sample3
    ...
    ```
    In `build` mode, the tool requires compressed (*.gz* / *.fna.gz*) or uncompressed (*.fna*) genome files for each sample (extensions are added automatically). In `build-kmer` mode, corresponding [KMC-generated](https://github.com/refresh-bio/KMC) k-mer files (*.pre* and *.suf*) are required. In `build-mh` mode, minhashed k-mer files (*.minhash*) must be generated by `minhash` command prior to the database construction. Note, that minhashing may be done during the database construction by specyfying `-f` option.

Options:
* `-f <filter>` - number from [0,1] interval determining a fraction of all k-mers to be accepted by the minhash filter during database construction (default: 1; applies to `build` and `build-kmer` modes).
* `-k <kmer-length>` - length of k-mers (default: 18; applies to `build` mode). 

 
### Minhashing k-mers
This is an optional analysis step which stores minhashed k-mers on the hard disk to be later consumed by `build-mh`. It can be skipped if one wants to use all k-mers from samples for distance estimation or employs minhashing during database construction. Syntax:
`kmer-db minhash <sample_list> <filter>`

Parameters:
 * `sample_list` (input) - file containing list of [KMC-generated](https://github.com/refresh-bio/KMC) k-mer files for samples (a pair of *.pre* and *.suf* files per sample is needed). 
 * `filter` (input) - number from [0,1] interval determining a fraction of all k-mers to be accepted by the filter.
 
  For each sample from the list, a binary file with *.minhash* extension containing filtered k-mers is created.

 
 ### Calculating number of common k-mers ###
Calculating number of common k-mers for all the samples in the database:
 
 `kmer_db all2all <database> <common_matrix>`
 
Parameters:
* `database` (input) - k-mer database file created by `build` mode,
* `common_matrix` (output) - file containing matrix with common k-mer counts.

Calculating number of common kmers between single sample and all the samples in the database:

`kmer_db one2all <database> <sample> <common_vector>`

Parameters:
 * `database` (input) - k-mer database file created by `build` mode,
 * `sample` (input) - raw k-mer file for a sample (minhashing is done automatically, if neccessary),
 * `common_vector` (output) - file containing vector with numbers of common k-mers.
 
 ### Calculating similarities/distances
 
`kmer_db distance <common_file>"`

Parameters:
* `common_file` (input) - file containing matrix (vector) with numbers of common k-mers produced by `all2all` (`one2all`) mode.

This mode generates a set of files containing matrices (vectors) with different similarity/distance measures:
* *<common_file>.jaccard*
* *<common_file>.min*
* *<common_file>.max*
* *<common_file>.cosine*
* *<common_file>.mash*

### Toy examples

Let *pathogens.list* be the file containing names of samples (there exist *.gz* or *.fasta* genome file for each sample):
```
acinetobacter
klebsiella
e.coli
...
```

Calculating similarities/distances between all samples listed in *pathogens.list* using all 20-mers: 
```
kmer-db build -k 20 pathogens.list pathogens.db
kmer-db all2all pathogens.db matrix.csv
kmer-db distance matrix.csv
```

Same as above, but using only 10% of 20-mers:
```
kmer-db build -k 20 -f 0.1 pathogens.list pathogens.db
kmer-db all2all pathogens.db matrix.csv
kmer-db distance matrix.csv
```

Calculating similarities/distances between samples listed in *pathogens.list* and *salmonella* using all 20-mers: 
```
kmer-db build -k 20 pathogens.list pathogens.db
kmer-db one2all pathogens.db salmonella vector.csv
kmer-db distance vector.csv
```

Same as above, but using only 10% of 20-mers:
```
kmer-db build -k 20 -f 0.1 pathogens.list pathogens.db
kmer-db one2all pathogens.db salmonella vector.csv
kmer-db distance vector.csv
```

## Datasets
List of the pathogens investigated in Kmer-db study can be found [here](https://github.com/refresh-bio/kmer-db/tree/master/data)

## Citing
[Deorowicz, S., Gudyś, A., Długosz, M., Kokot, M., Danek, A. (2018) Kmer-db: instant evolutionary distance estimation, Bioinformatics](https://academic.oup.com/bioinformatics/advance-article-abstract/doi/10.1093/bioinformatics/bty610/5050791?redirectedFrom=fulltext)
