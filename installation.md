# Installation

Το Nextflow απαιτεί Bash 3.2 και Java 11, τουλάχιστον.

Εγκατάσταση σε όποιο folder θέλουμε

```
wget -qO- https://get.nextflow.io | bash
chmod +x nextflow
sudo mv nextflow /usr/local/bin
sudo chmod 755 /usr/local/bin/nextflow
```

Update Nextflow

```
sudo nextflow self-update
```

Δοκιμή πρώτου script. Δημιουργούμε ένα αρχείο tutorial.nf σε έναν φάκελο εργασίας

```
#!/usr/bin/env nextflow
nextflow.enable.dsl=2

params.str = 'Hello world!'

process splitLetters {
  output:
    path 'chunk_*'

  """
  printf '${params.str}' | split -b 6 - chunk_
  """
}

process convertToUpper {
  input:
    file x
  output:
    stdout

  """
  cat $x | tr '[a-z]' '[A-Z]'
  """
}

workflow {
  splitLetters | flatten | convertToUpper | view { it.trim() }
}
```

Το τρέχουμε με την εντολή

```
nextflow run tutorial.nf
```

