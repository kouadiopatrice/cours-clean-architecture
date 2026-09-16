# Le polymorphisme en TypeScript

## Introduction

Le **polymorphisme** est l’un des concepts fondamentaux de la programmation orientée objet (POO).

Le terme vient du grec et signifie littéralement :

> **« Plusieurs formes »**

En programmation, le polymorphisme permet de **manipuler plusieurs objets différents à travers un type commun**, tout en laissant chaque objet fournir son propre comportement.

Autrement dit :

> **Le code appelant connaît le contrat dont il a besoin, mais n’a pas nécessairement besoin de connaître l’implémentation concrète de l’objet utilisé.**

Cette notion est particulièrement importante dans la conception d’architectures faiblement couplées et extensibles.

---

# 1. Comprendre le polymorphisme

Prenons un exemple simple avec des animaux :

```ts
class Animal {
  makeSound(): void {
    console.log('Some sound');
  }
}

class Dog extends Animal {
  makeSound(): void {
    console.log('Woof!');
  }
}

class Cat extends Animal {
  makeSound(): void {
    console.log('Meow!');
  }
}
```

Nous avons la relation suivante :

```text
              Animal
              /    \
             /      \
           Dog      Cat
```

`Dog` et `Cat` sont tous les deux des `Animal`.

Nous pouvons donc écrire :

```ts
const dog: Animal = new Dog();
const cat: Animal = new Cat();
```

L’objet réellement créé est :

```text
dog → Dog
cat → Cat
```

mais les variables sont manipulées à travers le type :

```text
dog → Animal
cat → Animal
```

C’est une première manifestation du polymorphisme.

---

# 2. Pourquoi le polymorphisme est-il intéressant ?

Le véritable intérêt du polymorphisme apparaît lorsqu’une fonction doit manipuler plusieurs types différents **sans connaître leur implémentation concrète**.

Par exemple :

```ts
function makeAnimalSpeak(animal: Animal): void {
  animal.makeSound();
}
```

La même fonction peut maintenant recevoir différents types d’animaux :

```ts
makeAnimalSpeak(new Dog());
makeAnimalSpeak(new Cat());
```

Le résultat dépend de l’implémentation réelle de l’objet :

```text
Dog → Woof!
Cat → Meow!
```

Pourtant, la fonction `makeAnimalSpeak()` ne connaît que :

```ts
Animal
```

Elle ne contient aucune logique spécifique à `Dog` ou `Cat`.

Nous avons donc :

```text
                  Animal
                    │
              makeSound()
                    │
          ┌─────────┴─────────┐
          ↓                   ↓
         Dog                 Cat
          │                   │
       Woof!                Meow!
```

Une seule fonction peut ainsi travailler avec plusieurs sous-types.

C’est l’un des principaux avantages du polymorphisme.

---

# 3. Le lien avec l’héritage

Le polymorphisme est souvent associé à l’héritage.

Considérons :

```ts
class Animal {
  move(): void {
    console.log('Moving...');
  }
}

class Dog extends Animal {}

class Bird extends Animal {}
```

La relation est :

```text
              Animal
                 │
          ┌──────┴──────┐
          ↓             ↓
         Dog           Bird
```

Grâce à `extends Animal`, `Dog` et `Bird` héritent de `move()`.

Nous pouvons donc écrire :

```ts
const dog = new Dog();
const bird = new Bird();

dog.move();
bird.move();
```

Mais nous pouvons également les manipuler à travers leur type parent :

```ts
const dog: Animal = new Dog();
const bird: Animal = new Bird();
```

Les objets réels sont :

```text
dog  → Dog
bird → Bird
```

mais leur type de référence est :

```text
dog  → Animal
bird → Animal
```

C’est précisément ce mécanisme qui permet au code appelant de travailler avec une abstraction commune.

---

# 4. La règle fondamentale : respecter le contrat du type

