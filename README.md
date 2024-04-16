# Antioxidant

## 1.Assembly
```
#Third-generation sequencing assembly 
input_reads_fastq="your_hifi_reads.fastq.gz"
output_dir="hifiasm_meta_output.log"
hifiasm_meta -t32 -oasm "${input_reads_fastq}" 2>"${output_dir}"
```


## 2.Improve quality of MAGs

```
#Improve the quality of MAGs using third-generation sequencing data (minimap2+SSPACE-LongRead)
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

## 6.Antismash

```
#/bin/bash

for mag in `ls CowSGB/Fix-33236mag/ |sed -n '1,14096p'` ; do SAMPLE=${mag%.fa}; antismash --genefinding-tool prodigal --asf  --fullhmmer  --cc-mibig  --smcog-trees --cb-general  --cb-knownclusters --cb-subclusters --pfam2go --rre  CowSGB/Fix-33236mag/$mag  --output-dir CowMAG_antismash_nomal_rest_3w/CowSGB-$SAMPLE; done
```

## 7.BIG-SCAPE

```
bash ~/application/BIG-SPACE/bin/run_bigscape GBKs/ new_antismash6_0.3_0.9_newmix --mibig -c 30 --include_singletons  --clan_cutoff  0.3  0.9  --mix
```

## 8.BIG-SLICE

```
target_folder="bigslice'

for file in "$target_folder"/*; do
  folder name=S(basename "sfile" |cut -d''-f1)
  mkdir -p "Sfolder_name"
  mv "Sfile" "Sfolder_name/"
done

bigslice  bigslice
```

## 9.Antioxidant identification models training using chemprop
```
#Use NRF2 data as an example

#Train
python train.py --data_path data/NRF2_data_for_train.csv \
    --dataset_type classification \
    --save_dir NRF2_test \
    --config_path NRF2_classification_ensemble_config --split_type scaffold_balanced \
    --ensemble_size 2 \
    --num_workers 20  \
    --no_features_scaling  \
    --aggregation sum \
    --aggregation_norm 50 \
    --split_sizes 0.8 0.1 0.1 \
    --split_type scaffold_balanced \
    --quiet \
    --features_generator morgan morgan_count rdkit_2d_normalized  --gpu 1 \
    --num_folds 10  \
    --loss_function binary_cross_entropy \
    --metric prc-auc \
    --extra_metrics f1 mcc auc

#Prediction
python predict.py \
    --test_path data/NRF2_data.csv \
    --checkpoint_dir trained_model/ET_classification_ensemble_loss_binary_cross_entropy_prc-auc1/ \
    --features_generator morgan morgan_count rdkit_2d_normalized   \
    --no_features_scaling    --gpu 0  \
    --preds_path predicted/NRF2_prediction.csv

#Interpret
python interpret.py \
    --data_path "data/Lipid_data_to_interpret2.csv" \
    --checkpoint_dir "trained_model/Lipid_classification_ensemble_loss_binary_cross_entropy_prc-auc1/" \
    --property_id 1  \
    --no_features_scaling \
    --num_workers 40\
    --features_generator morgan morgan_count rdkit_2d_normalized --gpu 1

```
