# Open-Closed Principle (OCP)

## 1. Qu'est-ce que l'Open‑Closed Principle ?

Le principe a été formulé par Bertrand Meyer en 1988.

Définition :

> Un élément logiciel doit être ouvert à l'extension, mais fermé à la modification.

En anglais : *Open for extension, closed for modification.*

Cela signifie qu'on doit pouvoir ajouter de nouveaux comportements à une application sans devoir modifier massivement le code existant qui fonctionne déjà.

---

## 2. Que signifie « ouvert à l'extension » ?

« Ouvert à l'extension » signifie que le système doit pouvoir accueillir de nouvelles fonctionnalités.

Exemple (TypeScript) :

```ts
interface PaymentMethod {
  pay(amount: number): void;
}

class CreditCardPayment implements PaymentMethod {
  pay(amount: number): void {
    console.log("Paiement par carte");
  }
}

class MobileMoneyPayment implements PaymentMethod {
  pay(amount: number): void {
    console.log("Paiement Mobile Money");
  }
}
```

On a étendu le système en ajoutant `MobileMoneyPayment` sans toucher aux autres classes.

---

## 3. Que signifie « fermé à la modification » ?

Ce n'est pas : « on ne doit jamais modifier le code ». C'est impossible.

Cela veut dire : lorsqu'on ajoute une nouvelle fonctionnalité, on devrait éviter de modifier les composants existants qui n'ont pas besoin de changer.

Exemple :

```ts
function processPayment(payment: PaymentMethod) {
  payment.pay(10000);
}

class WavePayment implements PaymentMethod {
  pay(amount: number): void {
    console.log("Paiement Wave");
  }
}
```

Si l'on ajoute `WavePayment`, on n'a pas besoin de modifier `processPayment()` — on ajoute simplement une nouvelle implémentation.

OUVERT À L'EXTENSION + FERMÉ À LA MODIFICATION

---

## 4. Mauvaise architecture vs bonne architecture

Mauvaise approche (anti‑pattern) :

```ts
function processPayment(type: string, amount: number) {
  if (type === "card") {
    // paiement carte
  }

  if (type === "mobile-money") {
    // paiement Mobile Money
  }
}
```

À chaque nouveau moyen de paiement (Wave, PayPal, Apple Pay) il faut modifier `processPayment()` → la fonction grossit et devient fragile.

Bonne approche avec polymorphisme :

```ts
interface PaymentMethod {
  pay(amount: number): void;
}

class CardPayment implements PaymentMethod { /* ... */ }
class MobileMoneyPayment implements PaymentMethod { /* ... */ }
class WavePayment implements PaymentMethod { /* ... */ }
class PaypalPayment implements PaymentMethod { /* ... */ }

function processPayment(payment: PaymentMethod, amount: number) {
  payment.pay(amount);
}
```

Schéma conceptuel :

- PaymentMethod
  - Card
  - Mobile Money
  - Wave
  - Paypal

`processPayment()` ne change pas quand on ajoute de nouvelles implémentations.

---

## 5. Le problème que veut résoudre l'OCP

Si une petite modification des exigences oblige à modifier énormément de code, l'architecture est mauvaise.

Exemple : on a aujourd'hui une présentation Web, et le client veut aussi imprimer ces informations. Si l'architecture lie calcul et présentation, beaucoup de code sera modifié. Une bonne architecture permet d'ajouter une nouvelle présentation (print) avec peu ou pas de changements au cœur métier.

Idéalement :

- Nouveau besoin → Nouveau code → Très peu de modifications → (Idéalement) aucune modification du cœur métier

---

## 6. Le premier principe utilisé : SRP

Le Single Responsibility Principle (SRP) : séparer les choses qui changent pour des raisons différentes.

Exemple de séparation :

- Calcul des données (FinancialData)
- Présentation Web
- Présentation papier

Chaque responsabilité est séparée pour limiter l'impact des changements.

---

## 7. Puis vient le Dependency Inversion Principle (DIP)

Après avoir séparé les responsabilités, il faut contrôler les dépendances.

