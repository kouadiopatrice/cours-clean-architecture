# LSP — Liskov Substitution Principle

## 1. Introduction

En 1988, **Barbara Liskov** a proposé une définition permettant de déterminer lorsqu'un type peut être considéré comme un véritable sous-type d'un autre.

Son idée peut être résumée ainsi :

> Si un programme fonctionne avec un objet de type `T`, on doit pouvoir remplacer cet objet par un objet de type `S` sans modifier le comportement attendu du programme.

C'est ce que l'on appelle le **Liskov Substitution Principle (LSP)**, ou en français :

> **Principe de substitution de Liskov.**

### Définition simple

> **Un sous-type doit pouvoir remplacer son type parent sans casser le comportement attendu du programme.**

Autrement dit :

```text
Si S est un sous-type de T

Alors :

S doit pouvoir être utilisé partout où T est attendu
sans modifier le comportement correct du programme.
```

---

# 2. Comprendre le principe

Prenons une abstraction :

```text
        T
        │
        ├── S1
        ├── S2
        └── S3
```

Si une fonction attend `T` :

```ts
function process(value: T): void {
  // ...
}
```

elle doit pouvoir recevoir :

```ts
process(new S1());
process(new S2());
process(new S3());
```

sans avoir besoin de connaître le type concret utilisé.

Le code client ne devrait donc pas avoir besoin de faire :

```ts
if (value instanceof S1) {
  // traitement spécial
}

if (value instanceof S2) {
  // autre traitement
}
```

Si de nombreux cas particuliers sont nécessaires, cela peut indiquer que les implémentations ne respectent pas correctement le contrat de l'abstraction.

---

# 3. Exemple : License

Imaginons une classe abstraite `License` :

```ts
abstract class License {
  abstract calcFee(): number;
}
```

Nous avons deux sous-types :

```ts
class PersonalLicense extends License {
  calcFee(): number {
    return 100;
  }
}
```

```ts
class BusinessLicense extends License {
  calcFee(): number {
    return 500;
  }
}
```

Une application de facturation peut simplement dépendre de `License` :

```ts
function calculateBilling(license: License): number {
  return license.calcFee();
}
```

Elle peut recevoir :

```ts
calculateBilling(new PersonalLicense());
```

ou :

```ts
calculateBilling(new BusinessLicense());
```

Le code de `calculateBilling()` ne change pas.

## Pourquoi le LSP est respecté ?

Parce que les deux implémentations respectent le contrat défini par `License`.

```text
             License
                │
        ┌───────┴────────┐
        │                │
 PersonalLicense   BusinessLicense
        │                │
    calcFee()         calcFee()
```

L'application n'a pas besoin de connaître le type concret.

Elle demande simplement :

```ts
license.calcFee();
```

C'est exactement l'objectif du polymorphisme et de la substituabilité.

---

# 4. Le problème Square / Rectangle

L'exemple classique d'une violation du LSP est le problème du **Square / Rectangle**.

Mathématiquement, un carré est un rectangle particulier :

```text
Square ⊂ Rectangle
```

On pourrait donc être tenté d'écrire :

```ts
class Rectangle {
  width = 0;
  height = 0;

  setWidth(width: number): void {
    this.width = width;
  }

  setHeight(height: number): void {
    this.height = height;
  }

  area(): number {
    return this.width * this.height;
  }
}
```

Puis :

```ts
class Square extends Rectangle {
  setWidth(width: number): void {
    this.width = width;
    this.height = width;
  }

  setHeight(height: number): void {
    this.width = height;
    this.height = height;
  }
}
```

À première vue, cela semble correct.

Mais examinons le comportement attendu par le code utilisateur :

```ts
const rectangle: Rectangle = new Rectangle();

rectangle.setWidth(5);
rectangle.setHeight(2);

console.assert(rectangle.area() === 10);
```

Avec un rectangle :

```text
width  = 5
height = 2

area = 5 × 2
     = 10
```

Tout fonctionne.

---

# 5. Que se passe-t-il avec un Square ?

Supposons maintenant :

```ts
const rectangle: Rectangle = new Square();

rectangle.setWidth(5);
rectangle.setHeight(2);
```

Après :

```ts
rectangle.setWidth(5);
```

le carré devient :

```text
width  = 5
height = 5
```

Puis :

```ts
rectangle.setHeight(2);
```

le carré devient :

```text
width  = 2
height = 2
```

Donc :

```ts
rectangle.area();
```

retourne :

```text
2 × 2 = 4
```

alors que le code appelant s'attendait à :

```text
5 × 2 = 10
```

L'affirmation suivante échoue donc :

