# Docker example

### Χρήση κάποιου docker image, από το οποίο θα τρέξουμε κάποιο script ή πρόγραμμα

Πρώτα δημιουργούμε ένα αρχείο nextflow.config (στο ίδιο folder με το script)

```
docker.enabled = true
process.container = 'python'
//process.container = 'gwissandbox.azurecr.io/pgx-to-report:latest'
//docker.runOptions = '-u $(id -u):$(id -g)'
docker.runOptions = "--volume /home:/home"
```

Μετά δημιουργούμε τo script σε αρχείο κατάληξης .nf

```
#! /usr/bin/env nextflow
nextflow.enable.dsl=2

// Basic params
params.panel = "NA" //panel name - not in use
params.genome_id = "hg38" //genome version
params.id = "87" // id 
params.template_id = "1" // template id (folder name)
params.run_loc = 'azure' // if running on azure, specify 'azure' in the command line
params.data_dir='patient-data' // data folder name ( possible values : patient-data and control-data)

if (params.run_loc == 'azure') {
    params.base_dir= "az://storegene-data/" // location of the SNP picker folder
}
else {
    params.base_dir="/home/user/nextflow/" // location of the SNP picker folder
}

params.outputFile = "phased" // filename of whatshap output file
params.finalReport = "report.html" // filename of the final report


// Basic variables
String patient_path_in = params.base_dir + params.data_dir + "/" + params.id + "/input/" + params.genome_id + "/" // location of patient data
String patient_path_out = params.base_dir + params.data_dir + "/" + params.id + "/output/" + params.genome_id + "/" // location of patient data
String patient_input_bam = patient_path_in + "*.final.bam" // location of the input folder
String patient_input_bam_bai = patient_path_in + "*.final.bam.bai" 
String patient_input_vcf = patient_path_in + "*.snp.vcf"  // location of vcf file
String outputFile = params.outputFile + ".vcf"
String finalReport = patient_path_out + params.finalReport

// Channels
BamFile = Channel.fromPath( patient_input_bam ) 
BamBaiFile = Channel.fromPath( patient_input_bam_bai ) 
VcfFile = Channel.fromPath( patient_input_vcf ) 
FinalReport = Channel.fromPath( finalReport ) 
PathOut = Channel.fromPath( patient_path_out ) 

log.info """\
         N F  P I P E L I N E    
         ====================

         params panel : ${params.panel}
         genome id : ${params.genome_id}
         id : ${params.id}
         template id : ${params.template_id}
         run location : ${params.run_loc}
         data dir : ${params.data_dir}
         base dir : ${params.base_dir}
         input bam file  : ${patient_input_bam}
         input bam bai file  : ${patient_input_bam_bai}
         input vcf file  : ${patient_input_vcf}
         phased output file : ${params.outputFile}
         final HTML report : ${params.finalReport}
         patient path in : ${patient_path_in}
         patient path out : ${patient_path_out}

         """
         .stripIndent()

// Run whatshap script. Generate phased.vcf file
process runWhatshap {
    debug true

    input: 
        file BamFile
        file BamBaiFile
        file VcfFile

    output:
        path '*.vcf'

    """
    echo "Processing files..."
    whatshap phase -o ${outputFile} --no-reference ${VcfFile} ${BamFile}
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
    publishDir "$patient_path_out/", pattern: '*.html', mode: 'copy'

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
        file FinalReport
        path PathOut

    """
    #mkdir -p ${PathOut}
    ls ${PathOut}
    #cp ${html_file} ${PathOut}${FinalReport}
    #echo "${FinalReport} generated on ${PathOut} folder"
    """
}

workflow {
    runWhatshap(BamFile, BamBaiFile, VcfFile)
    runPreprocessor(runWhatshap.out)
    runPharmcat(runPreprocessor.out)
    runFinal(runPharmcat.out, FinalReport, PathOut)
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
