# Chapitre 4 -- L abstraction multi-KMS et la signature GCP en detail

`provider.SignatureProvider` est une interface a deux methodes,
`SignDigest` et `GetPublicKey`, implementee par trois fournisseurs
interchangeables selon `config.yaml` : `GCPKMSSignatureProvider`,
`AWSKMSSignatureProvider`, et un `LocalKMSSignatureProvider` explicitement
marque "NE PAS UTILISER EN PRODUCTION" dans le README (fichiers de cles
PEM en clair, reserve au developpement). `NewSignatureProvider` selectionne
l implementation via un simple aiguillage sur `ProviderType`, gardant le
reste du service (chapitres 2 et 3) totalement independant du fournisseur
de cles reellement utilise.

Le fournisseur GCP illustre la complexite reelle de ce pont entre un KMS
generaliste et la cryptographie specifique a Ethereum. `SignDigest`
envoie le digest a signer via `AsymmetricSign` (avec un `DigestCrc32C`
precalcule que GCP renvoie verifie -- `VerifiedDigestCrc32C` --
garantissant l integrite du transport de la requete), puis verifie que le
CRC32C de la signature recue correspond egalement, avant de deleguer a
`convertToCompactRecoverableSignature`. Cette fonction realise trois
transformations successives : `convertToCompactSignature` parse la
signature DER renvoyee par GCP (format ASN.1 standard, R et S encodes
separement sur une longueur variable) et la reencode en format compact
64 octets (R et S concatenes, chacun aligne sur 32 octets par bourrage a
gauche) -- avec une etape de canonisation cruciale : si `S` depasse la
moitie de l ordre de la courbe secp256k1, la fonction le remplace par
`ordre - S`, evitant la malleabilite de signature (une meme transaction
aurait sinon deux signatures valides distinctes selon le signe de `S`
choisi). Une verification de non-malleabilite et une verification
cryptographique de la signature contre la cle publique
(`secp256k1.VerifySignature`) suivent, avant que
`calculateRecoveryID` ne determine lequel des candidats de bit de
recuperation (0 ou 1) permet effectivement de retrouver la cle publique
attendue a partir de la signature -- ce bit est ensuite ajoute comme 65e
octet, produisant exactement le format de signature recuperable
qu Ethereum attend (`r || s || v`).

Cette chaine de conversions -- DER vers compact, canonisation de S,
calcul du bit de recuperation -- est necessaire parce que GCP KMS, en tant
que service de signature generaliste, ne connait rien des conventions
specifiques a Ethereum ; `op-signer` porte seul la responsabilite de
combler cet ecart entre un KMS cloud generique et le format de signature
exact qu un noeud Ethereum ou OP Stack acceptera.
