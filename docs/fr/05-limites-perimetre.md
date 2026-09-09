# Chapitre 5 -- Limites et perimetre de ce parcours

Ce parcours couvre la presentation generale du monorepo infra-routing et
le positionnement de op-signer parmi ses composants, les deux espaces de
noms RPC exposes (`eth` pour la signature de transactions,
`opsigner` pour la signature de payloads de bloc OP Stack),
l architecture d autorisation fondee sur l identification mTLS du client
et sa configuration YAML de moindre privilege, et le detail complet de la
chaine de conversion de signature du fournisseur GCP KMS, depuis le format
DER jusqu au format recuperable Ethereum.

Sont volontairement laisses hors champ : les dix autres composants du
monorepo (`op-acceptor`, `op-conductor-mon`, `op-conductor-ops`,
`op-txproxy`, `op-ufm`, `ops`, `peer-mgmt-service`, `proxyd`,
`replica-healthcheck`, `cci-stats`) au-dela de leur mention au chapitre 1
-- chacun meriterait a lui seul un parcours dedie, en particulier `proxyd`
(le proxy RPC largement utilise par l ecosysteme Optimism) et
`op-acceptor` (le testeur d acceptation de devnets) ; le detail complet
des fournisseurs `AWSKMSSignatureProvider` et `LocalKMSSignatureProvider`
au-dela de leur mention au chapitre 4 (qui suivent des logiques de
conversion de signature similaires a celle documentee pour GCP, avec des
specificites propres a chaque fournisseur) ; et l outillage de deploiement,
de tests et de CI (`.circleci/`, `Dockerfile`, `.goreleaser.yaml`) present
dans chaque sous-dossier.

L objectif reste le meme que pour les parcours precedents de cette
bibliotheque : comprendre precisement comment ce service ponte un KMS
cloud generique et les conventions cryptographiques specifiques a
Ethereum et OP Stack, sans pretendre couvrir l integralite d un monorepo
d infrastructure aussi vaste.
