taf-rmblast 2.17.1-r2

RMBlast is a RepeatMasker-compatible BLAST+ distribution. This app defaults to
rmblastn and also exposes the companion BLAST+ commands bundled in the official
RMBlast package.

Usage:
  taf-rmblast [-h | --help]
  taf-rmblast [-v | --version]
  taf-rmblast [RMBLASTN_OPTIONS...]
  taf-rmblast -- [RMBLASTN_ARGS...]
  taf-rmblast COMMAND [ARGS...]
  taf-rmblast --compile [ARGS...]

Wrapper options:
  -h, --help       Show this TAFFISH app help
  -v, --version    Show TAFFISH package version
  --compile        Print generated shell code instead of running it
  --               Stop wrapper option parsing and pass following args to
                   rmblastn

Default upstream command:
  taf-rmblast -- -version
  taf-rmblast -- -help
  taf-rmblast -query query.fa -db repeats -task megablast -dust no

Command mode:
  taf-rmblast rmblastn -version
  taf-rmblast makeblastdb -in repeats.fa -dbtype nucl -out repeats
  taf-rmblast blastn -query query.fa -db db -outfmt 6
  taf-rmblast blastdbcmd -db db -entry all
  taf-rmblast dustmasker -in sequences.fa -out masked.asn1

Important wrapper detail:
  rmblastn uses single-dash BLAST options such as -help, -version, -query and
  -db. Option-leading calls are sent to rmblastn. To run another executable in
  the same container, put that executable first, for example:
    taf-rmblast makeblastdb -in repeats.fa -dbtype nucl -out repeats

Common rmblastn workflow:
  taf-rmblast makeblastdb -in repeats.fa -dbtype nucl -out repeats
  taf-rmblast -query genome.fa -db repeats -task megablast -dust no \
    -outfmt "6 qseqid sseqid pident length evalue bitscore" -out hits.tsv

RepeatMasker/RepeatModeler options:
  -complexity_adjust       Enable RMBlast complexity-adjusted scoring
  -mask_level INT          Filter overlapping HSPs by mask level
  -mask_level_algo STRING  cross_subject or per_subject
  -matrix FILE             Use a custom nucleotide scoring matrix
  -matrix_lambda FLOAT     Custom matrix lambda value
  -matrix_k FLOAT          Custom matrix K value
  -matrix_alpha FLOAT      Custom matrix length-correction slope
  -matrix_beta FLOAT       Custom matrix length-correction intercept

Packaged commands:
  rmblastn, blastn, blastp, blastx, tblastn, tblastx, psiblast, rpsblast,
  rpstblastn, deltablast, makeblastdb, blastdbcmd, blast_formatter,
  dustmasker, segmasker, windowmasker, makeprofiledb, makembindex,
  convert2blastmask, blastdb_aliastool, blastdbcheck, legacy_blast.pl,
  update_blastdb.pl, get_species_taxids.sh.

Runtime notes:
  BLAST_USAGE_REPORT is set to false by default so local TAFFISH runs do not
  report usage statistics. Set BLAST_USAGE_REPORT=true explicitly if you want
  upstream-style reporting.

Platform:
  This app uses the official RMBlast x64 Linux binary and is native
  linux/amd64 only. Docker and Podman runs request --platform linux/amd64.
  On arm64 hosts this means amd64 emulation, not native arm64 execution.

License:
  TAFFISH app packaging: Apache-2.0.
  Upstream software: OSL-2.1 with bundled NCBI public-domain BLAST+ components.
  Bundled components, data, models, and external resources keep their
  own license terms.

Boundaries:
  This app provides the RMBlast command suite, not RepeatMasker itself and not
  Dfam or RepBase repeat libraries. update_blastdb.pl and get_species_taxids.sh
  are included as upstream helper scripts, but online database downloads and
  Entrez Direct workflows are not part of smoke tests.