Lorsqu’un objet est manipulé à travers son type parent, le code appelant ne peut utiliser que ce que ce type garantit.

Considérons :

```ts
class Animal {
  move(): void {
    console.log('Moving');
  }
}

class Dog extends Animal {
  bark(): void {
    console.log('Woof!');
  }
}
```

Puis :

```ts
const dog: Animal = new Dog();
```

Cette instruction est valide :

```ts
dog.move();
```

Mais celle-ci ne l’est pas :

```ts
dog.bark();
```

Pourquoi ?

Parce que TypeScript regarde le type déclaré de la variable :

```ts
dog: Animal
```

et `Animal` ne garantit pas l’existence de :

```ts
bark()
```

Même si l’objet réel est bien une instance de `Dog`.

On peut le représenter ainsi :

```text
Variable
   │
   ▼
Animal
   │
   └── move() ✅

Objet réel
   │
   ▼
Dog
   │
   ├── move()
   └── bark()
```

Le code qui manipule `dog` à travers `Animal` doit donc se limiter au **contrat fourni par `Animal`**.

---

# 5. Le véritable intérêt : des comportements différents derrière un même contrat

Le polymorphisme devient particulièrement puissant lorsque plusieurs implémentations fournissent des comportements différents pour une même opération.

Exemple :

```ts
class Animal {
  performAction(): void {
    // Action générale
  }
}

class Dog extends Animal {
  performAction(): void {
    console.log('The dog is barking');
  }
}

class Bird extends Animal {
  performAction(): void {
    console.log('The bird is flying');
  }
}
```

Nous pouvons maintenant créer une collection d’animaux :

```ts
const animals: Animal[] = [
  new Dog(),
  new Bird(),
];
```

Puis :

```ts
for (const animal of animals) {
  animal.performAction();
}
```

Résultat :

```text
The dog is barking
The bird is flying
```

La boucle ne connaît jamais explicitement :

```text
Dog
Bird
```

Elle connaît uniquement :

```text
Animal
```

Pourtant, chaque objet exécute son propre comportement.

C’est **le cœur du polymorphisme**.

---

# 6. Le concept de contrat

Une manière très efficace de comprendre le polymorphisme consiste à considérer le type abstrait comme un **contrat**.

Par exemple :

```ts
abstract class Animal {
  abstract performAction(): void;
}
```

La classe abstraite impose une règle :

> **Tout `Animal` doit fournir une implémentation de `performAction()`.**

Mais elle ne définit pas nécessairement comment cette opération doit être réalisée.

`Dog` fournit sa propre implémentation :

```ts
class Dog extends Animal {
  performAction(): void {
    console.log('Bark');
  }
}
```

`Bird` fournit la sienne :

```ts
class Bird extends Animal {
  performAction(): void {
    console.log('Fly');
  }
}
```

Nous avons donc :

```text
                 Animal
                   │
                   └── performAction()
                          │
             ┌────────────┴────────────┐
             ↓                         ↓
            Dog                       Bird
             │                         │
             ↓                         ↓
      performAction()           performAction()
             │                         │
           Bark                       Fly
```

Le **contrat est commun**, mais l’implémentation est spécifique.

---

# 7. Exemple métier : application bancaire

Le concept devient encore plus intéressant lorsqu’on l’applique à un domaine métier réel.

Imaginons une application bancaire.

Nous pouvons définir une classe abstraite :

```ts
abstract class Account {
  abstract deposit(amount: number): void;

  transferFunds(
    amount: number,
    targetAccount: Account,
  ): void {
    // Retirer le montant du compte source
    // ...

    // Déposer le montant sur le compte cible
    targetAccount.deposit(amount);
  }
}
```

La partie essentielle est :

```ts
targetAccount: Account
```

Cela signifie :

> **`transferFunds()` peut recevoir n’importe quel compte qui respecte le contrat `Account`.**

La méthode n’a donc pas besoin de connaître le type concret du compte cible.

