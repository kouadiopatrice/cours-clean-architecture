

# Chapitre 13 — Cohésion des composants

## 1. Introduction

La question fondamentale du chapitre est :

> **Quelles classes doivent appartenir à quels composants ?**

Cette décision est importante et doit être guidée par de bons principes d'ingénierie logicielle plutôt que prise arbitrairement selon le contexte. 

Le chapitre présente trois principes de cohésion des composants :

1. **REP — Reuse/Release Equivalence Principle**

   * Principe d'équivalence réutilisation/version.
2. **CCP — Common Closure Principle**

   * Principe de fermeture commune.
3. **CRP — Common Reuse Principle**

   * Principe de réutilisation commune.

Ces trois principes répondent à une même question :

> **Comment décider quelles classes et quels modules doivent être regroupés dans un même composant ?**

---

# 2. REP — Reuse/Release Equivalence Principle

## Principe

> **The granule of reuse is the granule of release.**

### Traduction

> **L'unité de réutilisation doit être l'unité de livraison/versionnement.**

L'idée est simple :

> Si plusieurs classes sont réutilisées ensemble, elles devraient idéalement être versionnées et livrées ensemble.

Le texte explique que les composants réutilisables doivent être suivis à travers un processus de release avec des numéros de version. Les utilisateurs doivent savoir **quelle version ils utilisent** et **quels changements ont été introduits**. 

---

## Exemple TypeScript

Imaginons une librairie :

```text
@company/payment
```

avec :

```text
payment/
├── PaymentService.ts
├── PaymentResult.ts
├── PaymentException.ts
└── PaymentProvider.ts
```

Ces éléments forment un composant cohérent :

```ts
export interface PaymentProvider {
  pay(amount: number): Promise<PaymentResult>;
}
```

```ts
export interface PaymentResult {
  transactionId: string;
  status: 'success' | 'failed' | 'pending';
}
```

```ts
export class PaymentService {
  constructor(
    private readonly provider: PaymentProvider
  ) {}

  pay(amount: number): Promise<PaymentResult> {
    return this.provider.pay(amount);
  }
}
```

On peut publier l'ensemble :

```text
@company/payment@1.0.0
```

Puis :

```text
@company/payment@1.1.0
```

et les consommateurs savent que ces classes appartiennent au même composant et évoluent ensemble.

---

# 3. Ce que REP veut éviter

Imaginons une librairie qui contient :

```text
shared/
├── PaymentService.ts
├── UserValidator.ts
├── InvoiceCalculator.ts
├── DateUtils.ts
├── ProductMapper.ts
├── EmailService.ts
└── JwtHelper.ts
```

C'est typiquement un mauvais regroupement.

Pourquoi ?

Parce que ces éléments n'ont pas nécessairement la même raison d'être ni le même rythme d'évolution.

Par exemple :

```text
PaymentService
      ↓
évolue avec le domaine Payment

JwtHelper
      ↓
évolue avec l'authentification

InvoiceCalculator
      ↓
évolue avec la facturation
```

Les mettre dans une seule librairie :

```text
@company/shared
```

peut créer un composant artificiellement énorme.

---

# 4. CCP — Common Closure Principle

Le texte donne ensuite un principe particulièrement important :

> **Gather into components those classes that change for the same reasons and at the same times.**

### Traduction

> **Regroupez dans un même composant les classes qui changent pour les mêmes raisons et au même moment.**

Et inversement :

> **Séparez les classes qui changent à des moments différents ou pour des raisons différentes.** 

C'est essentiellement le **SRP appliqué au niveau des composants**.

---

# 5. CCP = SRP au niveau des composants

Tu connais déjà le **Single Responsibility Principle**.

Au niveau d'une classe :

```text
Une classe
    ↓
une raison de changer
```

Avec CCP :

```text
Un composant
    ↓
un ensemble cohérent de raisons de changer
```

Le livre le formule ainsi :

> Le SRP indique qu'une classe ne doit pas avoir plusieurs raisons de changer.

Le CCP applique exactement la même idée aux composants. 

---

# 6. Exemple TypeScript

Supposons une application e-commerce.

Une mauvaise organisation pourrait être :

```text
ecommerce/
├── UserService.ts
├── PaymentService.ts
├── ProductService.ts
├── InvoiceService.ts
├── EmailService.ts
└── ShippingService.ts
```

Tout est dans un même composant.

Mais les changements sont différents.

```text
UserService
     ↓
évolue avec les règles utilisateurs

PaymentService
     ↓
évolue avec les règles de paiement

ShippingService
     ↓
évolue avec les règles de livraison
```

Il serait préférable de séparer :

```text
components/
├── users/
├── payments/
├── products/
├── invoices/
└── shipping/
```

---

# 7. Exemple avec Angular + Nx

