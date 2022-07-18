# Channels

https://www.nextflow.io/docs/latest/channel.html

Οι processes επικοινωνούν μεταξύ τους, μέσω των channels. Υπάρχουν 2 είδη channels. Τα Queue Channels και τα Value Channels.

Το Queue Channel, ενώνει 2 processes. Μπορεί να χρησιμοποιηθεί ΜΟΝΟ μία φορά σαν output και μία σαν input. Δηλαδή αν παραχθεί από μία process, μετά μπορεί να χρησιμοποιηθεί μόνο από μία άλλη process σαν input.

Το Value Channel από την άλλη, μπορεί να πάρει μία τιμή και να χρησιμοποιηθεί όσες φορές θέλουμε, από όσες processes θέλουμε.

Ένα Channel μπορεί να δημιουργηθεί από διάφορες διαθέσιμες μεθόδους, ανάλογα με τον τύπο των δεδομένων.

Υπάρχουν επίσης κάποια events, που όταν συμβούν μπορούμε να τρέξουμε κάποιον κώδικα.