```ts
console.assert(rectangle.area() === 10);
```

---

# 6. Pourquoi est-ce une violation du LSP ?

Le problème est que le code appelant pense travailler avec un `Rectangle`.

Il suppose donc que :

```ts
rectangle.setWidth(5);
rectangle.setHeight(2);
```

permet de modifier indépendamment la largeur et la hauteur.

Mais `Square` impose une règle différente :

```text
width === height
```

Le contrat comportemental n'est donc plus le même.

On a :

```text
Rectangle
├── width peut être modifiée indépendamment
└── height peut être modifiée indépendamment

Square
├── width modifie également height
└── height modifie également width
```

Ainsi :

> `Square` ne peut pas remplacer `Rectangle` sans modifier le comportement attendu du programme.

C'est une violation du **LSP**.

---

# 7. Le problème n'est pas simplement l'héritage

Il est important de comprendre que le problème n'est pas uniquement :

```ts
class Square extends Rectangle
```

Le véritable problème est la **compatibilité comportementale**.

Le contrat de `Rectangle` permet :

```text
width = 5
height = 2
```

alors que `Square` impose :

```text
width = height
```

Les deux modèles ont donc des règles incompatibles.

Le LSP ne demande pas simplement :

```text
"Est-ce que S hérite de T ?"
```

Il demande plutôt :

```text
"Est-ce que S respecte réellement le contrat de T ?"
```

---

# 8. Pourquoi les `if` sont un signal d'alerte ?

Une solution pourrait consister à détecter le type concret :

```ts
function calculateArea(rectangle: Rectangle): number {
  if (rectangle instanceof Square) {
    // traitement particulier
  }

  return rectangle.area();
}
```

Mais cette approche augmente le couplage.

On finit potentiellement avec :

```ts
if (rectangle instanceof Square) {
  // ...
}

if (rectangle instanceof SpecialRectangle) {
  // ...
}

if (rectangle instanceof AnotherRectangle) {
  // ...
}
```

Le code client commence alors à connaître les détails des différentes implémentations.

Cela réduit l'intérêt du polymorphisme.

L'objectif devrait plutôt être :

```ts
rectangle.area();
```

et non :

```ts
if (rectangle instanceof Square) {
  // ...
}
```

### Signal architectural

Lorsque le code client doit connaître les sous-types pour fonctionner correctement, il faut se demander :

> **L'abstraction est-elle réellement substituable ?**

---

# 9. Le LSP ne concerne pas uniquement l'héritage

Le LSP était initialement associé à l'héritage :

```text
Parent
  ↑
Child
```

Mais son application est beaucoup plus large.

Il concerne également :

* les interfaces ;
* les classes qui implémentent ces interfaces ;
* les services REST ;
* les repositories ;
* les adaptateurs ;
* les composants architecturaux ;
* les implémentations interchangeables.

L'idée reste toujours la même :

> **Les utilisateurs d'une abstraction doivent pouvoir utiliser ses différentes implémentations sans avoir à connaître leurs particularités.**

---

# 10. Exemple avec une interface TypeScript

Prenons une interface de paiement :

```ts
interface PaymentMethod {
  pay(amount: number): void;
}
```

Nous pouvons avoir plusieurs implémentations :

```ts
class CreditCardPayment implements PaymentMethod {
  pay(amount: number): void {
    console.log(`Payment by card: ${amount}`);
  }
}
```

```ts
class MobileMoneyPayment implements PaymentMethod {
  pay(amount: number): void {
    console.log(`Payment by Mobile Money: ${amount}`);
  }
}
```

```ts
class WavePayment implements PaymentMethod {
  pay(amount: number): void {
    console.log(`Payment by Wave: ${amount}`);
  }
}
```

Le code métier peut dépendre uniquement de l'abstraction :

```ts
function processPayment(
  paymentMethod: PaymentMethod,
  amount: number
): void {
  paymentMethod.pay(amount);
}
```

On peut alors faire :

```ts
processPayment(
  new CreditCardPayment(),
  1000
);
```

ou :

```ts
processPayment(
  new MobileMoneyPayment(),
  1000
);
```

ou :

```ts
processPayment(
  new WavePayment(),
  1000
);
```

Le code de `processPayment()` reste identique.

---

# 11. Le LSP et les interfaces

Le même principe fonctionne avec une interface :

```text
             PaymentMethod
                   │
        ┌──────────┼──────────┐
        │          │          │
       Card      Mobile      Wave
```

Le code client dépend de :

```ts
PaymentMethod
```

et non de :

```ts
CreditCardPayment
MobileMoneyPayment
WavePayment
```

Chaque implémentation doit respecter le contrat :