---

# 8. Plusieurs types de comptes

Nous pouvons définir plusieurs implémentations :

```ts
class CheckingAccount extends Account {
  deposit(amount: number): void {
    console.log(`Deposit ${amount} into checking account`);
  }
}
```

Et :

```ts
class SavingsAccount extends Account {
  deposit(amount: number): void {
    console.log(`Deposit ${amount} into savings account`);
  }
}
```

La hiérarchie est :

```text
                    Account
                   /       \
                  /         \
                 ↓           ↓
      CheckingAccount    SavingsAccount
```

Les deux classes respectent le contrat `Account`.

---

# 9. Le polymorphisme en action

Nous pouvons maintenant écrire :

```ts
const checking = new CheckingAccount();
const savings = new SavingsAccount();

checking.transferFunds(100, savings);
```

La méthode attend :

```ts
transferFunds(
  amount: number,
  targetAccount: Account,
)
```

mais nous lui transmettons :

```ts
SavingsAccount
```

Cela fonctionne parce que :

```text
SavingsAccount
       ↓
    extends
       ↓
     Account
```

Donc :

> **Un `SavingsAccount` peut être utilisé partout où un `Account` est attendu.**

C’est une application concrète du polymorphisme.

---

# 10. Pourquoi éviter les tests sur le type concret ?

Une mauvaise conception consisterait à faire dépendre le comportement du type concret :

```ts
function transferFunds(
  target: CheckingAccount | SavingsAccount,
): void {
  if (target instanceof CheckingAccount) {
    target.deposit(100);
  }

  if (target instanceof SavingsAccount) {
    target.deposit(100);
  }
}
```

Cette approche devient problématique lorsqu’un nouveau type de compte apparaît.

Par exemple :

```ts
class InvestmentAccount extends Account {
  deposit(amount: number): void {
    console.log('Investment deposit');
  }
}
```

Il faudrait modifier `transferFunds()` pour prendre en compte ce nouveau type.

On obtient progressivement :

```text
transferFunds()
       │
       ├── CheckingAccount
       ├── SavingsAccount
       ├── InvestmentAccount
       └── ...
```

La fonction devient de plus en plus dépendante des implémentations concrètes.

---

# 11. La solution polymorphique

Avec le polymorphisme, la logique reste générique :

```ts
function transferFunds(
  source: Account,
  target: Account,
): void {
  target.deposit(100);
}
```

La fonction ne se préoccupe pas de savoir si `target` est :

```text
CheckingAccount
SavingsAccount
InvestmentAccount
PremiumAccount
...
```

Elle sait simplement que :

```text
Account
   │
   └── deposit()
```

Si nous ajoutons :

```ts
class InvestmentAccount extends Account {
  deposit(amount: number): void {
    console.log(`Investment deposit: ${amount}`);
  }
}
```

nous n’avons pas besoin de modifier `transferFunds()`.

La structure devient :

```text
                    Account
                       │
          ┌────────────┼────────────┐
          ↓            ↓            ↓
      Checking      Savings     Investment
          │            │            │
          └────────────┼────────────┘
                       │
                   deposit()
                       │
                       ▼
                transferFunds()
```

`transferFunds()` dépend uniquement de l’abstraction `Account` et de son contrat `deposit()`.

---

# 12. « Polymorphism at its finest »

C’est précisément ce qui rend le polymorphisme particulièrement puissant.

La méthode :

```ts
targetAccount.deposit(amount);
```

ne demande jamais :

```ts
if (targetAccount instanceof CheckingAccount) {
  // ...
}

if (targetAccount instanceof SavingsAccount) {
  // ...
}
```

Elle dit simplement :

> **« Je ne me préoccupe pas du type concret de l’objet. Je sais seulement qu’il respecte le contrat `Account` et qu’il fournit `deposit()`. »**

