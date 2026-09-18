# Interface Segregation Principle (ISP)

L’**Interface Segregation Principle (ISP)** est le principe **I** des principes **SOLID**.

> **Un client ne devrait pas être obligé de dépendre de méthodes qu’il n’utilise pas.**

Autrement dit :

> **Il vaut mieux avoir plusieurs petites interfaces spécialisées qu’une grosse interface générale.**

---

## 1. Le problème présenté dans le livre

Imaginons une classe `OPS` qui possède trois opérations :

```ts
class OPS {
  op1(): void {
    console.log("Operation 1");
  }

  op2(): void {
    console.log("Operation 2");
  }

  op3(): void {
    console.log("Operation 3");
  }
}
```

Nous avons trois utilisateurs :

```text
User1 → utilise seulement op1
User2 → utilise seulement op2
User3 → utilise seulement op3
```

On pourrait écrire :

```ts
class User1 {
  constructor(private ops: OPS) {}

  execute(): void {
    this.ops.op1();
  }
}
```

`User1` utilise uniquement :

```ts
ops.op1();
```

Mais `User1` dépend de toute la classe `OPS` :

```text
User1
  ↓
 OPS
 ├── op1()
 ├── op2()
 └── op3()
```

C’est là que se trouve le problème.

---

# 2. Pourquoi cette dépendance est problématique ?

Supposons que `op2()` soit modifiée :

```ts
class OPS {
  op1(): void {
    console.log("Operation 1");
  }

  op2(): void {
    console.log("Nouvelle implémentation");
  }

  op3(): void {
    console.log("Operation 3");
  }
}
```

`User1` n'utilise pas `op2()`.

Pourtant, il dépend de la classe `OPS` qui contient `op2()`.

Donc :

```text
User1
  ↓
 OPS
  ↓
 op2()
```

Il existe une dépendance indirecte envers quelque chose dont `User1` n'a pas besoin.

Dans des systèmes utilisant des dépendances de code source, cela peut provoquer des recompilations, redéploiements ou effets de bord inutiles.

---

# 3. La solution : séparer les interfaces

Au lieu d'avoir une grosse interface :

```ts
interface OPS {
  op1(): void;
  op2(): void;
  op3(): void;
}
```

On crée plusieurs interfaces spécialisées :

```ts
interface U1Ops {
  op1(): void;
}

interface U2Ops {
  op2(): void;
}

interface U3Ops {
  op3(): void;
}
```

On obtient alors :

```text
User1 → U1Ops → op1()

User2 → U2Ops → op2()

User3 → U3Ops → op3()
```

Chaque utilisateur dépend uniquement de ce dont il a besoin.

---

# 4. Exemple TypeScript complet

Nous pouvons avoir une classe qui implémente toutes ces interfaces :

```ts
class OPS implements U1Ops, U2Ops, U3Ops {
  op1(): void {
    console.log("Operation 1");
  }

  op2(): void {
    console.log("Operation 2");
  }

  op3(): void {
    console.log("Operation 3");
  }
}
```

Mais `User1` ne dépend plus directement de `OPS`.

Il dépend seulement de `U1Ops` :

```ts
class User1 {
  constructor(private operations: U1Ops) {}

  execute(): void {
    this.operations.op1();
  }
}
```

`User2` :

```ts
class User2 {
  constructor(private operations: U2Ops) {}

  execute(): void {
    this.operations.op2();
  }
}
```

Et `User3` :

```ts
class User3 {
  constructor(private operations: U3Ops) {}

  execute(): void {
    this.operations.op3();
  }
}
```

---

# 5. Avant et après l'ISP

## ❌ Avant : grosse interface

```text
             ┌───────────────┐
             │      OPS      │
             ├───────────────┤
             │    op1()      │
             │    op2()      │
             │    op3()      │
             └───────────────┘
                ↑    ↑    ↑
                │    │    │
              User1 User2 User3
```

Chaque utilisateur dépend de la même grosse classe.

---

## ✅ Après : interfaces séparées

```text
User1 ─────→ U1Ops ─────→ op1()

User2 ─────→ U2Ops ─────→ op2()

User3 ─────→ U3Ops ─────→ op3()
```

Chaque utilisateur ne voit que les opérations qui l'intéressent.

---

# 6. ISP et les langages de programmation

Le livre explique ensuite que le problème peut être différent selon le type de langage.

## Langages statiquement typés

Exemples :

* Java
* C#
* TypeScript

Dans ces langages, les types et interfaces sont explicitement déclarés.

Par exemple :

```ts
class User1 {
  constructor(private ops: OPS) {}
}
```

Ici, `User1` déclare explicitement :

> "Je dépends de `OPS`."