```ts
paymentMethod.pay(amount);
```

Si une implémentation nécessite un traitement spécial :

```ts
if (paymentMethod instanceof WavePayment) {
  // ...
}
```

cela peut être le signe que l'abstraction ne couvre pas correctement le comportement attendu.

---

# 12. Exemple architectural : les services de taxis

Le texte présente ensuite un exemple architectural.

Imaginons une application qui agrège plusieurs compagnies de taxis :

```text
                  Application
                       │
                       ▼
                Dispatch Service
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
       Purple         Acme         Other
        Taxi          Taxi          Taxi
```

L'application doit pouvoir envoyer une demande de course à différentes compagnies.

Elle suppose que toutes les compagnies utilisent le même format REST.

Par exemple :

```text
/pickupAddress/24 Maple St
/pickupTime/153
/destination/ORD
```

L'application peut donc construire automatiquement la requête.

---

# 13. Le problème avec Acme

Supposons que la majorité des services utilisent :

```text
destination
```

mais qu'Acme utilise :

```text
dest
```

Nous avons alors :

```text
Services standards :

/pickupAddress/%s
/pickupTime/%s
/destination/%s
```

et :

```text
Acme :

/pickupAddress/%s
/pickupTime/%s
/dest/%s
```

Acme ne respecte donc pas complètement le contrat REST attendu.

---

# 14. Conséquence dans l'architecture

Notre application doit maintenant connaître cette particularité.

Une solution naïve serait :

```ts
if (driver.getDispatchUri().startsWith('acme.com')) {
  // format Acme
} else {
  // format standard
}
```

Le problème est que notre code métier connaît désormais :

```text
Acme
```

On commence à introduire des règles spécifiques :

```text
Application
   │
   ├── Purple rules
   ├── Acme rules
   ├── Other rules
   └── ...
```

Si une nouvelle entreprise utilise encore un autre format, il faudra probablement ajouter une nouvelle condition.

---

# 15. La complexité architecturale

Un petit problème d'interface peut donc entraîner :

```text
Interface non substituable
          ↓
Cas particulier
          ↓
if / else
          ↓
Configuration supplémentaire
          ↓
Logique supplémentaire
          ↓
Couplage
          ↓
Complexité architecturale
```

C'est précisément ce que le texte veut démontrer.

> **Une violation du LSP peut polluer l'architecture entière du système.**

---

# 16. Une approche basée sur la configuration

Le texte propose d'isoler ces différences dans une configuration.

Par exemple :

```text
URI          Dispatch Format

Acme.com     /pickupAddress/%s/pickupTime/%s/dest/%s

*.*          /pickupAddress/%s/pickupTime/%s/destination/%s
```

L'application principale n'a donc pas besoin de multiplier les `if`.

On peut aller encore plus loin avec une abstraction :

```ts
interface DispatchFormatter {
  format(request: DispatchRequest): string;
}
```

Puis :

```ts
class StandardDispatchFormatter
  implements DispatchFormatter {

  format(request: DispatchRequest): string {
    return `/pickupAddress/${request.pickupAddress}` +
           `/pickupTime/${request.pickupTime}` +
           `/destination/${request.destination}`;
  }
}
```

et :

```ts
class AcmeDispatchFormatter
  implements DispatchFormatter {

  format(request: DispatchRequest): string {
    return `/pickupAddress/${request.pickupAddress}` +
           `/pickupTime/${request.pickupTime}` +
           `/dest/${request.destination}`;
  }
}
```

La particularité d'Acme est alors isolée.

---

# 17. LSP dans Clean Architecture

Le LSP est particulièrement important dans une architecture comme :

```text
              Use Case
                 │
                 ▼
            Abstraction
                 ▲
                 │
        ┌────────┼────────┐
        │        │        │
      Mongo    Postgres  Memory
```

Par exemple :

```ts
interface UserRepository {
  save(user: User): Promise<void>;
  findById(id: string): Promise<User | null>;
}
```

Implémentation MongoDB :

```ts
class MongoUserRepository implements UserRepository {
  async save(user: User): Promise<void> {
    // MongoDB
  }

  async findById(id: string): Promise<User | null> {
    // MongoDB
    return null;
  }
}
```

Implémentation en mémoire :

```ts
class InMemoryUserRepository implements UserRepository {
  async save(user: User): Promise<void> {
    // In-memory
  }

  async findById(id: string): Promise<User | null> {
    // In-memory
    return null;
  }
}
```

Le Use Case dépend de :

```ts
UserRepository
```

et non de :

```ts
MongoUserRepository
```

On peut donc remplacer :

```text
MongoUserRepository
```