Le code appelant dépend donc de **ce que l’objet sait faire**, et non de **l’identité concrète de l’objet**.

---

# 13. Polymorphisme et Open/Closed Principle

Le polymorphisme est fortement lié à l’**Open/Closed Principle (OCP)**.

L’OCP recommande qu’un système soit :

> **Ouvert à l’extension, mais fermé à la modification.**

Avec notre exemple :

```ts
function transferFunds(
  source: Account,
  target: Account,
): void {
  target.deposit(100);
}
```

Nous pouvons ajouter :

```ts
class PremiumAccount extends Account {
  deposit(amount: number): void {
    // ...
  }
}
```

sans modifier :

```ts
transferFunds()
```

Le système est donc extensible par ajout de nouvelles implémentations.

```text
                     Account
                        │
          ┌─────────────┼─────────────┐
          ↓             ↓             ↓
      Checking       Savings       Premium
```

Le code existant continue à fonctionner avec les nouvelles implémentations.

Le polymorphisme devient ainsi un **mécanisme permettant de mettre en œuvre l’OCP**.

---

# 14. Le polymorphisme avec les interfaces TypeScript

En TypeScript, il est important de comprendre que le polymorphisme **ne nécessite pas l’héritage**.

Il peut également être obtenu avec les interfaces et le typage structurel.

Par exemple :

```ts
interface PaymentMethod {
  pay(amount: number): void;
}
```

Nous pouvons créer plusieurs implémentations :

```ts
class CreditCardPayment implements PaymentMethod {
  pay(amount: number): void {
    console.log(`Pay ${amount} with credit card`);
  }
}
```

```ts
class MobileMoneyPayment implements PaymentMethod {
  pay(amount: number): void {
    console.log(`Pay ${amount} with Mobile Money`);
  }
}
```

Puis une fonction générique :

```ts
function processPayment(
  paymentMethod: PaymentMethod,
  amount: number,
): void {
  paymentMethod.pay(amount);
}
```

Nous pouvons lui transmettre différentes implémentations :

```ts
processPayment(
  new CreditCardPayment(),
  10000,
);

processPayment(
  new MobileMoneyPayment(),
  10000,
);
```

La fonction `processPayment()` ne connaît pas :

```text
CreditCardPayment
MobileMoneyPayment
```

Elle connaît uniquement :

```text
PaymentMethod
      │
      └── pay()
```

---

# 15. Le typage structurel de TypeScript

Cette approche est particulièrement naturelle en TypeScript grâce à son **structural typing**.

Le point essentiel est que TypeScript vérifie principalement si un objet possède la structure attendue.

Par exemple :

```ts
interface PaymentMethod {
  pay(amount: number): void;
}
```

Une classe qui possède :

```ts
pay(amount: number): void
```

peut satisfaire ce contrat.

L’idée devient :

```text
PaymentMethod
      │
      └── pay(amount)
            ▲
            │
    ┌───────┼────────┐
    │       │        │
 Credit   Mobile    Wave
 Card     Money
```

Le code métier dépend du contrat, tandis que les implémentations peuvent évoluer indépendamment.

---

# 16. Application à Clean Architecture, Hexagonal Architecture et DDD

Le polymorphisme joue également un rôle important dans les architectures modernes telles que :

* **Clean Architecture** ;
* **Hexagonal Architecture** ;
* **DDD (Domain-Driven Design)**.

Prenons un exemple de repository.

Le domaine définit uniquement le contrat :

```ts
interface UserRepository {
  save(user: User): Promise<void>;
}
```

Nous pouvons ensuite avoir plusieurs implémentations :

```ts
class MongoUserRepository implements UserRepository {
  async save(user: User): Promise<void> {
    // MongoDB
  }
}
```

```ts
class PostgresUserRepository implements UserRepository {
  async save(user: User): Promise<void> {
    // PostgreSQL
  }
}
```

Et éventuellement :