Dans un projet Nx, on pourrait avoir :

```text
libs/
├── users/
├── payments/
├── products/
├── invoices/
└── shipping/
```

Par exemple :

```text
libs/payments/
├── domain/
│   ├── payment.ts
│   └── payment-provider.ts
├── application/
│   └── process-payment.use-case.ts
└── infrastructure/
    └── payment-api.repository.ts
```

Si les règles métier du paiement changent, nous voulons idéalement modifier :

```text
libs/payments
```

sans provoquer des modifications dans :

```text
libs/users
libs/products
libs/shipping
```

C'est exactement l'esprit du **CCP**.

---

# 8. Pourquoi CCP améliore la maintenance ?

Supposons que la réglementation concernant les paiements change.

Avec une bonne architecture :

```text
Requirement change
       ↓
Payment
       ↓
libs/payments
```

On évite :

```text
Requirement change
       ↓
Payment
       ↓
shared
       ↓
20 composants impactés
```

Le texte souligne justement que lorsque les changements sont confinés à un seul composant, seul ce composant doit idéalement être redéployé et revalidé. 

---

# 9. CCP et OCP

Le CCP est fortement lié au **Open/Closed Principle**.

L'OCP dit :

> Les classes doivent être ouvertes à l'extension mais fermées à la modification.

Mais une fermeture à 100 % est impossible.

Il faut donc choisir :

> **De quels types de changements voulons-nous principalement protéger notre code ?**

Le CCP nous dit ensuite :

> **Regroupons dans le même composant les classes qui sont fermées aux mêmes types de changements.** 

---

# 10. Résumé CCP vs SRP

| Niveau    | Principe | Question                                        |
| --------- | -------- | ----------------------------------------------- |
| Classe    | SRP      | Qu'est-ce qui peut faire changer cette classe ? |
| Composant | CCP      | Quelles classes changent pour la même raison ?  |

On peut retenir :

```text
SRP
 ↓
Séparer les responsabilités au niveau des classes

CCP
 ↓
Séparer les raisons de changement au niveau des composants
```

Le livre résume les deux principes ainsi :

> **Regrouper ce qui change en même temps et pour les mêmes raisons. Séparer ce qui change à des moments différents ou pour des raisons différentes.** 

---

# 11. CRP — Common Reuse Principle

Le troisième principe est :

> **Don't force users of a component to depend on things they don't need.**

### Traduction

> **Ne forcez pas les utilisateurs d'un composant à dépendre de choses dont ils n'ont pas besoin.** 

C'est probablement le principe le plus facile à comprendre avec TypeScript.

---

# 12. Exemple simple

Imaginons :

```text
@company/shared
```

qui contient :

```text
shared/
├── DateUtils.ts
├── PaymentUtils.ts
├── UserUtils.ts
├── InvoiceUtils.ts
├── ProductUtils.ts
├── JwtUtils.ts
└── EmailUtils.ts
```

Ton application `orders` utilise uniquement :

```ts
import { DateUtils } from '@company/shared';
```

Mais elle dépend maintenant de tout le composant :

```text
orders
   ↓
@company/shared
   ├── DateUtils
   ├── PaymentUtils
   ├── UserUtils
   ├── InvoiceUtils
   ├── ProductUtils
   ├── JwtUtils
   └── EmailUtils
```

Même si `orders` n'utilise pas :

```text
JwtUtils
EmailUtils
PaymentUtils
```

il dépend quand même du composant qui les contient.

---

# 13. Pourquoi est-ce problématique ?

Supposons que quelqu'un modifie :

```ts
JwtUtils.ts
```

Ton application `orders` peut être obligée de :

```text
recompiler
    ↓
revalider
    ↓
redéployer
```

alors qu'elle n'utilise même pas `JwtUtils`.

C'est précisément le problème décrit par le CRP. 

---

# 14. Une meilleure organisation

Au lieu de :

```text
shared/
├── DateUtils
├── JwtUtils
├── EmailUtils
└── PaymentUtils
```

on peut créer des composants plus cohérents :

```text
libs/
├── date/
├── authentication/
├── email/
└── payment/
```

Ainsi :

```text
orders
   ↓
date
```

au lieu de :

```text
orders
   ↓
shared
   ├── date
   ├── authentication
   ├── email
   └── payment
```

---

# 15. CRP et ISP

Le texte fait un parallèle très important avec **ISP — Interface Segregation Principle**.

### ISP

> Ne forcez pas une classe à dépendre de méthodes qu'elle n'utilise pas.

### CRP

> Ne forcez pas un composant à dépendre de classes qu'il n'utilise pas.

On peut donc voir :

```text
ISP
 ↓
niveau interface / classe

CRP
 ↓
niveau composant
```

