# Bot Monopoly multijoueur (avec fausses CB)

## Objectif
Créer un bot de jeu **Monopoly en multijoueur** (2 à 8 joueurs) piloté via commandes textuelles, avec des règles proches du Monopoly classique: achats, loyers dynamiques, maisons/hôtels, hypothèques et échanges.

---

## Fonctionnalités principales
- Création de partie (`/mono create`)
- Rejoindre une partie (`/mono join`)
- Lancer les dés (`/mono roll`)
- Acheter/refuser une propriété (`/mono buy`, `/mono pass`)
- Enchères automatiques quand une propriété n'est pas achetée
- Payer loyers, taxes, amendes
- Gérer prison, doubles, tours, faillite
- Construire/vendre maisons et hôtels
- Hypothéquer/déshypothéquer des propriétés
- Échanger argent, propriétés et cartes de sortie de prison
- Sauvegarde de l'état de partie

## Mode multijoueur
- Une partie = un salon / une room unique
- Ordre de tour déterminé au début de partie
- Verrouillage du tour: seul le joueur actif peut jouer
- Timeout configurable (ex: 60s) pour éviter les blocages
- Bot arbitre: valide toutes les actions automatiquement

---

## Règles Monopoly complètes (version bot)

### 1) Départ de partie
- Chaque joueur commence avec un capital initial (configurable, par défaut 1500).
- Tous les titres de propriété sont à la banque.
- Le premier joueur est déterminé aléatoirement, puis sens horaire.

### 2) Déplacement, dés et doubles
- Le joueur lance 2 dés et avance de la somme.
- S'il fait un double, il rejoue.
- Après **3 doubles consécutifs**, il va en prison immédiatement (sans déplacer au 3e double).
- Passage sur la case Départ: créditer le montant configuré (par défaut 200).

### 3) Cases et résolution
- **Propriété libre**: le joueur peut acheter au prix affiché.
- **Refus d'achat**: la propriété part en enchère ouverte aux autres joueurs.
- **Propriété possédée par un autre joueur**: paiement d'un loyer.
- **Compagnie (Utilities)**: loyer basé sur le lancer de dés.
- **Gare (Railroads)**: loyer selon le nombre de gares possédées.
- **Taxe / amende**: débit immédiat vers la banque.
- **Chance / Caisse de communauté**: exécuter l'effet de carte.
- **Prison / Allez en prison**: appliquer les règles prison.

### 4) Loyers détaillés
#### Terrains (couleurs)
- Loyer de base si aucune construction.
- Si propriétaire détient le **monopole de couleur** et aucune maison: loyer de base doublé.
- Avec maisons/hôtel: loyer selon table de la case (1,2,3,4 maisons, puis hôtel).

#### Gares
- 1 gare: loyer 25
- 2 gares: loyer 50
- 3 gares: loyer 100
- 4 gares: loyer 200

#### Compagnies (Utilities)
- 1 compagnie détenue: loyer = `4 x somme des dés`
- 2 compagnies détenues: loyer = `10 x somme des dés`

### 5) Construction de maisons/hôtels
- Conditions pour construire:
  - Posséder toutes les propriétés du groupe de couleur.
  - Aucune propriété du groupe ne doit être hypothéquée.
  - Respect de la règle de construction **uniforme** (équilibrage des maisons).
- Maison:
  - Achat au prix maison de la case.
  - Maximum 4 maisons par propriété avant hôtel.
- Hôtel:
  - Nécessite 4 maisons sur la propriété.
  - Conversion des 4 maisons en 1 hôtel (selon disponibilité banque).
- Vente:
  - Revente maison/hôtel à 50% du coût d'achat.

### 6) Hypothèque
- Une propriété sans bâtiment peut être hypothéquée.
- Valeur reçue = valeur d'hypothèque de la case.
- Une propriété hypothéquée ne génère pas de loyer.
- Levée d'hypothèque: montant hypothèque + 10% d'intérêt (arrondi configurable).
- On ne peut pas construire sur un groupe contenant une propriété hypothéquée.

### 7) Échanges entre joueurs
- Échanges autorisés à tout moment hors résolution automatique bloquante (ex: paiement immédiat en cours).
- Un échange peut inclure:
  - Argent
  - Propriétés
  - Carte(s) « Sortie de prison »
