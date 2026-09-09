# Parcours francais : infra-routing / op-signer (passerelle de signature KMS de Base)

Lecture pedagogique du composant op-signer du monorepo base/infra-routing : la passerelle de signature qui ponte un KMS cloud (GCP, AWS) et les conventions de signature Ethereum/OP Stack, avec son modele d autorisation par certificat client mTLS.

Sommaire :

Chapitre 1 Presentation de infra-routing et de op-signer. Chapitre 2 Deux espaces de noms RPC, eth et opsigner. Chapitre 3 Autorisation, identifier le client par son certificat mTLS. Chapitre 4 L abstraction multi-KMS et la signature GCP en detail. Chapitre 5 Limites et perimetre de ce parcours.

Ce parcours est une lecture pedagogique du code source et de la documentation du depot, sans installation ni execution du projet.
