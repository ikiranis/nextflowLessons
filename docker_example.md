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
params.dataDir = "/pharmcat/data"
params.inputFile1 = "A_15_19.hg19.GATK.snp"
params.inputFile2 = "A_15_19.hg19.final"
params.outputFile = "phased"
params.finalReport = "report.html"

log.info """\
         N F  P I P E L I N E    
         ====================
         
         data directory   : ${params.dataDir}
         input file 1  : ${params.inputFile1}
         input file 1  : ${params.inputFile2}
         phased output file : ${params.outputFile}
         final HTML report : ${params.finalReport}

         """
         .stripIndent()

// Run whatshap script. Generate phased.vcf file
process runWhatshap {
    debug true

    output:
        path '*.vcf'

    """
    echo "Processing files..."
    whatshap phase -o ${params.outputFile}.vcf --no-reference $params.dataDir/${params.inputFile1}.vcf.gz $params.dataDir/${params.inputFile2}.bam
    """
}

// Run PharmCAT_VCF_Preprocess script. Generate pharmcat_ready_vcf... file
process runPreprocessor { 
    debug true

    input:
        path phased_file 

    output:
        path '*.vcf'   

    """
    echo "Processing file ${phased_file}..."
    fullpath="\$(readlink -f ${phased_file})"
    currentPath="\$(pwd)"
    cd /pharmcat
    ./PharmCAT_VCF_Preprocess.py --input_vcf "\${fullpath}" --output_folder "\${currentPath}"
    """
}

// Run pharmcat script. Generate ...report.html file
process runPharmcat {
    debug true

    input:
     	path ready_vcf

    output:
        path '*.html'

    """
    echo "Processing file ${ready_vcf}..."
    fullpath="\$(readlink -f ${ready_vcf})"
    currentPath="\$(pwd)"
    cd /pharmcat
    ./pharmcat -vcf "\${fullpath}" -o "\${currentPath}"
    """
}

// Copy final HTML file to data directory
process runFinal {
    debug true

    input:
     	path html_file

    """
    cp ${html_file} ${params.dataDir}/${params.finalReport}
    echo "${params.finalReport} generated on /data folder"
    """
}

workflow {
    runWhatshap()
    runPreprocessor(runWhatshap.out)
    runPharmcat(runPreprocessor.out)
    runFinal(runPharmcat.out)
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
