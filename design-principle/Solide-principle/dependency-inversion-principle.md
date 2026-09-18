# DIP — Dependency Inversion Principle

Le **Dependency Inversion Principle (DIP)** est le **D** des principes **SOLID**.

L'idée principale est :

> **Les modules de haut niveau ne doivent pas dépendre directement des modules de bas niveau. Les deux doivent dépendre d'abstractions.**

En pratique :

```text
❌ High-Level → Concrete Implementation

✅ High-Level → Abstraction ← Concrete Implementation
```

---

## 1. Qu'est-ce qu'une dépendance ?

Une dépendance signifie qu'une classe utilise une autre classe.

Par exemple :

```ts
class OrderService {
  private emailService = new GmailService();

  sendConfirmation(): void {
    this.emailService.send();
  }
}
```

Ici, `OrderService` dépend directement de `GmailService`.

```text
OrderService
     │
     ↓
GmailService
```

Le problème est que `OrderService` connaît directement Gmail.

---

# 2. Pourquoi cette dépendance est problématique ?

Supposons que demain on décide de remplacer Gmail par SendGrid.

On crée :

```ts
class SendGridService {
  send(): void {
    console.log("Sending email with SendGrid");
  }
}
```

Il faut modifier `OrderService` :

```ts
class OrderService {
  private emailService = new SendGridService();

  sendConfirmation(): void {
    this.emailService.send();
  }
}
```

On a donc :

```text
Avant :

OrderService
     ↓
GmailService
```

Puis :

```text
Après :

OrderService
     ↓
SendGridService
```

La logique métier doit être modifiée à chaque changement de fournisseur.

---

# 3. La solution : créer une abstraction

On crée une interface :

```ts
interface EmailService {
  send(): void;
}
```

Puis Gmail implémente cette interface :

```ts
class GmailService implements EmailService {
  send(): void {
    console.log("Sending email with Gmail");
  }
}
```

SendGrid peut également l'implémenter :

```ts
class SendGridService implements EmailService {
  send(): void {
    console.log("Sending email with SendGrid");
  }
}
```

On obtient :

```text
              EmailService
              (abstraction)
               ↑       ↑
               │       │
               │       │
       GmailService   SendGridService
```

---

# 4. Le service métier dépend de l'interface

Au lieu de faire :

```ts
class OrderService {
  private emailService = new GmailService();
}
```

on fait :

```ts
class OrderService {
  constructor(
    private emailService: EmailService
  ) {}

  sendConfirmation(): void {
    this.emailService.send();
  }
}
```

Architecture :

```text
OrderService
      │
      ↓
EmailService
      ↑
      │
 ┌────┴───────────────┐
 │                    │
GmailService     SendGridService
```

`OrderService` ne sait plus si l'email est envoyé avec :

* Gmail
* SendGrid
* Brevo
* Amazon SES
* etc.

Il connaît uniquement :

```ts
EmailService
```

C'est le cœur du **DIP**.

---

# 5. Pourquoi "Dependency Inversion" ?

## Avant

```text
Business Logic
      │
      ↓
Concrete Implementation
```

Exemple :

```text
OrderService
      ↓
GmailService
```

La logique métier dépend directement du détail technique.

---

## Après

```text
             Abstraction
             ↑        ↑
             │        │
             │        │
        Business    Concrete
```

Exemple :

```text
             EmailService
             ↑        ↑
             │        │
      OrderService   GmailService
```

On peut résumer le DIP ainsi :

```text
❌ Business → Concrete
```

devient :

```text
✅ Business → Abstraction ← Concrete
```

---

# 6. High-Level et Low-Level

Le livre parle de **High-Level** et **Low-Level**.

## High-Level

Le **High-Level** représente les règles métier de l'application.

Exemples :

```text
OrderService
ClientService
PaymentService
CheckoutService
CreateOrder
ProcessPayment
```

Ce sont les fonctionnalités importantes de l'application.

---

## Low-Level

Le **Low-Level** représente les détails techniques.

Exemples :

```text
MongoDB
PostgreSQL
Gmail
SendGrid
Stripe
Sycapay
Redis
HTTP
File System
```

Ces éléments permettent de réaliser les règles métier.

---

# 7. Exemple avec un système de paiement

Imaginons ton application avec Sycapay.

## ❌ Mauvaise approche

```ts
class OrderService {

  async pay(amount: number): Promise<void> {
    const sycapay = new SycapayService();

    await sycapay.pay(amount);
  }

}
```

Architecture :