Le livre résume les deux avec une idée très simple :

> **Ne dépendez pas de ce dont vous n'avez pas besoin.** 

---

# 16. Exemple TypeScript : ISP

Mauvais design :

```ts
interface UserService {
  createUser(): void;
  deleteUser(): void;
  sendEmail(): void;
  generateInvoice(): void;
}
```

Un composant qui veut uniquement créer un utilisateur dépend de tout :

```ts
class UserController {
  constructor(
    private readonly service: UserService
  ) {}
}
```

On peut séparer :

```ts
interface UserCreator {
  createUser(): void;
}

interface UserDeleter {
  deleteUser(): void;
}
```

Le consommateur dépend uniquement de ce dont il a besoin :

```ts
class UserController {
  constructor(
    private readonly userCreator: UserCreator
  ) {}
}
```

C'est **ISP**.

---

# 17. CRP au niveau des composants

Imaginons maintenant :

```text
UserComponent
```

qui contient :

```text
UserCreator
UserDeleter
PaymentProcessor
InvoiceGenerator
EmailSender
```

Même si `UserController` n'utilise que :

```text
UserCreator
```

il dépend du composant complet.

Le CRP nous pousse à éviter ce regroupement artificiel.

---

# 18. Les trois principes ensemble

Voici maintenant la partie la plus importante du chapitre.

Les trois principes ne vont **pas toujours dans la même direction**.

Le texte explique que :

* **REP** et **CCP** ont tendance à rendre les composants plus grands ;
* **CRP** pousse au contraire à rendre les composants plus petits. 

On peut le représenter ainsi :

```text
                 REP
                  ▲
                 / \
                /   \
               /     \
              /       \
             /         \
            ▼           ▼
          CCP --------- CRP
```

L'architecte doit trouver un équilibre entre ces trois forces.

---

# 19. Exemple concret

Supposons :

```text
libs/
└── payment/
    ├── PaymentService.ts
    ├── PaymentProvider.ts
    ├── PaymentValidator.ts
    ├── PaymentLogger.ts
    ├── PaymentReport.ts
    ├── PaymentEmail.ts
    └── PaymentHistory.ts
```

### REP pourrait dire :

> Ces éléments appartiennent au même domaine et peuvent être réutilisés ensemble.

### CCP pourrait dire :

> PaymentService, PaymentProvider et PaymentValidator changent souvent ensemble.

Donc :

```text
payment-core/
```

est cohérent.

### CRP pourrait cependant dire :

> PaymentEmail et PaymentReport ne sont utilisés que par certaines applications.

Ils pourraient donc être séparés :

```text
payment/
├── core/
├── reporting/
└── notifications/
```

---

# 20. Application à une architecture Nx

Pour ton contexte Angular/Nx, on peut traduire les principes ainsi :

```text
libs/
├── authentication/
├── payment/
├── customer/
├── contract/
└── notification/
```

### REP

Une librairie doit représenter une unité cohérente de réutilisation et de versionnement.

```text
@company/payment
```

contient ce qui forme réellement le composant Payment.

---

### CCP

Les éléments qui changent pour les mêmes raisons doivent être regroupés.

```text
payment/
├── domain/
├── application/
└── infrastructure/
```

Si les règles métier Payment changent :

```text
Payment
   ↓
payment component
```

---

### CRP

Ne mets pas dans `payment` quelque chose qui n'est pas réellement utilisé avec Payment.

Évite par exemple :

```text
payment/
├── PaymentService
├── UserService
├── EmailService
├── ProductService
└── DateUtils
```

Le composant devient une dépendance massive.

---

# 21. Architecture concrète

Une organisation possible :

```text
libs/
│
├── payment/
│   ├── domain/
│   │   ├── entities/
│   │   └── repositories/
│   │
│   ├── application/
│   │   └── use-cases/
│   │
│   └── infrastructure/
│       └── repositories/
│
├── customer/
│   ├── domain/
│   ├── application/
│   └── infrastructure/
│
└── notification/
    ├── domain/
    ├── application/
    └── infrastructure/
```

Cela permet de combiner plusieurs principes :

```text
                  CCP
                   ↓
        ┌───────────────────┐
        │     PAYMENT       │
        │                   │
        │ Domain            │
        │ Application       │
        │ Infrastructure    │
        └───────────────────┘
                   ↑
                   │
                  CRP
                   │
       éviter les dépendances
       inutiles entre domaines
```

---

# 22. Attention : il ne faut pas appliquer les trois principes dogmatiquement

C'est un point essentiel du chapitre.

Une architecture parfaite aujourd'hui peut devenir mauvaise demain.

Le texte explique que la structure des composants **évolue avec la maturité du projet et son utilisation**. 

Par exemple, au début :

```text
Application
    ↓
payment
customer
users
```