- Un échange doit être explicitement validé par les 2 joueurs.
- Si une propriété avec bâtiments est échangée, le bot doit refuser tant que les bâtiments ne sont pas vendus (règle classique).
- Si une propriété hypothéquée est reçue:
  - Le nouveau propriétaire peut payer immédiatement l'intérêt de 10%, ou
  - Garder l'hypothèque et payer lors de la levée.

### 8) Prison
- Entrée prison si:
  - carte « Allez en prison »,
  - case « Allez en prison »,
  - 3 doubles consécutifs.
- Sortie possible:
  - payer l'amende,
  - utiliser une carte « Sortie de prison »,
  - faire un double dans les 3 tours.
- Si échec sur 3 tours: paiement obligatoire puis déplacement.

### 9) Faillite
- Si un joueur ne peut pas payer, il peut (dans cet ordre):
  1. vendre bâtiments,
  2. hypothéquer,
  3. proposer des échanges.
- Si toujours insolvable: faillite.
- En faillite contre un joueur: transfert des actifs au créancier (avec gestion hypothèques).
- En faillite contre la banque: actifs retournent à la banque, puis enchères.

---

## Système « fausses CB » (mécanique spéciale)
Idée demandée: intégrer des **fausses cartes bancaires (CB)** comme mécanique personnalisée.

### Règle proposée
- Chaque joueur reçoit 1 carte « Fausse CB » au démarrage.
- Utilisable **1 fois par partie**.
- Effet: annuler ou réduire un paiement (loyer/taxe) pendant un tour.
- Risque de contrôle:
  - 50%: paiement annulé
  - 50%: pénalité (double paiement ou amende fixe)

### Compatibilité avec les règles classiques
- La fausse CB ne peut pas éviter une faillite déjà déclarée.
- La fausse CB ne s'applique pas aux achats volontaires (achat propriété, maison, levée d'hypothèque).
- Son usage et son résultat sont journalisés pour audit de partie.

### Commandes liées
- `/mono cb use` : tente l'utilisation de la fausse CB
- `/mono cb status` : affiche si la carte a déjà été utilisée

---

## Commandes multijoueur recommandées
- `/mono create`
- `/mono join`
- `/mono start`
- `/mono state`
- `/mono roll`
- `/mono buy`
- `/mono pass`
- `/mono auction bid <montant>`
- `/mono build <propriete> <nb>`
- `/mono sell-house <propriete> <nb>`
- `/mono mortgage <propriete>`
- `/mono unmortgage <propriete>`
- `/mono trade propose <joueur> ...`
- `/mono trade accept <tradeId>`
- `/mono trade reject <tradeId>`
- `/mono cb use`
- `/mono cb status`

---

## Modèle de données minimal (étendu)

```txt
Game {
  id,
  status,
  players[],
  currentTurn,
  boardState,
  bank,
  diceState,
  auctionState,
  createdAt,
  updatedAt
}

Player {
  id,
  name,
  cash,
  position,
  properties[],
  jailTurns,
  getOutOfJailCards,
  fakeCardUsed,
  bankrupt
}

Property {
  id,
  name,
  type,              // street, railroad, utility
  group,
  ownerId,
  purchasePrice,
  baseRent,
  rentTable,
  houseCost,
  houses,            // 0-4
  hotel,             // bool
  mortgaged
}

Trade {
  id,
  fromPlayerId,
  toPlayerId,
  offer,
  request,
  status             // pending, accepted, rejected, canceled
}
```

---

## Flow d'un tour (version complète)
1. Vérifier joueur actif
2. Résoudre état prison éventuel
3. Lancer dés
4. Déplacer pion
5. Résoudre case (achat/enchère, loyer, taxe, carte, prison...)
6. Autoriser actions de gestion (construction, hypothèque, échange)
7. Autoriser action spéciale « fausse CB » si éligible
8. Vérifier faillite éventuelle
9. Fin de tour => joueur suivant (ou relance si double valide)

---

## Critères d'acceptation
- La partie tourne de 2 à 8 joueurs sans blocage de tour.
- Le bot empêche les actions hors tour.
- Les loyers sont corrects pour terrains, gares et compagnies.
- Les règles de maisons/hôtels (monopole + équilibrage) sont respectées.
- Les hypothèques empêchent la perception de loyer et la construction.
- Les échanges sont validés par double confirmation.
- Les faillites transfèrent correctement les actifs.
- La mécanique « fausse CB » est disponible, limitée à 1 usage, et auditée.