```ts
class InMemoryUserRepository implements UserRepository {
  async save(user: User): Promise<void> {
    // Stockage en mémoire pour les tests
  }
}
```

Le use case dépend uniquement de l’abstraction :

```ts
class CreateUserUseCase {
  constructor(
    private readonly repository: UserRepository,
  ) {}

  async execute(user: User): Promise<void> {
    await this.repository.save(user);
  }
}
```

Le use case ne sait pas s’il utilise :

```text
MongoDB
PostgreSQL
MySQL
API externe
InMemoryRepository
```

Il connaît uniquement :

```text
UserRepository
      │
      └── save()
```

Cela permet de réduire fortement le couplage entre la logique métier et les détails techniques.

---

# 17. Polymorphisme et inversion des dépendances

Cette approche rejoint également le **Dependency Inversion Principle (DIP)**.

L'objectif est d'éviter une dépendance directe :

```text
CreateUserUseCase
       ↓
MongoUserRepository
       ↓
MongoDB
```

et de privilégier :

```text
CreateUserUseCase
       ↓
UserRepository
       ↑
MongoUserRepository
       ↓
MongoDB
```

Le use case dépend d’une abstraction :

```ts
UserRepository
```

et l’implémentation technique respecte cette abstraction.

Le polymorphisme permet alors au même use case de fonctionner avec différentes implémentations.

---

# 18. Héritage et polymorphisme : deux concepts différents

Il est important de ne pas confondre **héritage** et **polymorphisme**.

## Héritage

L’héritage exprime une relation de spécialisation :

```ts
class Dog extends Animal {}
```

Cela signifie :

```text
Dog IS-A Animal
```

Autrement dit :

> **Dog est une spécialisation de Animal.**

## Polymorphisme

Le polymorphisme permet d’utiliser cette spécialisation lorsqu’un type général est attendu :

```ts
const animal: Animal = new Dog();
```

Cela signifie :

```text
Dog
 ↓
peut être utilisé comme
 ↓
Animal
```

L’héritage est donc **un moyen courant d’obtenir du polymorphisme**, mais le polymorphisme ne se limite pas à l’héritage.

En TypeScript, les interfaces et le typage structurel permettent également de mettre en œuvre le polymorphisme.

---

# 19. Exemple complet : système de paiement

Considérons une abstraction :

```ts
interface Payment {
  pay(): void;
}
```

Plusieurs moyens de paiement peuvent implémenter ce contrat :

```ts
class CreditCardPayment implements Payment {
  pay(): void {
    console.log('Credit card');
  }
}

class OrangeMoneyPayment implements Payment {
  pay(): void {
    console.log('Orange Money');
  }
}

class WavePayment implements Payment {
  pay(): void {
    console.log('Wave');
  }
}
```

Notre service reste indépendant des implémentations :

```ts
function processPayment(payment: Payment): void {
  payment.pay();
}
```

Nous pouvons maintenant utiliser :

```ts
processPayment(new CreditCardPayment());
processPayment(new OrangeMoneyPayment());
processPayment(new WavePayment());
```

Et demain :

```ts
class PayPalPayment implements Payment {
  pay(): void {
    console.log('PayPal');
  }
}
```

La fonction :

```ts
processPayment()
```

n’a pas besoin d’être modifiée.

C’est exactement le type de découplage recherché dans une architecture extensible.

---

# 20. Synthèse du fonctionnement

Le polymorphisme peut être représenté ainsi :

```text
                 TYPE GÉNÉRAL
                      │
                      ▼
                ┌─────────────┐
                │   Account   │
                │             │
                │  deposit()  │
                └──────┬──────┘
                       │
          ┌────────────┼────────────┐
          ↓            ↓            ↓
      Checking      Savings     Investment
       Account       Account       Account
          │            │            │
          ↓            ↓            ↓
      deposit()     deposit()    deposit()
```

Le code appelant manipule :

```text
Account
```

mais l’exécution utilise :