On évite que la logique métier dépende directement d'une technologie (MongoDB, PostgreSQL). On préfère que la logique dépende d'une abstraction.

Exemple :

```ts
interface UserRepository {
  save(user: User): Promise<void>;
}
```

Le métier dépend de `UserRepository` et non d'une implémentation concrète (`MongoUserRepository`).

---

## 8. Le concept essentiel : protéger le métier

Hiérarchie (exemple) :

- BUSINESS RULES
  - Interactor
    - Controller
      - Presenter
        - View
          - Database

Plus on monte, plus on est proche des règles métier. Les composants importants doivent être protégés contre les changements des composants moins importants.

---

## 9. Pourquoi l'Interactor est très protégé ?

L'Interactor contient les règles métier principales.

```ts
class CreateOrderInteractor {
  execute(order: Order): void {
    // règles métier
  }
}
```

Il ne devrait pas être impacté parce qu'on change Angular, React, MongoDB, l'UI, ou le format d'impression. Les détails doivent dépendre du métier, et non l'inverse.

---

## 10. La phrase clé du chapitre

Si le composant A doit être protégé contre les changements du composant B, alors B doit dépendre de A.

Exemple :

- A = Business Rules
- B = Database

On veut que B dépende de A (via une abstraction), pas l'inverse.

---

## 11. Pourquoi inverser la dépendance ?

Mauvais exemple (le métier connaît MongoDB directement) :

```ts
class CreateUserUseCase {
  private repository = new MongoUserRepository();

  execute(user: User) {
    this.repository.save(user);
  }
}
```

Si on change de base de données, `CreateUserUseCase` doit changer — c'est mauvais.

Meilleure approche (abstraction + injection) :

```ts
interface UserRepository {
  save(user: User): Promise<void>;
}

class CreateUserUseCase {
  constructor(private readonly repository: UserRepository) {}

  execute(user: User) {
    return this.repository.save(user);
  }
}

class MongoUserRepository implements UserRepository {
  save(user: User) {
    // MongoDB
  }
}

class PostgresUserRepository implements UserRepository {
  save(user: User) {
    // PostgreSQL
  }
}
```

Le Use Case ne change pas quand on ajoute une nouvelle implémentation. C'est une application concrète de DIP + OCP.

---

## 12. Le rôle des interfaces

Exemples d'interfaces :

```ts
interface FinancialDataGateway {
  getFinancialData(): FinancialData;
}
```

L'Interactor utilise `FinancialDataGateway` sans savoir si les données viennent de MongoDB, PostgreSQL, REST API, GraphQL, fichier, ou mock.

---

## 13. Le principe de protection

Architecture typique :

```
Interactor (Business Rules)
  ↑
Controller    Database
  ↑
Presenter
  ↑
View
```

Le changement se fait généralement vers l'extérieur (View, Database, présentation). Le métier ne devrait pas changer pour ces modifications externes. En revanche, une nouvelle règle métier modifie naturellement l'Interactor.

---

## 14. Une hiérarchie de protection

Plus protégé → Interactor → Controller → Presenter → View → moins protégé.

Les règles métier (Interactor) sont généralement plus stables que les détails d'interface (View).

---

## 15. « Directional Control » : contrôler la direction

On doit contrôler la direction des dépendances.

On ne veut pas : `Interactor → Database`

On veut : `Database → Interactor` via une abstraction (`FinancialDataGateway`).

---

## 16. « Information Hiding »

Cacher les détails internes que les autres composants n'ont pas besoin de connaître.

Le Controller ne doit pas connaître toute la structure interne du Use Case. On expose seulement une abstraction :

```ts
interface FinancialReportRequester {
  generateReport(request: ReportRequest): ReportResponse;
}
```

Le Controller connaît seulement `FinancialReportRequester`.

---

## 17. Pourquoi c'est important ?

Si l'Interactor contient de nombreuses entités et règles (FinancialEntity, FinancialCalculation, TaxRules, ReportRules, ValidationRules...), et que le Controller y dépend directement, une modification interne peut casser le Controller. Avec une abstraction entre Controller et Interactor, le Controller est protégé.

---

