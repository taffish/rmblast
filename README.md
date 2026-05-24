# rmblast

TAFFISH app for [RMBlast](https://www.repeatmasker.org/rmblast/), the
RepeatMasker-compatible BLAST+ distribution that adds `rmblastn` and
RepeatMasker/RepeatModeler-specific nucleotide search features to the BLAST+
toolkit.

## Package Identity

- name: `rmblast`
- command: `taf-rmblast`
- version: `2.17.1-r2`
- kind: `tool`
- image: `ghcr.io/taffish/rmblast:2.17.1-r2`
- upstream: RMBlast `2.17.1`, based on NCBI BLAST+ `2.17.0`
- runtime version: `rmblastn: 2.17.1+`
- native platform: `linux/amd64`
- default command: `rmblastn`

The RMBlast download page still has a stale heading that says "Latest Release:
2.14.1", but the linked Linux tarball, source patch, and release notes are for
RMBlast `2.17.1`. This app is bound to the `2.17.1` tarball and verifies its
SHA256 during image build.

## Install

```sh
taf install rmblast
```

## Usage

TAFFISH wrapper help:

```sh
taf-rmblast --help
taf-rmblast --version
taf-rmblast --compile
```

Upstream `rmblastn` help and version:

```sh
taf-rmblast -- -help
taf-rmblast -- -version
taf-rmblast rmblastn -version
```

Build a nucleotide BLAST database and search it with `rmblastn`:

```sh
taf-rmblast makeblastdb -in repeats.fa -dbtype nucl -out repeats
taf-rmblast -query genome.fa -db repeats -task megablast -dust no \
  -outfmt "6 qseqid sseqid pident length evalue bitscore" -out hits.tsv
```

Run bundled companion commands through command mode:

```sh
taf-rmblast blastn -query query.fa -db db -outfmt 6
taf-rmblast blastdbcmd -db db -entry all
taf-rmblast dustmasker -in sequences.fa -out masked.asn1
```

## Wrapper Semantics

`taf-rmblast` defaults to `rmblastn`. RMBlast and BLAST+ use single-dash
options such as `-query`, `-db`, `-help`, and `-version`, so normal option-led
calls are passed directly to `rmblastn`.

The app keeps TAFFISH command mode enabled. If the first argument is another
executable name, TAFFISH runs that command inside the same RMBlast container:

```sh
taf-rmblast makeblastdb -in repeats.fa -dbtype nucl -out repeats
taf-rmblast blastdbcmd -db repeats -entry all
```

## Runtime Contents

The image installs the official `rmblast-2.17.1+-x64-linux.tar.gz` binary
package from repeatmasker.org. The tarball checksum used by the Dockerfile is:

```text
9ddf0ffdc01f3dd7bf62bbcb01516757b1e732e54674e81d93a7c143157aa527
```

Packaged user-facing commands include:

- `rmblastn`
- `blastn`, `blastp`, `blastx`
- `tblastn`, `tblastx`
- `psiblast`, `rpsblast`, `rpstblastn`, `deltablast`
- `makeblastdb`, `blastdbcmd`, `blast_formatter`
- `dustmasker`, `segmasker`, `windowmasker`
- `makeprofiledb`, `makembindex`, `convert2blastmask`
- `blastdb_aliastool`, `blastdbcheck`
- `legacy_blast.pl`, `update_blastdb.pl`, `get_species_taxids.sh`

Runtime libraries and tools include Perl, `perl-doc`, curl, gzip, tar,
`libsqlite3`, `libgomp`, `libbz2`, `libzstd`, zlib, libstdc++ and glibc runtime
libraries.

`BLAST_USAGE_REPORT=false` is set in the image. Upstream RMBlast notes that
recent BLAST+ tools can report usage statistics, and RMBlast redirects those
statistics to repeatmasker.org. TAFFISH runs default to no usage reporting for
offline, reproducible command execution. Users can override this environment
variable if they intentionally want upstream-style reporting.

## Platform

This app is native `linux/amd64` only because the official RMBlast Linux
binary is x86-64. `src/main.taf` asks Docker and Podman to run the image with
`--platform linux/amd64`. On arm64 hosts, Docker/Podman may run it through
amd64 emulation; that is not native arm64 support. Apptainer behavior depends
on the host's ability to run amd64 containers.

## Boundaries

This app packages RMBlast, not RepeatMasker, RepeatModeler, Dfam, or RepBase.
It is intended to be useful both directly and as an eventual building block for
RepeatMasker/RepeatModeler images.

`update_blastdb.pl` and `get_species_taxids.sh` are included because they ship
with the upstream binary package. Online BLAST database downloads, Entrez
Direct setup, and remote searches are not part of the smoke-tested offline
surface. Users who need those workflows may need network access and additional
NCBI tooling.

## License Boundary

The TAFFISH app packaging files are licensed under Apache-2.0. The packaged upstream RMBlast software is covered by: OSL-2.1 with bundled NCBI public-domain BLAST+ components. Bundled third-party components, datasets, models, and external resources keep their own license terms.

## License And Citation

The RMBlast download page states that RMBlast is licensed under Open Source
License v2.1. The bundled BLAST+ components include an NCBI public domain
notice in `LICENSE.rmblast`.

For BLAST+ methods, cite:

- Camacho et al. 2009. BLAST+: architecture and applications. BMC
  Bioinformatics. DOI: `10.1186/1471-2105-10-421`; PMID: `20003500`.

Upstream RMBlast authors/credits are listed in the distributed
`README.rmblast`: Arian Smit, Robert Hubley, Jeb Rosen, and NCBI contributors.

## Smoke Coverage

Smoke tests cover:

- `rmblastn` and bundled `blastn` runtime version strings.
- RMBlast-specific help surface: `-complexity_adjust`, `-mask_level`, and
  `-mask_level_algo`.
- Dynamic library closure for `rmblastn`, including `libsqlite3` and `libgomp`.
- Core BLAST+ help commands such as `makeblastdb`, `blastdbcmd`,
  `blast_formatter`, `dustmasker`, and `segmasker`.
- A tiny offline `makeblastdb` + `rmblastn` nucleotide search.
- A tiny offline `makeblastdb` + bundled `blastn` nucleotide search.
- Perl helper help for `legacy_blast.pl` and `update_blastdb.pl`.
