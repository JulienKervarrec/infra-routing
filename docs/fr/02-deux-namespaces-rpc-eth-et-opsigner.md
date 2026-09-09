# Chapitre 2 -- Deux espaces de noms RPC : eth et opsigner

`SignerService` (dans `service/service.go`) expose deux services RPC
distincts sous deux espaces de noms differents, enregistres via
`RegisterAPIs` : `eth` (porte par `EthService`) et `opsigner` (porte par
`OpsignerService`) -- tous deux partageant en realite le meme fournisseur
de signature et la meme configuration sous-jacente, mais exposant des
methodes semantiquement distinctes.

`EthService.SignTransaction` implemente la methode `eth_signTransaction`
: elle recoit des `signer.TransactionArgs` (le type standard op-service
pour une transaction Ethereum non signee), les valide (`args.Check()`),
les convertit en donnees de transaction typees, calcule le digest de
signature via `types.LatestSignerForChainID(tx.ChainId())`, delegue la
signature du digest au fournisseur (`s.provider.SignDigest`), reconstruit
la transaction signee (`tx.WithSignature`), et effectue une verification
de coherence critique : elle recalcule l adresse `From` a partir de la
signature produite (`txSigner.Sender(signed)`) et la compare a l adresse
attendue, rejetant la transaction si le fournisseur a signe avec une cle
inattendue -- une garde-fou contre une mauvaise configuration silencieuse
du KMS.

`OpsignerService.SignBlockPayload` et `SignBlockPayloadV2` implementent
la signature de payloads de bloc pour le protocole OP Stack (utilisees par
les sequenceurs et conducteurs pour attester des blocs proposes) : les
deux methodes partagent une implementation commune,
`signBlockPayload`, qui calcule un hash de signature specifique au type
de message (`msg.ToSigningHash()`) plutot qu un hash de transaction
Ethereum standard, verifie que l adresse d expediteur et le `chainID`
demandes correspondent a la configuration autorisee du client, puis
delegue de la meme maniere au fournisseur de signature. Chaque appel,
qu il s agisse d une transaction ou d un payload de bloc, incremente des
compteurs Prometheus etiquetes par client et par statut
(`MetricSignTransactionTotal`, `MetricSignBlockPayloadTotal`), donnant une
observabilite fine sur qui signe quoi et avec quel taux d echec.
