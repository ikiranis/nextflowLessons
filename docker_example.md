# Docker example

### Χρήση κάποιου docker image, από το οποίο θα τρέξουμε κάποιο script ή πρόγραμμα

Πρώτα δημιουργούμε ένα αρχείο nextflow.config (στο ίδιο folder με το script)

```
docker.enabled = true
process.container = 'python'
//docker.runOptions = '-u $(id -u):$(id -g)'
docker.runOptions = "--volume ${projectDir}/data:/pharmcat/data"
```

Μετά δημιουργούμε τo script σε αρχείο κατάληξης .nf

```
#! /usr/bin/env nextflow
nextflow.enable.dsl=2

// Basic variables
dataDir = "/pharmcat/data"
inputFile1 = "A_15_19.hg19.GATK.snp"
inputFile2 = "A_15_19.hg19.final"
outputFile = "phased"

process runWhatshap {

    output:
        path '*.vcf'
        

    """
    #cd /pharmcat
    #echo hello > out.vcf
    whatshap phase -o ${outputFile}.vcf --no-reference $dataDir/${inputFile1}.vcf.gz $dataDir/${inputFile2}.bam
    """
}

process runPreprocessor {  
    input:
        path phased_file 

    output:
        path '*.vcf'

    """
    fullpath="\$(readlink -f ${phased_file})"
    cd /pharmcat
    ./PharmCAT_VCF_Preprocess.py --input_vcf "\${fullpath}"
    """
}

process runPharmcat {
    input:
     	path ready_vcf

    output:
        stdout

    """
    ls -la
    fullpath="\$(readlink -f ${ready_vcf})"
    cd /pharmcat
    ./pharmcat -vcf "\${fullpath}"
    """
}

workflow {
    runWhatshap()
    runPreprocessor(runWhatshap.out)
    runPharmcat(runPreprocessor.out)
    runPharmcat.out.view()
}
```

Το τρέχουμε με

```
nextflow run run.nf
```

ή (χωρίς το αρχείο .config)

```
nextflow run run.nf -with-docker [docker image]
```