```text
OrderService
     ↓
SycapayService
```

`OrderService` connaît directement Sycapay.

---

# 8. Application du DIP

On crée une abstraction :

```ts
interface PaymentGateway {
  pay(amount: number): Promise<void>;
}
```

Sycapay :

```ts
class SycapayService implements PaymentGateway {

  async pay(amount: number): Promise<void> {
    console.log(`Payment with Sycapay: ${amount}`);
  }

}
```

Orange Money :

```ts
class OrangeMoneyService implements PaymentGateway {

  async pay(amount: number): Promise<void> {
    console.log(`Payment with Orange Money: ${amount}`);
  }

}
```

Wave :

```ts
class WaveService implements PaymentGateway {

  async pay(amount: number): Promise<void> {
    console.log(`Payment with Wave: ${amount}`);
  }

}
```

Notre service métier devient :

```ts
class OrderService {

  constructor(
    private paymentGateway: PaymentGateway
  ) {}

  async checkout(amount: number): Promise<void> {
    await this.paymentGateway.pay(amount);
  }

}
```

Architecture :

```text
                    PaymentGateway
                    (abstraction)
                    ↑     ↑     ↑
                    │     │     │
                Sycapay  Orange  Wave
                    │     │     │
                    └─────┴─────┘

OrderService
     │
     ↓
PaymentGateway
```

`OrderService` ne dépend d'aucun fournisseur spécifique.

---

# 9. Le principal avantage

Aujourd'hui tu utilises :

```text
Sycapay
```

Demain tu veux utiliser :

```text
Orange Money
```

Avec le DIP, tu n'as pas besoin de modifier `OrderService`.

`OrderService` utilise toujours :

```ts
PaymentGateway
```

Seule l'implémentation change.

---

# 10. Les dépendances concrètes et "volatile"

Le livre insiste sur le mot **volatile**.

Dans ce contexte, quelque chose de volatile est quelque chose qui peut changer fréquemment.

Par exemple :

```text
Sycapay
Stripe
Gmail
SendGrid
MongoDB
PostgreSQL
API externe
Framework
```

On veut éviter :

```text
Business Logic
      ↓
Volatile Concrete Implementation
```

On préfère :

```text
Business Logic
      ↓
Stable Abstraction
      ↑
      │
Volatile Implementation
```

---

# 11. Stable vs Volatile

| Élément      | Nature              | Exemple                   |
| ------------ | ------------------- | ------------------------- |
| Règle métier | Stable              | `OrderService`            |
| Interface    | Relativement stable | `PaymentGateway`          |
| Gmail        | Volatile            | Service externe           |
| SendGrid     | Volatile            | Service externe           |
| MongoDB      | Volatile            | Base de données           |
| PostgreSQL   | Volatile            | Base de données           |
| Sycapay      | Volatile            | Fournisseur de paiement   |
| `String`     | Très stable         | Fonctionnalité du langage |

Le DIP ne signifie donc pas :

> "Il ne faut avoir aucune dépendance concrète."

Il signifie plutôt :

> **Il faut éviter de dépendre directement des éléments concrets et volatils.**

---

# 12. Pourquoi `String` n'est pas un problème ?

Le livre donne l'exemple de `String`.

Par exemple :

```ts
const name: string = "Patrice";
```

Nous dépendons ici du type `string`.

Mais ce n'est pas un problème important parce que `string` est très stable.

Il est très peu probable que le langage change complètement son fonctionnement.

Le DIP concerne surtout les dépendances qui peuvent évoluer fréquemment.

---

# 13. Pourquoi les abstractions sont importantes ?

Prenons :

```ts
interface PaymentGateway {
  pay(amount: number): Promise<void>;
}
```

Nous pouvons avoir :

```ts
class SycapayService implements PaymentGateway {

  async pay(amount: number): Promise<void> {
    // Implementation Sycapay
  }

}
```

Supposons que nous modifions l'implémentation interne de Sycapay :

```ts
class SycapayService implements PaymentGateway {

  async pay(amount: number): Promise<void> {
    // Nouvelle implementation
  }

}
```

L'interface peut rester exactement la même :

```ts
interface PaymentGateway {
  pay(amount: number): Promise<void>;
}
```

Les utilisateurs de `PaymentGateway` n'ont donc pas nécessairement besoin d'être modifiés.

---

# 14. Mais si l'interface change ?

Supposons que nous changions :

```ts
interface PaymentGateway {
  pay(amount: number): Promise<void>;
}
```

en :

