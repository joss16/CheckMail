# CheckMail 2024-01-17
#!/bin/bash

# Kontrollera om lista.txt finns
if [ ! -f lista.txt ]; then
    echo "lista.txt hittades inte"
    exit 1
fi

# Skapa eller töm filen notvalid.txt
> notvalid.txt

# Läs varje rad i lista.txt
while IFS= read -r domain; do
    echo "Kontrollerar domän: $domain"
    isValid=true

    # Kontrollera SPF-post
    spfRecord=$(dig +short TXT "$domain" | grep "v=spf1")
    if [ -z "$spfRecord" ]; then
        echo "Ingen SPF-post hittad för $domain"
        isValid=false
    fi

    # Kontrollera DMARC-post
    dmarcRecord=$(dig +short TXT "_dmarc.$domain" | grep "v=DMARC1")
    if [ -z "$dmarcRecord" ]; then
        echo "Ingen DMARC-post hittad för $domain"
        isValid=false
    fi

    # Kontrollera DKIM-post (förutsätter att du känner till selektorn, exempelvis 'default')
    dkimRecord=$(dig +short TXT "default._domainkey.$domain")
    if [ -z "$dkimRecord" ]; then
        echo "Ingen DKIM-post hittad för $domain"
        isValid=false
    fi

    # Lägg till domänen i notvalid.txt om den inte uppfyller kraven
    if [ "$isValid" = false ]; then
        echo "$domain" >> notvalid.txt
    fi

    echo "-----------------------------------"
done < lista.txt

