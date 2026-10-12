# Chapitre 14 — Couplage des composants

Le chapitre précédent avait pour objet la cohésion des composants avec les principes REP, CCP et CRP.

Ce chapitre se concentre désormais sur le couplage : c'est-à-dire sur la manière dont les composants dépendent les uns des autres.

Trois principes sont particulièrement importants :

- ADP — Acyclic Dependencies Principle
- SDP — Stable Dependencies Principle
- SAP — Stable Abstractions Principle

---

## 1. ADP — Acyclic Dependencies Principle

### Principe des dépendances acycliques

> « N'autorisez aucun cycle dans le graphe des dépendances entre composants. »

En anglais :

> Allow no cycles in the component dependency graph.

### Idée fondamentale

Une dépendance simple est acceptable :

A → B → C

Mais un cycle est un signe de mauvais couplage :

A → B → C
↑     |
└─────┘

Ce schéma crée un cycle de dépendances.

### Problème principal

Les cycles rendent les composants fortement couplés. Ils créent une forme de dépendance circulaire où chacun dépend de l'autre. Cela engendre de nombreux effets négatifs :

- difficulté de compilation,
- tests fragiles,
- releases plus risquées,
- ordre de construction difficile à déterminer,
- modification coûteuse et peu sûre.

---

## 2. Le syndrome du lendemain matin

Imaginez une situation classique de travail en équipe :

- 08:00 → développement
- 12:00 → développement
- 17:00 → tests
- 18:00 → tout fonctionne

Vous rentrez chez vous.

Le lendemain matin :

- le projet ne compile plus,
- vos tests échouent,
- votre fonctionnalité ne fonctionne plus.

### Pourquoi ?

Parce qu'un autre développeur a modifié un composant dont votre code dépendait.

Le chapitre appelle cela le :

- « morning after syndrome »
- ou « syndrome du lendemain matin »

Ce problème devient particulièrement grave lorsque le nombre de développeurs augmente et que le projet grandit.

---

## 3. Le problème du développement en équipe

Supposons une architecture avec plusieurs composants :

Component A
↓
Component B
↓
Component C

Si chaque équipe travaille indépendamment et publie régulièrement ses composants, les autres équipes peuvent décider quand adopter une nouvelle version.

Par exemple :

- payment@1.0.0
- payment@1.1.0
- payment@2.0.0

L'équipe Order peut continuer à utiliser :

- payment@1.1.0

pendant que l'équipe Payment travaille sur :

- payment@2.0.0

Cette séparation réduit fortement le couplage entre les équipes.

Le but d'une bonne architecture de composants est précisément d'éviter qu'un changement dans un module force automatiquement une cascade de changements dans d'autres modules.

---

## 4. Pourquoi les cycles sont dangereux

Considérons ce graphe :

Entities → Database
↓
Interactors
↓
Presenters

Cette structure est acyclique.

Elle peut être représentée comme un DAG :

- Directed Acyclic Graph
- graphe orienté acyclique

Autrement dit, il est impossible de partir d'un composant, de suivre les dépendances et de revenir au composant initial.

---

## 5. Exemple TypeScript d'un graphe acyclique

Supposons une structure en couches :

- domain
- application
- infrastructure

### Domaine

```ts
// domain/user.ts
export interface User {
  id: string;
  name: string;
}
```

### Application

```ts
// application/user-service.ts
import { User } from '../domain/user';

export class UserService {
  getUser(): User {
    return {
      id: '1',
      name: 'Patrice',
    };
  }
}
```

### Infrastructure

```ts
// infrastructure/user-controller.ts
import { UserService } from '../application/user-service';

export class UserController {
  constructor(
    private readonly userService: UserService,
  ) {}

  getUser() {
    return this.userService.getUser();
  }
}
```

Dans ce cas, la direction est :

Controller → Application → Domain

Il n'y a pas de retour :

Domain → Controller

Donc le graphe est acyclique.

---

## 6. L'effet d'un cycle

Supposons maintenant que Entities dépende de Authorizer.

On obtient alors :

Entities → Authorizer → Interactors → Entities

C'est un cycle.

### Conséquence

Les composants concernés deviennent pratiquement un seul gros composant.

Ils doivent être compatibles avec les mêmes versions. Le découplage disparaît entièrement.

---

## 7. Conséquence sur les tests

Un cycle peut rendre les tests unitaires très difficiles.

Par exemple, nous voulons tester :

- Entities

Mais Entities dépend de :

- Authorizer
- Interactors

Pour tester le composant principal, il faut donc construire et intégrer plusieurs composants.

Au lieu de :

- Test Entities

on obtient :

- Test Entities
- Authorizer
- Interactors
- Database
- ...

Le test devient plus lent, plus fragile et plus difficile à isoler.

### Donc les cycles rendent plus difficile :

- les tests unitaires,
- les releases,
- les builds,
- l'ordre de compilation,
- la compréhension globale du système.

---

## 8. Comment casser un cycle ?

Le chapitre présente deux solutions majeures.

### Solution 1 — Dependency Inversion Principle (DIP)

Le DIP permet d'inverser la dépendance.

Supposons qu'un composant stable dépende d'un composant flexible, alors que l'on souhaite éviter cette dépendance.

On crée une abstraction :

