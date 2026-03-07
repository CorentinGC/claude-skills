---
name: smart-contract-audit
description: Audit de securite pour smart contracts EVM (Solidity/Vyper) — reentrancy, access control, overflow, flash loans, oracle manipulation, proxy patterns, gas optimization. A invoquer avec /smart-contract-audit.
user_invocable: true
---

# Skill : Smart Contract Security Audit

Audit de securite complet pour smart contracts EVM-compatible (Ethereum, BSC, Polygon, Arbitrum, etc.). Couvre Solidity et Vyper.

## Procedure d'audit

Executer chaque section dans l'ordre. Pour chaque probleme trouve, indiquer la severite (CRITICAL / HIGH / MEDIUM / LOW / GAS) et proposer un correctif.

---

### 1. Reentrancy

- Rechercher les appels externes (`call`, `transfer`, `send`, interactions avec d'autres contrats) suivis de modifications d'etat
- Verifier le pattern Checks-Effects-Interactions (CEI) : modifier l'etat AVANT l'appel externe
- Verifier la presence de `ReentrancyGuard` (OpenZeppelin) sur les fonctions sensibles
- Attention a la reentrancy cross-function et cross-contract (read-only reentrancy incluse)

```solidity
// CRITICAL — reentrancy
function withdraw(uint amount) external {
    (bool success, ) = msg.sender.call{value: amount}("");
    require(success);
    balances[msg.sender] -= amount; // Etat modifie APRES l'appel
}

// Correctif — CEI
function withdraw(uint amount) external nonReentrant {
    balances[msg.sender] -= amount; // Etat modifie AVANT l'appel
    (bool success, ) = msg.sender.call{value: amount}("");
    require(success);
}
```

---

### 2. Access Control

- Verifier que toutes les fonctions sensibles ont des modificateurs d'acces (`onlyOwner`, `onlyRole`, `onlyAdmin`)
- Verifier que `tx.origin` n'est jamais utilise pour l'authentification (utiliser `msg.sender`)
- Verifier les fonctions `public` / `external` — chaque fonction doit avoir la visibilite minimale necessaire
- Verifier les fonctions d'initialisation (`initialize`) — appelables une seule fois ? Protegees ?
- Verifier la gestion de l'ownership : transfert en 2 etapes (Ownable2Step) prefere a Ownable simple
- Verifier les fonctions `selfdestruct` / `delegatecall` — acces restreint obligatoire

```solidity
// CRITICAL — pas de controle d'acces
function setPrice(uint _price) external {
    price = _price;
}

// HIGH — tx.origin
function transfer(address to, uint amount) external {
    require(tx.origin == owner); // Vulnerable au phishing
}
```

---

### 3. Integer Overflow / Underflow

- Solidity >= 0.8.0 : overflow/underflow check automatique — verifier les blocs `unchecked {}`
- Solidity < 0.8.0 : verifier l'utilisation de SafeMath sur TOUTES les operations arithmetiques
- Attention aux casts : `uint256` -> `uint128`, `int256` -> `uint256` (perte de donnees, valeurs negatives)
- Verifier les divisions par zero
- Verifier les multiplications avant division (precision) : `a * b / c` au lieu de `a / c * b`

```solidity
// HIGH — perte de precision
uint256 reward = totalReward / totalStakers * userStake; // Arrondi a zero si totalStakers > totalReward

// Correctif
uint256 reward = totalReward * userStake / totalStakers;
```

---

### 4. Manipulation d'Oracle / Prix

- Verifier les sources de prix — ne jamais utiliser `balanceOf` ou `getReserves` de pool AMM comme oracle spot
- Verifier l'utilisation de TWAP (Time-Weighted Average Price) ou d'oracles decentralises (Chainlink, Pyth)
- Verifier les controles de sante sur les reponses oracle : `updatedAt` recente, `answeredInRound >= roundId`, `price > 0`
- Verifier les prix stale (delai max acceptable entre 2 mises a jour)
- Verifier les oracles multi-source (fallback en cas de panne)

```solidity
// CRITICAL — oracle sans validation
(, int price, , , ) = priceFeed.latestRoundData();
return uint256(price);

// Correctif — validation complete
(uint80 roundId, int price, , uint updatedAt, uint80 answeredInRound) = priceFeed.latestRoundData();
require(price > 0, "Invalid price");
require(updatedAt > block.timestamp - MAX_STALENESS, "Stale price");
require(answeredInRound >= roundId, "Stale round");
return uint256(price);
```

---

### 5. Flash Loan Attacks

- Identifier les fonctions dont le comportement depend de balances instantanees ou de ratios de pool
- Verifier que les mecanismes de gouvernance (votes, quorum) ne sont pas manipulables via flash loan
- Verifier les calculs de prix bases sur les reserves de pool (manipulables dans la meme transaction)
- Verifier les mecanismes de snapshot vs balance instantanee pour le voting power

---

### 6. Front-running / MEV

- Identifier les transactions sensibles a l'ordre d'execution (swaps, liquidations, minting)
- Verifier la presence de slippage protection (`amountOutMin`, `deadline`) sur les swaps
- Verifier les mecanismes commit-reveal pour les actions sensibles (encheres, loteries)
- Verifier les deadlines sur les transactions (`block.timestamp` vs parametre deadline)
- Attention aux fonctions `approve` avec race condition — utiliser `increaseAllowance` / `decreaseAllowance`

```solidity
// HIGH — pas de protection slippage
router.swapExactTokensForTokens(amountIn, 0, path, to, block.timestamp);

// Correctif
router.swapExactTokensForTokens(amountIn, amountOutMin, path, to, deadline);
```

---

### 7. Proxy et Upgradeability

- Verifier les storage collisions entre proxy et implementation (layout compatible)
- Verifier que `initialize()` remplace le `constructor` et est protegee par `initializer`
- Verifier l'absence de `constructor` avec logique dans les contrats d'implementation
- Verifier que `_disableInitializers()` est appele dans le constructor de l'implementation
- Verifier les gaps de storage (`uint256[50] private __gap`) pour les futurs upgrades
- Verifier que les fonctions `upgradeTo` / `upgradeToAndCall` sont protegees
- Verifier la compatibilite du storage layout entre versions (pas de reordonnancement de variables)
- UUPS : verifier que `_authorizeUpgrade` est protege

```solidity
// CRITICAL — initializer non protege
function initialize(address _owner) public {
    owner = _owner;
}

// Correctif
function initialize(address _owner) public initializer {
    __Ownable_init(_owner);
}
```

---

### 8. Appels externes et interactions

- Verifier les retours de `call` / `delegatecall` — toujours verifier le `bool success`
- Verifier les retours de `transfer` ERC20 — certains tokens ne retournent pas `bool` (USDT)
- Utiliser SafeERC20 (OpenZeppelin) pour les interactions avec des tokens ERC20 arbitraires
- Verifier les tokens avec fee-on-transfer (le montant recu != montant envoye)
- Verifier les tokens avec rebase (balance change sans transfert)
- Verifier les tokens avec blacklist (USDC, USDT)
- Attention aux `delegatecall` vers des adresses controlees par l'utilisateur

```solidity
// HIGH — retour non verifie
IERC20(token).transfer(to, amount);

// Correctif
IERC20(token).safeTransfer(to, amount); // SafeERC20
```

---

### 9. Denial of Service (DoS)

- Verifier les boucles sur des tableaux de taille non bornee (gas limit)
- Verifier les patterns push-over-pull (preferer le pull pattern pour les distributions)
- Verifier les `require` / `revert` dependant d'appels externes (un contrat malveillant peut bloquer)
- Verifier les blocages par `selfdestruct` force (ETH envoye a un contrat sans `receive`)
- Verifier les deadlocks dans les mecanismes de timelock/governance

```solidity
// HIGH — DoS par gas limit
function distributeRewards(address[] memory users) external {
    for (uint i = 0; i < users.length; i++) {
        payable(users[i]).transfer(rewards[users[i]]);
    }
}

// Correctif — pull pattern
function claimReward() external {
    uint amount = rewards[msg.sender];
    rewards[msg.sender] = 0;
    payable(msg.sender).transfer(amount);
}
```

---

### 10. Randomness

- `block.timestamp`, `block.difficulty`, `blockhash` ne sont PAS des sources de hasard securisees (manipulables par les mineurs/validateurs)
- Utiliser Chainlink VRF ou un equivalent pour le hasard on-chain
- Verifier les mecanismes commit-reveal pour les loteries/jeux

```solidity
// CRITICAL — randomness previsible
uint random = uint(keccak256(abi.encodePacked(block.timestamp, msg.sender))) % 100;
```

---

### 11. Conformite aux standards (ERC)

#### ERC-20
- Verifier `approve` / `transferFrom` — race condition, valeurs de retour
- Verifier `decimals()`, `totalSupply()`, `balanceOf()`
- Verifier les events `Transfer` et `Approval` emis correctement

#### ERC-721
- Verifier `safeTransferFrom` — callback `onERC721Received` sur le destinataire
- Verifier les `tokenURI` — base URI correcte, metadata conforme

#### ERC-1155
- Verifier `safeBatchTransferFrom` — callbacks, arrays de meme longueur
- Verifier les balances batch

#### ERC-4626 (Vault)
- Verifier les fonctions `deposit`, `withdraw`, `redeem`, `mint` — rounding correct
- Verifier la protection contre l'inflation attack (virtual shares/assets)

---

### 12. Gas Optimization

Severite GAS (non-bloquant mais recommande) :

- Variables de storage : packing (variables < 256 bits groupees dans le meme slot)
- `calldata` au lieu de `memory` pour les parametres de fonction `external` en lecture seule
- `++i` au lieu de `i++` dans les boucles (pre-increment)
- `!= 0` au lieu de `> 0` pour les comparaisons uint
- `immutable` pour les variables assignees uniquement dans le constructor
- `constant` pour les valeurs connues a la compilation
- Events au lieu de storage pour les donnees qui n'ont pas besoin d'etre lues on-chain
- Utiliser `error CustomError()` au lieu de `require(condition, "long string")` (Solidity >= 0.8.4)
- Verifier les lectures de storage repetees — cacher dans une variable locale

```solidity
// GAS — storage reads repetees
function calculate() external view returns (uint) {
    return storedValue * storedValue + storedValue; // 3 SLOAD

    // Correctif
    uint _value = storedValue; // 1 SLOAD
    return _value * _value + _value;
}
```

---

### 13. Logique metier et invariants

- Identifier les invariants du protocole et verifier qu'ils tiennent dans tous les cas
- Verifier les edge cases : montants a zero, tableaux vides, adresses zero
- Verifier les conditions de liquidation — seuils corrects, pas de manipulation possible
- Verifier les mecanismes de fees — pas de contournement possible, pas d'accumulation d'erreurs d'arrondi
- Verifier les timelocks et delais — coherents, pas bypassables
- Verifier les conditions de pause/unpause — qui peut, quand, quelles fonctions sont affectees

---

### 14. Signature et Cryptographie

- Verifier la protection contre le replay (nonce, chainId, deadline dans le hash)
- Verifier la malleabilite des signatures ECDSA — utiliser OpenZeppelin ECDSA
- Verifier que `ecrecover` ne retourne pas `address(0)` sans verification
- EIP-712 : verifier le domain separator (chainId, verifyingContract, name, version)
- Verifier les signatures EIP-2612 (permit) — deadline, nonce

```solidity
// HIGH — signature replay
function execute(bytes memory signature, uint amount) external {
    address signer = ECDSA.recover(keccak256(abi.encode(amount)), signature);
    // Pas de nonce, pas de chainId — rejouable
}
```

---

## Outils recommandes

| Outil | Usage |
|-------|-------|
| Slither | Analyse statique automatique (detection patterns connus) |
| Mythril | Analyse symbolique (chemins d'execution) |
| Foundry (forge test) | Tests unitaires et fuzz testing |
| Echidna | Fuzzing property-based |
| Certora | Verification formelle |
| Solhint | Linter Solidity |

---

## Format du rapport

Produire un rapport structure :

```
## Rapport de securite — [Nom du contrat]

### Resume
- CRITICAL : X
- HIGH : Y
- MEDIUM : Z
- LOW : W
- GAS : V

### Informations generales
- **Compilateur** : solc X.Y.Z
- **Chain cible** : Ethereum / BSC / Polygon / ...
- **Standards** : ERC-20 / ERC-721 / ...
- **Dependances** : OpenZeppelin vX.Y.Z / ...

### Problemes detectes

#### [CRITICAL] Titre du probleme
- **Contrat** : `ContractName.sol`
- **Fonction** : `functionName()`
- **Ligne** : XX
- **Description** : ...
- **Impact** : ...
- **PoC** (si applicable) : ...
- **Correctif** : ...

(repeter pour chaque probleme)

### Recommandations generales
- ...
```

Proposer des correctifs pour les problemes CRITICAL et HIGH. Demander confirmation a l'utilisateur avant de modifier la logique metier du contrat.