Donc `User1` est lié à la définition de `OPS`.

---

# 7. Langages dynamiquement typés

Exemples :

* Python
* Ruby
* JavaScript

En Python, on peut écrire :

```python
class User1:
    def __init__(self, ops):
        self.ops = ops

    def execute(self):
        self.ops.op1()
```

On ne déclare pas explicitement :

```text
ops doit être de type OPS
```

Python vérifie essentiellement au moment de l'exécution si l'objet possède `op1()`.

C'est notamment lié au **duck typing**.

> Si un objet possède le comportement dont j'ai besoin, je peux l'utiliser.

---

# 8. Est-ce que l'ISP concerne seulement les langages ?

**Non.**

C'est une partie très importante du texte.

Le livre explique que l'ISP n'est pas seulement un problème de langage.

Le problème plus général est :

> **Dépendre de quelque chose qui contient plus de choses que ce dont on a besoin est dangereux.**

Cela concerne également l'architecture logicielle.

---

# 9. ISP au niveau architectural

Imaginons une application :

```text
Application S
      ↓
Framework F
      ↓
Database D
```

Donc :

```text
S → F → D
```

Supposons que le framework `F` utilise seulement certaines fonctionnalités de `D`.

Par exemple :

```text
Database D
│
├── Users
├── Orders
├── Payments
├── Reports
├── Analytics
└── Backup
```

Mais `F` utilise uniquement :

```text
Users
Orders
```

Pourtant :

```text
Application S
      ↓
Framework F
      ↓
Database D
```

L'application est indirectement dépendante de la base de données entière.

---

# 10. Pourquoi cela peut être dangereux ?

Supposons qu'une fonctionnalité d'`Analytics` soit modifiée dans `D`.

```text
Database D
│
├── Users        ← utilisé
├── Orders       ← utilisé
├── Payments
├── Reports
├── Analytics    ← modification
└── Backup
```

Même si `F` n'utilise pas `Analytics`, une modification importante de `D` peut avoir des conséquences sur `F`, puis sur `S`.

On peut avoir :

```text
D
↓
Problème dans une fonctionnalité inutile à F
↓
F affecté
↓
S affectée
```

C'est le même problème que précédemment, mais à une échelle architecturale.

---

# 11. Le concept de "baggage"

Le livre utilise le terme **baggage**.

On peut le comprendre comme :

> **Des fonctionnalités supplémentaires dont tu ne veux pas, mais dont tu deviens quand même dépendant.**

Imagine que tu veux voyager avec seulement :

```text
👕 T-shirt
👖 Pantalon
```

Mais on t'oblige à transporter :

```text
🧳 Énorme valise
 ├── vêtements
 ├── chaussures
 ├── livres
 ├── ordinateur
 ├── matériel de sport
 └── autres objets
```

Tu transportes beaucoup plus que nécessaire.

En architecture logicielle, c'est similaire :

```text
Tu as besoin de :
    op1()

Mais tu dépends de :
    op1()
    op2()
    op3()
    op4()
    op5()
    ...
```

C'est ce que l'ISP cherche à éviter.

---

# 12. Exemple avec un système de paiement

Imaginons une grosse interface :

```ts
interface PaymentService {
  pay(): Promise<void>;
  refund(): Promise<void>;
  generateInvoice(): Promise<void>;
  sendEmail(): Promise<void>;
  generateReport(): Promise<void>;
  exportToExcel(): Promise<void>;
}
```

Maintenant, notre `CheckoutService` a uniquement besoin de :

```ts
pay()
```

Mais il dépend de toute l'interface :

```ts
class CheckoutService {
  constructor(
    private paymentService: PaymentService
  ) {}

  async checkout(): Promise<void> {
    await this.paymentService.pay();
  }
}
```

Le `CheckoutService` dépend donc de :

```text
PaymentService
├── pay()
├── refund()
├── generateInvoice()
├── sendEmail()
├── generateReport()
└── exportToExcel()
```

Alors qu'il utilise uniquement :

```text
pay()
```

---

# 13. Application de l'ISP

On peut séparer l'interface :

```ts
interface PaymentProcessor {
  pay(): Promise<void>;
}

interface RefundProcessor {
  refund(): Promise<void>;
}

interface InvoiceGenerator {
  generateInvoice(): Promise<void>;
}

interface EmailSender {
  sendEmail(): Promise<void>;
}
```

Maintenant :

```ts
class CheckoutService {
  constructor(
    private paymentProcessor: PaymentProcessor
  ) {}

  async checkout(): Promise<void> {
    await this.paymentProcessor.pay();
  }
}
```

Le `CheckoutService` dépend uniquement de :

```ts
PaymentProcessor
```

