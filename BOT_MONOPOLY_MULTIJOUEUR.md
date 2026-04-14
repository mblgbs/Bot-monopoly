# Bot Monopoly multijoueur (avec fausses CB)

## Objectif
Créer un bot de jeu **Monopoly en multijoueur** (2 à 8 joueurs) piloté via commandes textuelles.

## Fonctionnalités principales
- Création de partie (`/mono create`)
- Rejoindre une partie (`/mono join`)
- Lancer les dés (`/mono roll`)
- Acheter/refuser une propriété (`/mono buy`, `/mono pass`)
- Payer loyers, taxes, amendes
- Gérer prison, doubles, tours, faillite
- Sauvegarde de l'état de partie

## Mode multijoueur
- Une partie = un salon / une room unique
- Ordre de tour déterminé au début de partie
- Verrouillage du tour: seul le joueur actif peut jouer
- Timeout configurable (ex: 60s) pour éviter les blocages
- Bot arbitre: valide toutes les actions automatiquement

## Système "fausses CB" (faux moyens de paiement)
Idée demandée: intégrer des **fausses cartes bancaires (CB)** comme mécanique spéciale.

### Règle proposée
- Chaque joueur reçoit 1 carte "Fausse CB" au démarrage.
- Utilisation possible **1 fois par partie**.
- Effet: annuler ou réduire un paiement (ex: loyer ou taxe) pendant un tour.
- Risque de contrôle:
  - 50%: paiement annulé
  - 50%: pénalité (double paiement ou amende fixe)

### Commandes liées
- `/mono cb use` : tente l'utilisation de la fausse CB
- `/mono cb status` : affiche si la carte a déjà été utilisée

## Modèle de données minimal

```txt
Game {
  id,
  status,
  players[],
  currentTurn,
  boardState,
  bank,
  createdAt
}

Player {
  id,
  name,
  cash,
  position,
  properties[],
  inJail,
  fakeCardUsed,
  bankrupt
}
```

## Flow d'un tour
1. Vérifier joueur actif
2. Lancer dés
3. Déplacer pion
4. Résoudre case (achat, loyer, taxe, carte, prison...)
5. Autoriser action spéciale (dont fausse CB si éligible)
6. Fin de tour => joueur suivant

## MVP recommandé
1. Plateau simplifié (20 cases)
2. Pas d'enchères au départ
3. Loyers fixes
4. 1 seule carte spéciale: "fausse CB"
5. Interface commandes texte uniquement

## Exemples de commandes utilisateur
- `/mono create`
- `/mono join`
- `/mono start`
- `/mono roll`
- `/mono buy`
- `/mono cb use`
- `/mono state`

## Critères d'acceptation
- La partie tourne de 2 à 8 joueurs sans blocage de tour.
- Le bot empêche les actions hors tour.
- Les paiements et loyers sont calculés automatiquement.
- La mécanique "fausse CB" est disponible et tracée dans l'état du joueur.
