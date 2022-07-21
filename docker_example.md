# Docker example

### Χρήση κάποιου docker image, από το οποίο θα τρέξουμε κάποιο script ή πρόγραμμα

Πρώτα δημιουργούμε ένα αρχείο nextflow.config (στο ίδιο folder με το script)

```
docker.enabled = true
process.container = 'python'
```

Μετά δημιουργούμε τo script σε αρχείο κατάληξης .nf

```
#! /usr/bin/env nextflow

nextflow.enable.dsl=2

process runWhatshap {   
    output:
        stdout

    '''
    whatshap --version
    '''
}

workflow {
    runWhatshap()
    runWhatshap.out.view()
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