Il peut être préférable de privilégier la **développabilité**.

Plus tard, lorsque plusieurs applications utilisent le même code :

```text
App A ─┐
       ├── payment
App B ─┤
       └── shared-payment
App C ─┘
```

la **réutilisabilité** devient plus importante.

---

# 23. Le triangle REP / CCP / CRP

Il faut donc comprendre les trois principes comme trois forces :

| Principe | Question principale                                   | Tendance               |
| -------- | ----------------------------------------------------- | ---------------------- |
| **REP**  | Qu'est-ce qui doit être réutilisé et livré ensemble ? | Composants plus grands |
| **CCP**  | Qu'est-ce qui change pour les mêmes raisons ?         | Composants plus grands |
| **CRP**  | De quoi les consommateurs ont-ils réellement besoin ? | Composants plus petits |

### REP

```text
Réutiliser ensemble
      ↓
Livrer ensemble
```

### CCP

```text
Changer ensemble
      ↓
Regrouper ensemble
```

### CRP

```text
Utiliser ensemble
      ↓
Regrouper ensemble
```

Mais :

```text
Ne pas utiliser ensemble
      ↓
Séparer
```

---

# 24. Exemple complet TypeScript

Imaginons une plateforme de paiement :

```text
libs/payment/
```

Nous avons :

```ts
export interface PaymentProvider {
  pay(amount: number): Promise<PaymentResult>;
}
```

```ts
export interface PaymentResult {
  transactionId: string;
  status: 'success' | 'failed' | 'pending';
}
```

Puis :

```ts
export class ProcessPayment {
  constructor(
    private readonly provider: PaymentProvider
  ) {}

  execute(amount: number): Promise<PaymentResult> {
    return this.provider.pay(amount);
  }
}
```

Et différentes implémentations :

```ts
export class WavePayment
  implements PaymentProvider {

  async pay(amount: number): Promise<PaymentResult> {
    // API Wave
    return {
      transactionId: 'TX-001',
      status: 'success',
    };
  }
}
```

```ts
export class OrangeMoneyPayment
  implements PaymentProvider {

  async pay(amount: number): Promise<PaymentResult> {
    // API Orange Money
    return {
      transactionId: 'TX-002',
      status: 'success',
    };
  }
}
```

On peut ensuite avoir :

```text
payment/
├── domain/
│   ├── PaymentProvider.ts
│   └── PaymentResult.ts
│
├── application/
│   └── ProcessPayment.ts
│
└── infrastructure/
    ├── WavePayment.ts
    └── OrangeMoneyPayment.ts
```

Cette organisation respecte assez bien :

```text
CCP
↓
Les éléments qui changent pour les mêmes raisons sont regroupés.

CRP
↓
Les consommateurs ne dépendent pas d'éléments inutiles.

REP
↓
Le composant Payment constitue une unité cohérente
de réutilisation et de release.
```

---

# 25. Différence avec les autres principes SOLID

Il est important de ne pas confondre ces principes.

```text
SRP
│
├── Classe
│   └── Une raison de changer
│
CCP
│
├── Composant
│   └── Même raison de changer
│
ISP
│
├── Interface
│   └── Ne pas dépendre de méthodes inutiles
│
CRP
│
├── Composant
│   └── Ne pas dépendre de classes inutiles
│
OCP
│
└── Classe / module
    └── Ouvert à l'extension,
        fermé à la modification
```

---

# 26. La leçon principale à retenir

Le chapitre remet en question une vision trop simpliste de la cohésion.

La cohésion n'est pas simplement :

> « Une classe fait une seule chose. »

Au niveau des composants, il faut également réfléchir à :

```text
Réutilisation
      +
Release
      +
Raisons de changement
      +
Dépendances
```

Le texte conclut que le découpage approprié des composants est **dynamique** : ce qui est pertinent aujourd'hui peut ne plus l'être l'année prochaine, notamment lorsque le projet évolue d'une priorité de développabilité vers une priorité de réutilisabilité. 

## La formule à retenir

```text
REP
→ Regrouper ce qui est réutilisé et livré ensemble.

CCP
→ Regrouper ce qui change pour les mêmes raisons.

CRP
→ Séparer ce dont les consommateurs n'ont pas besoin.
```

Ou encore, de manière très pratique pour une architecture **Angular/Nx/TypeScript** :

> **CCP te dit quoi mettre ensemble parce que ça change ensemble.**
> **CRP te dit quoi séparer parce que ça n'est pas utilisé ensemble.**
> **REP te dit quoi maintenir ensemble parce que ça doit être réutilisé et versionné ensemble.**

C'est cette **tension entre développabilité, réutilisabilité et dépendances minimales** qui permet de construire une architecture de composants réellement évolutive.
