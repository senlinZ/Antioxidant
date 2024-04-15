# Antioxidant

## 1.Assembly


## 2.Improve quality of MAGs using third-generation sequencing data (minimap2+SSPACE-LongRead)
```
#minimap2
minimap2 raw/10572_0014_idba_bin.58.fa hifi-3cells.fq -t 8 > 10572_0014_idba_bin.58.paf
Perl /mnt/sdb/zengl/taoye/bin/mini-paf-fqget.pl hifi-3cells.fq 10572_0014_idba_bin.58.paf 10572_0014_idba_bin.58.hifi.fq

#SSPACE-LongRead
Perl /mnt/sdb/zengl/taoye/bin/sspace_longread/SSPACE-LongRead.pl -c raw/10572_0014_idba_bin.58.fa -p 10572_0014_idba_bin.58.hifi.fq -t 8 -b 10572_0014_idba_bin.58.sspaceout
```

## 3.CheckM
```
checkm lineage_wf ./01.raw checkmout/ -x fa -t 88 --pplacer_threads 4 --tab_table -f checkm.summary
```

## 4.dRep
```
dRep dereplicate dRepOut -p 56 -g 02.filter/*.fa --ignoreGenomeQuality -pa 0.95 -sa 0.99 -cm larger
```
## 5.GTDB
```
gtdbtk classify_wf --extension fa --genome_dir 02.filter/ --cpus 88 --out_dir gtdbtkout --prefix cowmag
```
## 6.antismash
```
#/bin/bash

for mag in `ls CowSGB/Fix-33236mag/ |sed -n '1,14096p'` ; do SAMPLE=${mag%.fa}; antismash --genefinding-tool prodigal --asf  --fullhmmer  --cc-mibig  --smcog-trees --cb-general  --cb-knownclusters --cb-subclusters --pfam2go --rre  CowSGB/Fix-33236mag/$mag  --output-dir CowMAG_antismash_nomal_rest_3w/CowSGB-$SAMPLE; done
```
