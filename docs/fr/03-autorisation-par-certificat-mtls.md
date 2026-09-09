# Chapitre 3 -- Autorisation : identifier le client par son certificat mTLS

`op-signer` n identifie jamais un client appelant par une cle API ou un
jeton -- il s appuie entierement sur l authentification mutuelle TLS
(mTLS). `NewAuthMiddleware` (dans `service/auth.go`) est un middleware
HTTP qui recupere les informations du certificat client deja verifie en
amont par le serveur HTTP (`optls.PeerTLSInfoFromContext`), exige qu un
certificat feuille ait ete fourni (sinon HTTP 401), exige que ce
certificat contienne au moins une extension SAN (Subject Alternative
Name) de type DNS (sinon 401 egalement), puis utilise le premier nom DNS
de cette extension comme identite du client (`clientInfo.ClientName`),
propagee dans le contexte de la requete pour le reste du traitement RPC.
Le nom du client n est donc pas declaratif (un en-tete que le client
pourrait falsifier) mais cryptographiquement garanti par la chaine de
certification TLS elle-meme.

Cote configuration, `ProviderConfig.GetAuthConfigForClient` (dans
`provider/config.go`) recherche, dans la liste `auth` du fichier
`config.yaml`, l entree dont le champ `name` correspond exactement au nom
DNS du client authentifie -- et, si une adresse `fromAddress` est
precisee dans la requete, verifie en plus qu elle correspond a celle
configuree pour ce client. Chaque entree `AuthConfig` associe un client a
un `KeyName` (le localisateur de cle KMS specifique a ce client), un
`chainID` optionnel, et deux garde-fous optionnels appliques au moment de
la signature de transaction (chapitre 2) : `ToAddresses` (liste blanche
de destinataires autorises, comparee insensible a la casse via
`containsNormalized`) et `MaxValue` (plafond de valeur transferable,
compare en tant que grand entier). Un client authentifie par TLS mais
absent de la configuration, ou dont la transaction viole l une de ces
restrictions, se voit refuser la signature -- le fichier de configuration
YAML fonctionne ainsi comme une politique de moindre privilege explicite,
un client ne pouvant jamais signer au-dela de ce que son entree
`AuthConfig` autorise.