Il ne dépend pas de :

```text
RefundProcessor
InvoiceGenerator
EmailSender
```

C'est une meilleure séparation des responsabilités.

---

# 14. Exemple avec NestJS

Dans une application NestJS, on pourrait avoir une grosse interface :

```ts
interface ClientService {
  createClient(): Promise<void>;
  updateClient(): Promise<void>;
  deleteClient(): Promise<void>;
  resetPassword(): Promise<void>;
  sendEmail(): Promise<void>;
}
```

Mais imaginons qu'un composant ait uniquement besoin de créer un client.

On peut créer :

```ts
interface ClientCreator {
  createClient(): Promise<void>;
}
```

Puis :

```ts
class ClientController {
  constructor(
    private clientCreator: ClientCreator
  ) {}

  create() {
    return this.clientCreator.createClient();
  }
}
```

Le contrôleur dépend uniquement de l'opération dont il a besoin.

---

# 15. Attention : ISP ne signifie pas "une interface par méthode"

Il ne faut pas tomber dans l'excès.

L'ISP ne signifie pas qu'il faut forcément faire :

```ts
interface CreateClient {
  createClient(): Promise<void>;
}

interface UpdateClient {
  updateClient(): Promise<void>;
}

interface DeleteClient {
  deleteClient(): Promise<void>;
}

interface GetClient {
  getClient(): Promise<void>;
}
```

Ce serait parfois inutilement fragmenté.

Le principe est plutôt :

> **Regrouper les opérations qui sont réellement utilisées ensemble par un même client.**

Il faut donc regarder les besoins des utilisateurs de l'interface.

---

# 16. La règle pratique

Quand tu conçois une interface, pose-toi cette question :

> **"Est-ce que tous les utilisateurs de cette interface ont réellement besoin de toutes ses méthodes ?"**

Si la réponse est **non**, il peut être intéressant de séparer l'interface.

### ❌ Mauvais signe

```ts
interface BigService {
  methodA(): void;
  methodB(): void;
  methodC(): void;
  methodD(): void;
  methodE(): void;
}
```

Et :

```ts
class ClientA {
  constructor(private service: BigService) {}

  execute() {
    this.service.methodA();
  }
}
```

`ClientA` utilise 1 méthode sur 5.

---

### ✅ Approche ISP

```ts
interface ServiceA {
  methodA(): void;
}
```

Puis :

```ts
class ClientA {
  constructor(private service: ServiceA) {}

  execute() {
    this.service.methodA();
  }
}
```

Maintenant :

```text
ClientA
   ↓
ServiceA
   ↓
methodA()
```

La dépendance est beaucoup plus précise.

---

# 17. L'idée fondamentale du chapitre

Le texte peut être résumé par cette règle :

> **Ne force pas un client à dépendre de fonctionnalités qu'il n'utilise pas.**

Cela s'applique à plusieurs niveaux :

```text
                    ISP
                     │
        ┌────────────┴────────────┐
        ↓                         ↓
   Code source               Architecture
        │                         │
        ↓                         ↓
Petites interfaces       Petites dépendances
        │                         │
        └────────────┬────────────┘
                     ↓
       Moins de couplage inutile
```

---

# 18. ISP dans SOLID

ISP correspond au **I** de SOLID :

```text
S → Single Responsibility Principle
O → Open/Closed Principle
L → Liskov Substitution Principle
I → Interface Segregation Principle
D → Dependency Inversion Principle
```

Il est particulièrement lié au **Dependency Inversion Principle (DIP)**.

Par exemple :

```ts
interface PaymentProcessor {
  pay(): Promise<void>;
}

class CheckoutService {
  constructor(
    private payment: PaymentProcessor
  ) {}
}
```

`CheckoutService` dépend d'un petit contrat :

```ts
PaymentProcessor
```

et non d'une énorme classe contenant toutes les fonctionnalités possibles du système de paiement.

---

# 19. Résumé à retenir

| Concept               | Explication                                    |
| --------------------- | ---------------------------------------------- |
| **ISP**               | Interface Segregation Principle                |
| **Objectif**          | Éviter les dépendances inutiles                |
| **Mauvaise approche** | Une grosse interface avec beaucoup de méthodes |
| **Bonne approche**    | Plusieurs interfaces spécialisées              |
| **Bénéfice**          | Moins de couplage                              |
| **Résultat**          | Code plus flexible et plus facile à maintenir  |

### En une phrase

```text
❌ Ne dépends pas de tout quand tu as besoin de seulement une partie.

✅ Dépends uniquement de l'interface correspondant à ton besoin.
```

Ou, encore plus simplement :

> **"Depend on what you need, not on everything that exists."**