```ts
interface PaymentGateway {
  pay(
    amount: number,
    currency: string
  ): Promise<void>;
}
```

Toutes les implémentations doivent maintenant s'adapter :

```text
PaymentGateway
      │
      │ modification
      ↓
 ┌────┴──────────────┐
 ↓                   ↓
Sycapay            Stripe
 ↓                   ↓
doit changer       doit changer
```

C'est pourquoi une bonne architecture cherche à créer des interfaces **stables et bien conçues**.

---

# 15. Première règle du DIP

## Ne pas dépendre des classes concrètes volatiles

### ❌ Mauvais

```ts
class OrderService {

  constructor(
    private payment: SycapayService
  ) {}

}
```

`OrderService` dépend directement de Sycapay.

### ✅ Mieux

```ts
interface PaymentGateway {
  pay(amount: number): Promise<void>;
}

class OrderService {

  constructor(
    private payment: PaymentGateway
  ) {}

}
```

---

# 16. Deuxième règle du DIP

## Ne pas hériter des classes concrètes volatiles

Éviter :

```ts
class MyPaymentService extends SycapayService {

}
```

Pourquoi ?

Parce que `MyPaymentService` dépend fortement de l'implémentation de `SycapayService`.

Préférer :

```ts
interface PaymentGateway {
  pay(amount: number): Promise<void>;
}

class MyPaymentService implements PaymentGateway {

  async pay(amount: number): Promise<void> {
    // Implementation
  }

}
```

---

# 17. Troisième règle du DIP

## Éviter de surcharger des fonctions concrètes

Supposons :

```ts
class BaseService {

  send(): void {
    console.log("Sending...");
  }

}
```

Puis :

```ts
class MyService extends BaseService {

  send(): void {
    console.log("Different implementation");
  }

}
```

`MyService` dépend toujours de `BaseService`.

Une approche plus flexible :

```ts
interface Sender {
  send(): void;
}
```

Puis :

```ts
class EmailSender implements Sender {

  send(): void {
    console.log("Email");
  }

}
```

Et :

```ts
class SmsSender implements Sender {

  send(): void {
    console.log("SMS");
  }

}
```

---

# 18. Quatrième règle du DIP

## Ne pas mentionner inutilement les éléments concrets et volatils

Par exemple :

```ts
class OrderService {

  constructor(
    private stripe: StripeService
  ) {}

}
```

Ici, `OrderService` connaît directement Stripe.

Avec DIP :

```ts
class OrderService {

  constructor(
    private paymentGateway: PaymentGateway
  ) {}

}
```

Maintenant `OrderService` ne sait pas si le paiement utilise :

```text
Stripe
Sycapay
Wave
Orange Money
MTN Money
```

Il connaît uniquement :

```text
PaymentGateway
```

---

# 19. Le problème de `new`

Même avec une interface, ceci crée toujours une dépendance concrète :

```ts
class OrderService {

  private payment = new SycapayService();

}
```

Pourquoi ?

Parce que nous avons écrit :

```ts
new SycapayService()
```

Donc :

```text
OrderService
     ↓
new SycapayService()
```

Le code métier connaît toujours Sycapay.

C'est pourquoi le livre introduit le concept de **Factory**.

---

# 20. Abstract Factory

On peut créer une abstraction pour la création :

```ts
interface PaymentGatewayFactory {
  create(): PaymentGateway;
}
```

Puis :

```ts
class SycapayFactory implements PaymentGatewayFactory {

  create(): PaymentGateway {
    return new SycapayService();
  }

}
```

L'application peut maintenant faire :

```ts
const paymentGateway = factory.create();
```

au lieu de :

```ts
const paymentGateway = new SycapayService();
```

La création de l'objet concret est déplacée dans la partie technique.

---

# 21. Pourquoi utiliser une Factory ?

Parce que :

```ts
new SycapayService();
```

est une dépendance directe vers une classe concrète.

Alors que :

```ts
factory.create();
```

permet au code métier de ne connaître qu'une abstraction.

Architecture :

```text
Application
     │
     ↓
PaymentGatewayFactory
     │
     ↓
PaymentGateway
     ↑
     │
SycapayService
```

---

# 22. DIP et Dependency Injection

Le **DIP** et la **Dependency Injection** sont liés, mais ce ne sont pas exactement la même chose.

### DIP

C'est un **principe de conception** :

> Les modules importants doivent dépendre d'abstractions plutôt que de détails concrets.

### Dependency Injection

C'est une **technique** permettant d'appliquer ce principe.

Exemple :