```ts
export interface PermissionChecker {
  hasPermission(userId: string, permission: string): boolean;
}
```

Puis :

```ts
export class User {
  constructor(
    private readonly permissionChecker: PermissionChecker,
  ) {}

  canAccess(permission: string): boolean {
    return this.permissionChecker.hasPermission('user-1', permission);
  }
}
```

L'implémentation concrète peut être fournie ailleurs :

```ts
export class AuthorizationService implements PermissionChecker {
  hasPermission(userId: string, permission: string): boolean {
    return true;
  }
}
```

La dépendance concrète est ainsi remplacée par une abstraction.

On obtient :

User → PermissionChecker
↑
AuthorizationService

au lieu de :

User → AuthorizationService

Le DIP permet donc de casser le cycle.

---

### Solution 2 — Créer un nouveau composant

La seconde solution consiste à extraire ce qui est partagé.

Avant :

Entities ─────→ Authorizer
↑              ↓
└──────────────┘

On peut créer un composant partagé :

Shared
↗      ↖
Entities  Authorizer

Par exemple :

```text
libs/
├── entities/
├── authorizer/
└── shared-contracts/
```

Et dans `shared-contracts` :

```ts
export interface PermissionChecker {
  hasPermission(userId: string, permission: string): boolean;
}
```

Les deux composants dépendent alors de `shared-contracts`, mais ne dépendent plus directement l'un de l'autre.

---

## 9. Le « Jitters »

La structure des composants n'est pas figée.

À mesure que les exigences changent :

- nouvelles exigences,
- nouvelles dépendances,
- nouveaux composants,
- nouveaux risques de cycles,
- refactoring.

Le graphe des dépendances peut donc :

- évoluer,
- grandir,
- se restructurer.

Le rôle de l'architecte consiste à surveiller continuellement ce graphe et à casser les cycles lorsqu'ils apparaissent.

---

## 10. Pourquoi le design des composants ne se fait pas entièrement au début

Le chapitre tire une conclusion importante :

La structure des composants ne peut pas être conçue entièrement de manière top-down.

Elle évolue avec le système.

### Au début

- peu de code,
- peu de dépendances,
- architecture simple.

### Plus tard

- plus de fonctionnalités,
- plus de classes,
- plus de composants,
- plus de dépendances,
- besoin de restructuration.

Le graphe des dépendances représente donc moins la fonctionnalité métier elle-même que :

- la capacité du système à être construit,
- la capacité du système à être maintenu,
- la capacité du système à être modifié.

---

## 11. Isoler la volatilité

Une idée importante est de protéger les composants stables contre les composants qui changent fréquemment.

Par exemple :

GUI → Application → Business Rules

Nous ne voulons pas que :

- une modification du CSS
- ou du rendu visuel

provoque une modification des :

- Business Rules

De la même manière, une modification d'un rapport ne devrait pas avoir d'impact sur des politiques métier de haut niveau.

L'architecte cherche donc à isoler les éléments volatils des éléments stables.

---

## 12. SDP — Stable Dependencies Principle

### Principe des dépendances stables

> « Dépendez dans la direction de la stabilité. »

En anglais :

> Depend in the direction of stability.

### Sens du principe

Un composant que nous voulons rendre facilement modifiable ne doit pas être utilisé par un composant difficile à modifier.

Sinon :

Stable → Unstable

Le composant instable devient lui aussi difficile à modifier.

---

## 13. Exemple

Supposons :

Stable → Flexible

Flexible a été conçu pour être modifié rapidement.

Mais Stable dépend de Flexible.

Une modification de Flexible oblige alors à vérifier :

- Flexible,
- Stable,
- tous les composants dépendant de Stable.

La flexibilité de Flexible disparaît.

C'est une violation du SDP.

---

## 14. Que signifie « stable » ?

Stable ne signifie pas simplement :

> « qui ne change jamais ».

La stabilité concerne surtout :

- la quantité d'effort nécessaire pour modifier un composant,
- le nombre de dépendants qu'un composant a,
- le niveau de risque associé à une modification.

Un composant avec beaucoup de dépendants est généralement plus difficile à modifier.

Exemple :

A → B
→ C
→ D

X reçoit beaucoup de dépendances entrantes.

Modifier X implique potentiellement de vérifier l'impact sur :

- B,
- C,
- D,
- et tous les composants qui en dépendent.

C'est précisément ce que l'on appelle un composant stable.

---

## 15. Conclusion

Le couplage entre composants est un sujet central du design logiciel.

Les composants doivent être conçus pour être :

- indépendants,
- testables,
- faciles à maintenir,
- robustes face aux évolutions.

Les principes ADP, SDP et SAP fournissent une base solide pour y parvenir :

- ADP : évite les cycles,
- SDP : oriente les dépendances vers la stabilité,
- SAP : fait converger stabilité et abstraction.

En pratique, cela signifie que l'architecte doit constamment :

- surveiller le graphe des dépendances,
- éliminer les cycles,
- isoler la volatilité,
- protéger les composants stables,
- rendre les abstractions plus résistantes que les implémentations concrètes.

Une bonne architecture de composants n'est donc pas seulement un plan initial. C'est un mécanisme de contrôle continu pour garder le système adaptable, sûr et évolutif.
