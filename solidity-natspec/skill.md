---
name: solidity-natspec
description: Applique la documentation NatSpec obligatoire sur tous les contrats Solidity — contrats, interfaces, libraries, fonctions, events, errors, modifiers et variables d'etat publiques.
---

# Skill : NatSpec — Documentation Solidity

Ce skill s'applique automatiquement lors de la creation ou modification de fichiers Solidity (`.sol`).

## Header de contrat / interface / library

Chaque contrat, interface ou library doit avoir un header NatSpec :

```solidity
/// @title Nom descriptif du contrat
/// @notice Description metier courte (1-2 lignes).
/// @dev Details techniques : pattern utilise, roles d'acces, invariants.
contract MyContract {
```

- `@title` : nom lisible du contrat.
- `@notice` : ce que fait le contrat pour un utilisateur final.
- `@dev` : details pour les developpeurs (pattern, acces, risques).

## Fonctions

Toute fonction public/external/internal doit avoir un NatSpec :

```solidity
/// @notice Transfere des jetons au destinataire.
/// @dev Emet {Transfer}. Revert si solde insuffisant. Suit checks-effects-interactions.
/// @param to Adresse du destinataire
/// @param amount Montant en unites minimales
/// @return success Vrai si le transfert a abouti
function transfer(address to, uint256 amount) external returns (bool success);
```

### Regles

- `@notice` : objectif metier de la fonction.
- `@dev` : hypotheses, side-effects, gas, events emis, pattern (checks-effects-interactions).
- `@param` : un par parametre, description concise.
- `@return` : un par valeur de retour.
- Mentionner les exigences d'acces (Ownable, AccessControl, roles).

## Events / Errors / Modifiers

```solidity
/// @notice Emis lors d'un transfert de jetons.
/// @param from Adresse source
/// @param to Adresse destinataire
/// @param amount Montant transfere
event Transfer(address indexed from, address indexed to, uint256 amount);

/// @notice Revert si le solde est insuffisant.
/// @param available Solde disponible
/// @param required Montant requis
error InsufficientBalance(uint256 available, uint256 required);

/// @notice Restreint l'acces au proprietaire du contrat.
modifier onlyOwner() {
```

## Variables d'etat publiques

Les variables d'etat publiques significatives ont un NatSpec court :

```solidity
/// @notice Solde de chaque adresse.
mapping(address => uint256) public balances;
```

## Securite

- Mentionner `@custom:security` pour les risques identifies :

```solidity
/// @custom:security Reentrancy — protege par nonReentrant.
/// @custom:security Flash-loan — prix oracle TWAP.
```

## Ce qui n'a PAS besoin de NatSpec

- Fonctions `private` dont le nom est explicite.
- Variables internes evidentes.
- Constructeurs triviaux (juste des assignments).

## Langue

- Les NatSpec sont rediges en **francais**.
- Les noms de parametres restent en anglais (convention code Solidity).

## Application

Lors de l'ecriture ou la modification de code Solidity :

1. Ajouter le header contrat/interface/library s'il est absent.
2. Documenter les fonctions public/external nouvelles ou modifiees.
3. Documenter events, errors et modifiers.
4. Mettre a jour le NatSpec si la signature ou le comportement change.
5. Ne pas sur-documenter — le code lisible se suffit a lui-meme.