```text
CheckingAccount
SavingsAccount
InvestmentAccount
```

selon l’objet réellement fourni.

---

# 21. Résumé conceptuel

Le polymorphisme peut être défini comme suit :

> **Le polymorphisme permet d’écrire du code contre un type abstrait ou général, puis de lui fournir différentes implémentations concrètes respectant ce contrat.**

Une formulation plus intuitive serait :

> **« Je sais ce que l’objet sait faire, mais je n’ai pas besoin de savoir exactement quel objet c’est. »**

Par exemple :

```ts
function transfer(target: Account): void {
  target.deposit(100);
}
```

Cette fonction signifie :

> « Donne-moi un `Account` capable d’effectuer `deposit()`. Je n’ai pas besoin de savoir s’il s’agit d’un compte courant, d’un compte épargne ou d’un compte investissement. »

---

# 22. Une analogie simple : la prise électrique

Une bonne manière de mémoriser le concept est d’utiliser l’analogie d’une **prise électrique standard**.

La prise ne demande pas :

> « Es-tu une télévision ? »

Elle vérifie simplement :

> **« Est-ce que tu respectes le format que j’attends ? »**

En programmation :

```ts
interface Device {
  turnOn(): void;
}
```

Nous pouvons avoir :

```ts
class TV implements Device {
  turnOn(): void {
    console.log('TV ON');
  }
}

class Computer implements Device {
  turnOn(): void {
    console.log('Computer ON');
  }
}

class AirConditioner implements Device {
  turnOn(): void {
    console.log('Air conditioner ON');
  }
}
```

Puis :

```ts
function start(device: Device): void {
  device.turnOn();
}
```

Nous pouvons appeler :

```ts
start(new TV());
start(new Computer());
start(new AirConditioner());
```

`start()` ne connaît pas les types concrets.

Il connaît uniquement le contrat :

```text
Device
  │
  └── turnOn()
```

---

# 23. Modèle mental à retenir

Le polymorphisme peut être résumé par le flux suivant :

```text
                 POLYMORPHISME
                       │
                       ▼
                Type général
                       │
                       ▼
                Contrat commun
                       │
                       ▼
            Plusieurs implémentations
                       │
                       ▼
               Même code appelant
                       │
                       ▼
             Comportements différents
```

Ou, avec l’exemple du paiement :

```text
                    Payment
                       │
                     pay()
                       │
          ┌────────────┼────────────┐
          ↓            ↓            ↓
    Credit Card     OrangeMoney     Wave
          │            │            │
          └────────────┼────────────┘
                       ↓
                processPayment()
```

Le code appelant dépend de :

```text
Payment
```

et non de :

```text
CreditCardPayment
OrangeMoneyPayment
WavePayment
```

---

# 24. Conclusion

Le polymorphisme ne consiste donc pas simplement à savoir utiliser `extends` ou `implements`.

Son véritable intérêt architectural est de permettre au code de **dépendre d’abstractions et de contrats plutôt que d’implémentations concrètes**.

Cette approche permet notamment de :

* réduire le couplage ;
* faciliter l’extension du système ;
* éviter les conditions basées sur le type concret ;
* favoriser le principe Open/Closed ;
* faciliter les tests grâce aux implémentations alternatives ;
* isoler les détails techniques ;
* faciliter l’application du Dependency Inversion Principle ;
* construire des architectures plus flexibles.

Dans une approche **Senior TypeScript**, la question essentielle n’est donc pas :

> « Est-ce que je peux utiliser `extends` ? »

mais plutôt :

> **« De quelle abstraction mon code a-t-il réellement besoin ? »**

Le principe peut finalement être résumé en une phrase :

> 🧠 **Programmer contre un contrat, et laisser l’implémentation concrète déterminer le comportement.**

C’est cette capacité à **substituer différentes implémentations derrière une même abstraction** qui constitue le cœur du polymorphisme.
