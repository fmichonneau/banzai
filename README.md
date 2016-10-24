#banzai!#

🏄

**banzai** is a shell script (bash) that links together the disparate programs needed to process the raw results from an Illumina sequencing run of PCR amplicons into a contingency table of the number of similar sequences found in each of a set of samples.

The script should run on Unix (Mac OSX) and Linux machines. It makes heavy usage of Unix command line utilities (such as find, grep, sed, awk, and more) and is written for the BSD versions of those programs as found on standard installations of Mac OSX. I tried to use POSIX-compliant commands wherever possible.

Banzai was designed for sequencing data that were generated like so:

```
Genomic DNA:    ----------------------------------------------------------------
target region:                   ~~~~~~~~~~~~~~~~~~~~~~~~
```

PCR:
```
Primers:                   ******                        ******
Secondary index:        +++                                    +++
full primer:            +++******                        ******+++
amplicon:               +++******~~~~~~~~~~~~~~~~~~~~~~~~******+++
```

Library Prep:
```
primary index:       :::                                          :::
adapter:           aa                                                aa
final fragment:    aa:::+++******~~~~~~~~~~~~~~~~~~~~~~~~******+++:::aa
```

SEQUENCING
```
Read 1:            aa:::+++******~~~~~~~~~~~~~~
Read 2:                                    ~~~~~~~~~~~~~~******+++:::aa
```

DEMULTIPLEXING (PRIMARY)
```
Read 1:                 +++******~~~~~~~~~~~~~~
Read 2:                                    ~~~~~~~~~~~~~~******+++
```

READ MERGING
```
merged reads:           +++******~~~~~~~~~~~~~~~~~~~~~~~~******+++
```

DEMULTIPLEXING (SECONDARY)
```
                           ******~~~~~~~~~~~~~~~~~~~~~~~~******
 ```
PRIMER REMOVAL
```
                                 ~~~~~~~~~~~~~~~~~~~~~~~~
```
(Layout inspired in part by [this](https://github.com/geraldinepascal/FROGS).)

## Basic implementation ##
Copy the file 'banzai_params.sh' into a new folder and set parameters as desired. Then run the banzai script, using your newly edited parameter file like so (Mac OSX):

```sh
bash /Users/user_name/path/to/the/file/banzai.sh   /User/user_name/path/to/param_file.sh
```

It's probably important to use `bash` rather than `sh` or `.` to invoke the script. Someday I'll figure out a better workaround, but for now this was the only way I could guarantee the log file was created in the way I wanted.


## Dependencies ##
Aside from the standard command line utilities (awk, sed, grep, etc) that are already included on Unix machines, this script relies on the following tools:

* **[PEAR](http://sco.h-its.org/exelixis/web/software/pear/)**: merging paired-end reads
* **[cutadapt](https://github.com/marcelm/cutadapt)**: primer removal (I might replace with awk)
* **[vsearch](https://github.com/torognes/vsearch)**: sequence quality filtering (requires version 1.4.0 or greater); OTU clustering
* **[swarm](https://github.com/torognes/swarm)**: OTU clustering
* **[seqtk](https://github.com/lh3/seqtk)**: reverse complementing entire fastq/a files
* **[python](https://www.python.org/)**: fast consolidation of duplicate sequences (installed by default on Macs)
* **[blast+](http://www.ncbi.nlm.nih.gov/books/NBK279690/)**: taxonomic assignment
* **[MEGAN](http://ab.inf.uni-tuebingen.de/software/megan5/)**: taxonomic assignment
* **[R](https://www.r-project.org/)**: ecological analyses. Requires the packages **vegan** and **gtools**

Follow the [Vagrant-VirtualBox instructions](doc/vagrant_install.md) to automatically install your own virtual machine that includes all of these dependencies.

### Recommended ###
* Compressing and decompressing files can be slow because standard, built-in utilities (gzip) do not run in parallel. Installing the parallel compression tool **[pigz](http://zlib.net/pigz/)** can yield substantial speedups. Banzai will check for pigz and use it if available.

* I recommend that before analyzing data, you check and report basic properties of the sequencing runs using **[fastqc](http://www.bioinformatics.babraham.ac.uk/projects/fastqc/)**. I have included a script to do this for all the fastq or fastq.gz files in any subdirectory of a directory (run_fastqc.sh).


## Sequencing Pool Metadata ##
You must provide a CSV spreadsheet that contains metadata about the samples. Banzai will read some of the parameters from it, like the primers and multiplex index sequences. You need to provide the file path to the spreadsheet, and the relevant column names. Some of the details seem tedious (like listing each file name), but they are inspired by the EMBL/EBIMetagenomics metadata requirements. That is, you're going to have to do this stuff at some point anyway.

This file should be encoded with UNIX newline characters (LF). Banzai will attempt to check for and fix files encoded with Windows newline characters (CRLF), but this is a place to look if you get mysterious errors. Early in the logfile you can check to be sure the correct number of tags and primer sequences were found.

No field should contain any spaces. That means row names, column names, and cells. Accommodating this would require an advanced degree in bash-quoting judo, which I do not have.

## Library Names ##
As of 2015-10-09, libraries no longer have to be named anything in particular (e.g. A, B, lib1, lib2),
BUT THEY CANNOT CONTAIN UNDERSCORES or spaces! (This will be moot once library index sequences are required)

## Organization of raw data ##
Your data (fastq files) can be compressed or not; but banzai currently only works with paired-end Illumina data. Thus, the bare minimum input is two fastq files corresponding to the first and second read. *Banzai will fail if there are files in your library folders that are not your raw data but have 'fastq' in the filename!* For example, if your library contains four files: "R1.fastq", "R1.fastq.gz", "R2.fastq", and "R2.fastq.gz". banzai will grab the first two (R1.fastq and R1.fastq.gz) and try to merge them, and (correctly) fail miserably. Note that while PEAR 0.9.7 merges compressed (\*.gz) files directly, PEAR 0.9.6 does not do so correctly. If given compressed files as input, banzai first decompresses them, which will add a little bit of time to the overall analysis.

## A note on removal of duplicate sequences##

###  (dereplicate_fasta.py) ###

* Input: a fasta file (e.g. 'infile.fasta')

* Output: a file with the same name as the input but with the added extension '.derep' (e.g. 'infile.fasta.derep')

This output file contains each unique DNA sequence from the fasta file, followed by the labels of the reads matching this sequence
Thus, if an input fasta file consisted of three reads with identical DNA sequences:

	>READ1
	AATAGCGCTACGT
	>READ2
	AATAGCGCTACGT
	>READ3
	AATAGCGCTACGT

The output file is as follows:

	AATAGCGCTACGT; READ1; READ2; READ3

Note that the original script also output a file of the sequences only (no names), but I removed this functionality on 20150417


## Known Issues/Bugs ##
* Currently awaiting catastrophic finding...

###Notes###
An alternate hack to have the pipeline print to terminal AND file, in case logging breaks:
sh script.sh  2>&1 | tee ~/Desktop/logfile.txt

* 2016-10-22 Began major reorganization. Created v0.1.0-beta for MBON in case anything causes a fire.
* 2015-10-19 expected error filtering implemented via vsearch. OTU clustering can be done with swarm or usearch.
* 2015-10-09 read length calculated from raw data. Library names are flexible.
* 2014-11-12 Noticed that the reverse tag removal step removed the tag label from the sequenceID line of fasta files if the tag sequence is RC-palindromic!