```ts
class OrderService {

  constructor(
    private paymentGateway: PaymentGateway
  ) {}

}
```

La dépendance est fournie depuis l'extérieur :

```ts
const payment = new SycapayService();

const orderService = new OrderService(payment);
```

Ou :

```ts
const payment = new OrangeMoneyService();

const orderService = new OrderService(payment);
```

`OrderService` reste identique.

---

# 23. DIP et les tests unitaires

Le DIP facilite énormément les tests.

On définit :

```ts
interface PaymentGateway {
  pay(amount: number): Promise<void>;
}
```

Implémentation réelle :

```ts
class SycapayService implements PaymentGateway {

  async pay(amount: number): Promise<void> {
    // Appel réel à Sycapay
  }

}
```

Pour les tests, on peut créer une fausse implémentation :

```ts
class FakePaymentGateway implements PaymentGateway {

  async pay(amount: number): Promise<void> {
    console.log(`Fake payment: ${amount}`);
  }

}
```

Puis :

```ts
const fakePayment = new FakePaymentGateway();

const orderService = new OrderService(fakePayment);

await orderService.checkout(10000);
```

Le test n'a pas besoin :

```text
❌ d'une vraie API Sycapay
❌ d'une connexion Internet
❌ d'une vraie transaction
❌ d'un compte bancaire
```

---

# 24. Exemple avec Repository et MongoDB

C'est particulièrement important dans une application NestJS.

## ❌ Sans DIP

```ts
class ClientService {

  constructor(
    private clientRepository: MongoClientRepository
  ) {}

  async create() {
    return this.clientRepository.create();
  }

}
```

Architecture :

```text
ClientService
     ↓
MongoClientRepository
     ↓
MongoDB
```

Le service métier connaît MongoDB.

---

# 25. Avec DIP

On définit une abstraction :

```ts
interface ClientRepository {
  create(): Promise<void>;
  findById(id: string): Promise<void>;
}
```

Puis l'implémentation MongoDB :

```ts
class MongoClientRepository implements ClientRepository {

  async create(): Promise<void> {
    // MongoDB
  }

  async findById(id: string): Promise<void> {
    // MongoDB
  }

}
```

Le service métier dépend uniquement de l'interface :

```ts
class ClientService {

  constructor(
    private clientRepository: ClientRepository
  ) {}

  async create() {
    return this.clientRepository.create();
  }

}
```

Architecture :

```text
                 ClientRepository
                    (interface)
                    ↑        ↑
                    │        │
                    │        │
             ClientService   MongoClientRepository
              Business            MongoDB
```

---

# 26. Pourquoi cette architecture est meilleure ?

Aujourd'hui :

```text
MongoDB
```

Demain :

```text
PostgreSQL
```

On peut créer :

```ts
class PostgresClientRepository implements ClientRepository {

  async create(): Promise<void> {
    // PostgreSQL
  }

  async findById(id: string): Promise<void> {
    // PostgreSQL
  }

}
```

Le `ClientService` ne change pas.

Il utilise toujours :

```ts
ClientRepository
```

C'est exactement l'objectif du DIP.

---

# 27. La frontière architecturale

Le livre présente une idée très importante : la **frontière entre abstraction et détails**.

```text
┌──────────────────────────────────────────┐
│               BUSINESS                   │
│                                          │
│  OrderService                            │
│  ClientService                           │
│  PaymentService                          │
│  ClientRepository                        │
│  PaymentGateway                          │
│                                          │
└────────────────────▲─────────────────────┘
                     │
                     │ Abstractions
                     │
─────────────────────┼──────────────────────
                     │
                     │
┌────────────────────┴─────────────────────┐
│                DETAILS                   │
│                                          │
│  MongoDB                                 │
│  PostgreSQL                              │
│  Sycapay                                 │
│  Stripe                                  │
│  Gmail                                   │
│  SendGrid                                │
│  HTTP                                    │
│                                          │
└──────────────────────────────────────────┘
```

Les détails techniques sont isolés à l'extérieur.

Les règles métier restent protégées.

---

# 28. Flux de contrôle vs dépendances

Le livre explique un point subtil.

Supposons :

```text
OrderService → PaymentGateway
```

Lors de l'exécution :

```text
OrderService
     ↓
PaymentGateway
     ↓
SycapayService
```

Le **flux d'exécution** va vers l'implémentation concrète.

Mais les **dépendances du code** vont dans l'autre direction :

```text
SycapayService
      ↓
PaymentGateway
```

On peut représenter cela ainsi :