par :

```text
InMemoryUserRepository
```

sans modifier le Use Case, **à condition que les deux respectent le même contrat comportemental**.

---

# 18. Attention : respecter la signature ne suffit pas

C'est une nuance essentielle.

Considérons :

```ts
interface UserRepository {
  findById(id: string): Promise<User | null>;
}
```

Deux implémentations peuvent respecter exactement la même signature :

```ts
class MongoUserRepository implements UserRepository {
  async findById(id: string): Promise<User | null> {
    // retourne null si l'utilisateur n'existe pas
    return null;
  }
}
```

et :

```ts
class BadUserRepository implements UserRepository {
  async findById(id: string): Promise<User | null> {
    throw new Error('User not found');
  }
}
```

Techniquement, les deux respectent l'interface TypeScript.

Mais si le contrat métier indique :

```text
Utilisateur absent → null
```

alors `BadUserRepository` ne respecte pas le comportement attendu.

Cela signifie :

```text
LSP ≠ seulement respecter les signatures
```

mais plutôt :

```text
LSP = respecter le contrat
      +
      respecter le comportement attendu
```

---

# 19. Le contrat comporte plusieurs dimensions

Quand on parle de contrat, il ne faut donc pas penser uniquement aux signatures TypeScript.

Le contrat peut également définir :

### Entrées acceptées

```ts
pay(amount: number)
```

Par exemple :

```text
amount > 0
```

### Sorties

```text
Utilisateur trouvé → User
Utilisateur absent → null
```

### Exceptions

```text
Une erreur spécifique peut être levée
```

### Effets secondaires

```text
Le paiement doit être enregistré
```

### Invariants

```text
Le solde ne doit jamais devenir négatif
```

Une implémentation qui viole ces attentes peut violer le LSP même si TypeScript accepte son code.

---

# 20. LSP et SOLID

Le LSP est le troisième principe de SOLID :

```text
S → Single Responsibility Principle
O → Open/Closed Principle
L → Liskov Substitution Principle
I → Interface Segregation Principle
D → Dependency Inversion Principle
```

Le LSP est particulièrement lié à **OCP** et **DIP**.

---

## 20.1 LSP et Open/Closed Principle

Le principe Open/Closed dit qu'un système doit être :

```text
Ouvert à l'extension
Fermé à la modification
```

Si les nouvelles implémentations respectent correctement l'abstraction :

```text
              Abstraction
                   ▲
          ┌────────┼────────┐
          │        │        │
         Impl A   Impl B   Impl C
```

le code client n'a pas besoin d'être modifié.

Le LSP facilite donc l'application de l'OCP.

---

## 20.2 LSP et Dependency Inversion Principle

Le Dependency Inversion Principle encourage une architecture :

```text
Use Case
   │
   ▼
Interface
   ▲
   │
Implementation
```

Mais cette architecture n'est utile que si les implémentations respectent réellement l'abstraction.

On peut résumer :

```text
DIP
 ↓
Le code dépend d'une abstraction.

LSP
 ↓
Les implémentations respectent cette abstraction.

OCP
 ↓
De nouvelles implémentations peuvent être ajoutées
sans modifier le code client.
```

---

# 21. LSP et polymorphisme

Le LSP est étroitement lié au polymorphisme.

Le polymorphisme permet :

```ts
function processPayment(
  paymentMethod: PaymentMethod
): void {
  paymentMethod.pay(1000);
}
```

d'utiliser différentes implémentations :

```text
PaymentMethod
      ▲
      │
 ┌────┼─────┐
 │    │     │
Card Mobile Wave
```

Mais le polymorphisme devient réellement utile lorsque ces implémentations sont **substituables**.

Autrement dit :

```text
Polymorphisme
      +
Substituabilité
      ↓
Code faiblement couplé
```

---

# 22. Comment détecter une violation du LSP ?

Voici plusieurs signaux d'alerte.

### 1. Des `instanceof` partout

```ts
if (value instanceof A) {
  // ...
}

if (value instanceof B) {
  // ...
}
```

### 2. Des exceptions du type "non supporté"

```ts
class SpecialPayment implements PaymentMethod {
  pay(amount: number): void {
    throw new Error('Not supported');
  }
}
```

si le contrat exige réellement que `pay()` fonctionne.

### 3. Une implémentation qui ignore une méthode

```ts
interface Repository {
  save(): void;
  delete(): void;
}
```

et :

```ts
class ReadOnlyRepository implements Repository {
  save(): void {
    throw new Error('Not supported');
  }

  delete(): void {
    throw new Error('Not supported');
  }
}
```

