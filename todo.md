# TODOs

#### General
- TODO: check for input files
- TODO: check for filename with .gz extension
- TODO: decompress if .gz
- TODO: add checks for installation of R packages (replace crap in  analyses_prelim.R)
- TODO: Put sample ID start ('ID1_') to variable
- TODO: fix subset to not decompress and recompress entire files

- note: would be good to use only adapter/primary index SEQUENCES (not names); but I have now encountered Illumina data where this is neither in the read headers OR the filenames. Bummer.

reading/writing wide format OTU/duplicate csv file takes an unreasonable amount of time (hours!).
- TODO: incorporate/require data.table (dev version with fwrite)

- TODO: make function for file checking/reporting/timestamping

```sh
timestamp () {
  echo $(date +%Y-%m-%d\ %H:%M) "${0}"
}

filetest () {
  if [[ -s "${1}" ]]; then
    timestamp "The following file is empty or absent:"
    echo "${1}"
    echo "This is a critical file, so the script will exit."
    exit
  fi
}
```

#### Annotation/blast
TODO clean up blast results parsing.
TODO add automatic output of blast methods.
TODO: consider concatenating all tabular blast hit files into single file

#### Fragment size calculation
TODO: It appears that the Minimum and Maximum assembly lengths, and the Minimum Overlap lengths being calculated in the banzai.sh script are causing the problem.

These parameters are calculated from the fragment length and the Read length of the sequence in the fastq file, which is 251.
Banzai is calculating:
- Minimum Overlap=176;
- Minimum assembly length=100;
- Maximum assembly length=200.
- I'm using the 150 fragment length that you suggested.

PEAR Minimum Overlap default is 10; Minimum assembly length default is 50; Maximum assembly length default is 0 which disables the restriction.

@kyamahara: Minimum assembly length = length_fragement - 50;  Maximum assembly length = length_fragment + 50;

```sh
OVERLAP_EXPECTED=$(($LENGTH_FRAG - (2 * ($LENGTH_FRAG - $LENGTH_READ) ) ))
MINOVERLAP=$(( $OVERLAP_EXPECTED / 2 ))
Where LENGTH_READ = 251
```

TODO: add sequence table? (this would be a huge file)
possible columns:
original seqid (from sequencer)
modified seqid (redundancies/whitespace removed)
sampleid (lib/tag combo?)
duplicate (duplicate name from dups.fasta)
otu (otu name from otus.fasta)
 - rows: libraries, tags (including rows for whole libraries)
 - column: numbers for number of sequences: successfully merged, filtered, forward index, both indexes (parallelizable using grep during awk demultiplexing?), primer 1, primer 2, singletons, (dups? otus?)

TODO: compress nosingle.txt and 7_no_primers.fasta.derep

TODO: add vsearch clustering

TODO: add library-specific size variable

TODO: An attempt to cause the script to exit if any of the commands returns a non-zero status (i.e. FAILS).

TODO: LIB_TAG_MOD originally contained sort | uniq; this is unnecessary I think

TODO: add `trap "killall background" EXIT` or `trap 'kill $(jobs -p)' EXIT` to kill background processes on exit

TODO: Add decontamination script

TODO: Add normalization

TODO: streamline config file

TODO: Find primer index as a function of 6bp preceeding or following primer seq  `grep -E -o '^.{6}'"${primer_F}"''`

TODO: add --debug / --verbose flag to generate files.

TODO: Potential files to be removed as part of cleanup at the end of script:
- homopolymer_line_numbers.txt
- 2_filtered.fasta
- 1_merged.assembled.fastq
- 1_merged.assembled_A.fastq
- 1_merged.assembled_B.fastq

TODO: remove whitespace from sequence labels? `sed 's/ /_/'`

TODO: Copy sequences to fasta files into separate directories based on tag sequence on left side of read
TODO: test for speed against removing the tag while finding it: wrap first tag regex in gsub(/pattern/,""):  awk 'gsub(/^.{0,9}'"$TAG_SEQ"'/,""){if . . .
TODO: MOVE THE VARIABLE ASSIGNMENT TO TOP; MOVE MKDIR TO TOP OF CONCAT IF LOOP
```sh
echo $(date +%H:%M) "Concatenating fasta files..."
CONCAT_DIR="$ANALYSIS_DIR"/all_lib
mkdir "${CONCAT_DIR}"
CONCAT_FILE="${CONCAT_DIR}"/1_demult_concat.fasta
```
#### PRIMER REMOVAL
TODO:: Parallelize cutadapt using gnu parallel: (? Actually, on a decent sized (20GB) MiSeq run, this only took 7 minutes; probably not worth worrying about optimizing for now). [example here](https://github.com/marcelm/cutadapt/issues/157)