## 18. Exemple complet en NestJS

Exemple :

```ts
interface OrderRepository {
  save(order: Order): Promise<void>;
}

class CreateOrderUseCase {
  constructor(private readonly repository: OrderRepository) {}

  async execute(order: Order): Promise<void> {
    // règles métier
    await this.repository.save(order);
  }
}

class MongoOrderRepository implements OrderRepository {
  async save(order: Order): Promise<void> {
    // sauvegarde MongoDB
  }
}

class PostgresOrderRepository implements OrderRepository {
  async save(order: Order): Promise<void> {
    // sauvegarde PostgreSQL
  }
}
```

Le Use Case (`CreateOrderUseCase`) n'a pas besoin de changer quand on change la persistence.

---

## 19. Et c'est là que l'OCP devient architectural

À petite échelle, l'OCP se manifeste par une interface et ses implémentations.

À l'échelle d'une application, on construit des frontières architecturales qui permettent d'ajouter ou remplacer des détails sans toucher au cœur métier.

---

## 20. OCP + SRP + DIP

Ces trois principes fonctionnent ensemble :

- SRP : séparer ce qui change pour des raisons différentes (Métier ≠ Présentation ≠ Persistence)
- DIP : faire dépendre le métier d'abstractions plutôt que de détails
- OCP : pouvoir étendre sans modifier massivement l'existant

Ensemble, ils permettent de séparer responsabilités, inverser les dépendances et étendre sans casser l'existant.

---

## 21. Exemple concret : ton application (cotation automobile)

Structure possible :

- Cotation
  - Calcul de la prime
  - Sauvegarde
  - Affichage Web
  - Génération PDF

Mauvaise architecture (tout dans un service) :

```ts
class CotationService {
  calculatePremium() {}
  saveToMongo() {}
  generateHtml() {}
  generatePdf() {}
}
```

Tout changement (base, export, format PDF) force des modifications dans `CotationService` → couplage élevé.

---

## 22. Architecture plus conforme à l'OCP

Séparer en :

- `CotationUseCase`
  - `CotationRepository` ← MongoDB | PostgreSQL
  - `CotationPresenter` ← Web | PDF | Excel

Ajouter Excel → ajouter `ExcelPresenter` sans toucher au cœur métier. Changer MongoDB → Postgres → remplacer l'implémentation de `CotationRepository`.

---

## 23. La vraie philosophie de l'OCP

Ne pas comprendre OCP comme : « Je ne dois jamais modifier mon code. »

Bonne compréhension :

- Concevoir le système pour que les changements prévisibles puissent être ajoutés sans que leurs effets se propagent partout.

---

## 24. Comment savoir si ton architecture respecte l'OCP ?

Quand tu reçois une nouvelle demande, pose‑toi :

1. Quel composant doit réellement changer ?
2. Est‑ce que cette modification force d'autres composants à changer ?
3. Pourquoi ces composants doivent-ils changer ?
4. Peut‑on ajouter une nouvelle implémentation au lieu de modifier l'ancienne ?
5. Le cœur métier dépend‑il d'un détail technique ?

---

## 25. Exemple de diagnostic

Demande : « Ajouter le paiement Wave. »

- Si tu dois modifier `PaymentService`, `PaymentController`, `OrderService`, `DatabaseService` → ton architecture est probablement trop couplée.
- Si tu peux créer `WavePayment implements PaymentMethod` et brancher la nouvelle implémentation sans toucher le reste → tu es proche de l'OCP.

---

## 26. Résumé à retenir

- OCP : Ouvert à l'extension + Fermé à la modification.
- SRP : Séparer les responsabilités.
- DIP : Organiser les dépendances via des abstractions.
- Objectif : Protéger le cœur métier et isoler les détails techniques (UI, DB, frameworks, API).

Phrase clé : une bonne architecture permet d'ajouter de nouvelles fonctionnalités avec un minimum de modifications du code existant, en particulier du code métier.

Détails → Database / UI / Framework / HTTP → MÉTIER (le plus protégé)

---

Souhaitez‑vous que j'enregistre cette version Markdown dans open-close.md ?