```text
                 PaymentGateway
                /               \
               ↑                 ↑
               │                 │
      OrderService         SycapayService
```

Le code métier définit le contrat à travers l'abstraction.

C'est ce mécanisme qui donne le nom :

> **Dependency Inversion**

---

# 29. DIP dans Clean Architecture

C'est pour cette raison que le DIP est très important dans **Clean Architecture**.

Une architecture simplifiée :

```text
┌─────────────────────────────────────┐
│          Infrastructure             │
│                                     │
│ MongoDB                             │
│ Sycapay                             │
│ SendGrid                            │
│ HTTP                                │
└──────────────────┬──────────────────┘
                   │
                   ↓
┌─────────────────────────────────────┐
│           Interfaces                │
│                                     │
│ ClientRepository                    │
│ PaymentGateway                      │
│ EmailService                        │
└──────────────────┬──────────────────┘
                   ↑
                   │
┌──────────────────┴──────────────────┐
│            Application              │
│                                     │
│ CreateClient                        │
│ CreateOrder                         │
│ ProcessPayment                      │
└──────────────────┬──────────────────┘
                   ↑
                   │
┌──────────────────┴──────────────────┐
│              Domain                 │
│                                     │
│ Entities                            │
│ Business Rules                      │
└─────────────────────────────────────┘
```

L'objectif est de garder les règles métier indépendantes des détails techniques.

---

# 30. DIP vs ISP

Il ne faut pas confondre **DIP** et **ISP**.

## ISP — Interface Segregation Principle

L'ISP dit :

> **Un client ne doit pas être obligé de dépendre de méthodes qu'il n'utilise pas.**

Par exemple :

```ts
interface PaymentService {
  pay(): void;
  refund(): void;
  sendEmail(): void;
  generateReport(): void;
}
```

On peut séparer :

```ts
interface PaymentProcessor {
  pay(): void;
}

interface RefundProcessor {
  refund(): void;
}
```

---

## DIP — Dependency Inversion Principle

Le DIP dit :

> **Le code métier ne doit pas dépendre directement des implémentations concrètes. Il doit dépendre d'abstractions.**

Exemple :

```ts
interface PaymentGateway {
  pay(): void;
}

class OrderService {

  constructor(
    private payment: PaymentGateway
  ) {}

}
```

---

# 31. Résumé visuel

## ❌ Sans DIP

```text
┌───────────────────┐
│   Business Logic  │
│                   │
│   OrderService    │
└─────────┬─────────┘
          │
          ↓
┌───────────────────┐
│ SycapayService    │
│ Concrete          │
└───────────────────┘
```

La logique métier dépend du détail.

---

## ✅ Avec DIP

```text
                 ┌───────────────────┐
                 │ PaymentGateway    │
                 │   Abstraction     │
                 └────────▲──────────┘
                          │
              ┌───────────┴───────────┐
              │                       │
┌─────────────┴────────┐  ┌──────────┴─────────┐
│   OrderService       │  │  SycapayService    │
│   Business Logic     │  │  Concrete          │
└──────────────────────┘  └────────────────────┘
```

Les deux dépendent de l'abstraction.

---

# 32. Ce qu'il faut absolument retenir

Le DIP peut être résumé par cette formule :

```text
❌ Business → Concrete
```

contre :

```text
✅ Business → Abstraction ← Concrete
```

### Exemple final

```ts
// Abstraction
interface PaymentGateway {
  pay(amount: number): Promise<void>;
}

// Business Logic
class OrderService {

  constructor(
    private paymentGateway: PaymentGateway
  ) {}

  async checkout(amount: number): Promise<void> {
    await this.paymentGateway.pay(amount);
  }

}

// Concrete implementation
class SycapayService implements PaymentGateway {

  async pay(amount: number): Promise<void> {
    // Sycapay implementation
  }

}
```

`OrderService` ne connaît pas Sycapay.

Il connaît seulement :

```ts
PaymentGateway
```

---

# 🎯 La phrase à retenir

> **Les règles métier ne doivent pas dépendre des détails techniques. Ce sont les détails techniques qui doivent dépendre des abstractions définies autour des règles métier.**

En très simple :

```text
          WHAT
           │
           ↓
     Abstraction
           ↑
           │
          HOW
           │
           ↓
    Implementation
```

* **WHAT** → ce dont l'application a besoin.
* **HOW** → comment cela est techniquement réalisé.
* **DIP** → le code métier doit principalement connaître le **WHAT**, pas le **HOW**.