Cela peut indiquer que l'abstraction est mal conçue.

### 4. Des conditions spécifiques dans le code client

```ts
if (provider === 'ACME') {
  // comportement spécial
}
```

### 5. Des résultats ou erreurs différents de ceux attendus

Par exemple :

```text
Implémentation A → retourne null
Implémentation B → lance une exception
```

alors que le contrat prévoit un comportement précis.

---

# 23. Une bonne question à se poser

Lorsque tu crées une interface ou une classe abstraite, demande-toi :

> **"Si je remplace cette implémentation par une autre, est-ce que le code client continue de fonctionner correctement ?"**

Par exemple :

```ts
interface PaymentMethod {
  pay(amount: number): void;
}
```

Puis :

```ts
function checkout(
  paymentMethod: PaymentMethod,
  amount: number
): void {
  paymentMethod.pay(amount);
}
```

On devrait pouvoir utiliser :

```ts
checkout(new CreditCardPayment(), 1000);

checkout(new MobileMoneyPayment(), 1000);

checkout(new WavePayment(), 1000);
```

sans modifier `checkout()`.

Si on commence à écrire :

```ts
if (paymentMethod instanceof WavePayment) {
  // ...
}
```

il faut se demander si l'abstraction `PaymentMethod` est suffisamment bien définie.

---

# 24. LSP : mauvaise et bonne architecture

## ❌ Mauvaise approche

```text
                  Client
                    │
                    ▼
              PaymentMethod
                    │
                    ├── Card
                    ├── Mobile Money
                    └── Wave
                         │
                         ▼
                   comportement
                   particulier
                         │
                         ▼
                   if instanceof
```

Le client doit connaître les détails des implémentations.

---

## ✅ Approche préférable

```text
                  Client
                    │
                    ▼
              PaymentMethod
                    ▲
          ┌─────────┼─────────┐
          │         │         │
         Card      Mobile     Wave
```

Le client connaît uniquement :

```ts
PaymentMethod
```

Chaque implémentation respecte le contrat.

---

# 25. La vision architecturale

Le LSP ne doit donc pas être vu uniquement comme :

```text
"Est-ce que mon héritage est correct ?"
```

Il faut le voir comme :

```text
"Est-ce que mes abstractions sont réellement substituables ?"
```

Cela concerne :

```text
Classes
   ↓
Interfaces
   ↓
Services
   ↓
Repositories
   ↓
Adapters
   ↓
APIs
   ↓
Architecture globale
```

Une violation locale peut entraîner une complexité beaucoup plus importante dans les couches supérieures.

---

# 26. Résumé

Le **Liskov Substitution Principle** peut être résumé en cinq idées.

### 1. Un sous-type doit pouvoir remplacer son parent

```text
S peut remplacer T
```

sans casser le programme.

### 2. Il faut respecter le contrat comportemental

Ce n'est pas uniquement une question de signatures TypeScript.

```text
Contrat
+
Comportement
```

doivent être respectés.

### 3. Le code client ne doit pas connaître les implémentations

Éviter :

```ts
if (value instanceof ConcreteType)
```

lorsque cela sert à compenser une mauvaise abstraction.

### 4. Le LSP concerne aussi les interfaces

Il s'applique à :

```text
Interfaces
Repositories
Services
APIs
Adapters
```

et pas uniquement à l'héritage.

### 5. Une violation du LSP peut polluer l'architecture

```text
Violation
   ↓
Cas particuliers
   ↓
if / else
   ↓
Couplage
   ↓
Complexité
   ↓
Architecture difficile à maintenir
```

---

# 27. 🧠 La phrase à retenir

> **Le LSP signifie qu'une implémentation doit pouvoir remplacer l'abstraction qu'elle représente sans modifier le comportement attendu du programme.**

Ou, dans une formulation plus orientée **Senior Developer** :

> **Une abstraction n'est réellement utile que si ses différentes implémentations sont comportementalement substituables. Le code client doit pouvoir programmer contre le contrat, sans connaître ni gérer les particularités de chaque implémentation.**

### Mental model

```text
        PROGRAMMER CONTRE UN CONTRAT
                    │
                    ▼
             ┌─────────────┐
             │ Abstraction │
             └──────┬──────┘
                    ▲
          ┌─────────┼─────────┐
          │         │         │
       Impl. A   Impl. B   Impl. C
          │         │         │
          └─────────┴─────────┘
                    │
                    ▼
             Même comportement
             attendu par le client
```

**En une seule phrase :**

> **Si mon code attend `T`, je dois pouvoir lui donner `S` sans que le code sache qu'il s'agit de `S` et sans que son comportement attendu soit modifié.**
