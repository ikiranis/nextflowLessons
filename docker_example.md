# Docker example

### Χρήση κάποιου docker image, από το οποίο θα τρέξουμε κάποιο script ή πρόγραμμα

Πρώτα δημιουργούμε ένα αρχείο nextflow.config (στο ίδιο folder με το script)

```
docker.enabled = true
```

Μετά δημιουργούμε τo script σε αρχείο κατάληξης .nf

```
#! /usr/bin/env nextflow

nextflow.enable.dsl=2

process runWhatshap {
    container 'python'
   
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