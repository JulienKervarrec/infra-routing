# Chapitre 1 -- Presentation de infra-routing et de op-signer

Ce depot est un monorepo d infrastructure : le README le decrit
explicitement comme une extension du monorepo Optimism, contenant les
services operationnels qui soutiennent l ecosysteme -- pas des contrats
ni des bibliotheques client, mais des services deployes en production
autour des noeuds OP Stack. Onze composants cohabitent au niveau racine
(`cci-stats`, `op-acceptor`, `op-conductor-mon`, `op-conductor-ops`,
`op-signer`, `op-txproxy`, `op-ufm`, `ops`, `peer-mgmt-service`, `proxyd`,
`replica-healthcheck`), chacun avec son propre `go.mod` et sa propre
chaine de build -- une structure de monorepo multi-module plutot qu un
module Go unique. Le README n en met formellement en avant que quatre
comme "Components" principaux : `op-acceptor` (testeur d acceptation
reseau pour les devnets), `op-conductor-mon` (supervision de plusieurs
instances `op-conductor`), `op-signer` (passerelle de signature legere
appuyee sur un KMS), et `op-ufm` (monitoring de bout en bout par
transactions de sonde a intervalles reguliers).

Ce parcours se concentre sur `op-signer`, le composant le plus dense en
decisions d architecture de securite : c est un service qui signe des
transactions et des payloads de bloc pour le compte d autres services OP
Stack, sans jamais exposer la cle privee elle-meme -- la cle reste geree
par un fournisseur de gestion de cles (KMS) externe (GCP KMS, AWS KMS, ou
des fichiers de cles locales pour le developpement). Le README precise
explicitement que le nom du module est `@eth-optimism/signer`, signe de
son origine partagee avec le monorepo Optimism amont plutot qu un
composant invente par Base.

`op-signer` est structure en trois paquets : `service/` (le serveur RPC et
la logique d autorisation), `provider/` (l abstraction sur les differents
fournisseurs de KMS), et `cmd/` (le point d entree CLI qui les assemble).
