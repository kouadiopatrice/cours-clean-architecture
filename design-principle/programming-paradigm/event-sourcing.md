# Event Sourcing

L’**Event Sourcing** est un pattern architectural directement lié aux notions d’**immutabilité**, de **mutation d’état** et de **reconstruction de l’état**.

L’idée fondamentale est simple :

> **Au lieu de considérer l’état actuel comme la source de vérité, on conserve l’historique des événements qui ont produit cet état.**

Cette approche permet notamment de conserver un historique complet des changements, de faciliter l’audit et de reconstruire l’état d’une entité à partir des événements qui lui sont associés.

---

## 1. Le problème : comment gérer l’état ?

Prenons l’exemple d’une application bancaire.

Dans une approche CRUD traditionnelle, on stocke directement l’état courant du compte :

```text
Account
────────────────
id: 123
balance: 1000 €
```

Si le client effectue un dépôt de 200 € :

```text
balance = 1200 €
```

Puis retire 300 € :

```text
balance = 900 €
```

Le système modifie donc continuellement la valeur `balance`.

```text
1000 €
  ↓
1200 €
  ↓
 900 €
```

Cette approche est parfaitement classique : **l’état courant est stocké directement et mis à jour au fil des opérations.**

---

# 2. L’approche Event Sourcing

L’Event Sourcing propose une approche différente.

Au lieu de stocker uniquement :

```text
balance = 900 €
```

on conserve les événements qui ont produit cet état :

```text
Deposit 1000
Deposit 200
Withdraw 300
```

Le solde peut ensuite être reconstruit :

```text
1000
 + 200
 - 300
───────
  900 €
```

On obtient donc le flux suivant :

```text
Événements
    ↓
Application des événements
    ↓
État courant
```

L’état devient ainsi **une conséquence des événements**, plutôt que la principale source de vérité.

---

# 3. Exemple concret

Supposons qu’un compte bancaire commence avec un solde de `0 €`.

Les opérations suivantes sont effectuées :

1. Dépôt de 1 000 €
2. Dépôt de 500 €
3. Retrait de 200 €
4. Dépôt de 300 €

Dans une approche classique, on pourrait simplement stocker :

```text
balance = 1600 €
```

Avec Event Sourcing, on conserve plutôt :

```text
Event 1 → Deposit 1000
Event 2 → Deposit 500
Event 3 → Withdraw 200
Event 4 → Deposit 300
```

Pour reconstruire le solde :

```text
0
 ↓
+1000 = 1000
 ↓
+500  = 1500
 ↓
-200  = 1300
 ↓
+300  = 1600
```

Résultat :

```text
balance = 1600 €
```

On peut donc représenter le principe ainsi :

```text
Events
   ↓
Apply Events
   ↓
Current State
```

---

# 4. Représentation des événements en TypeScript

En TypeScript, les événements peuvent être modélisés à l’aide d'une **union discriminée** :

```ts
type AccountEvent =
  | {
      type: 'DEPOSIT';
      amount: number;
    }
  | {
      type: 'WITHDRAW';
      amount: number;
    };
```

Nous pouvons ensuite définir l’historique du compte :

```ts
const events: AccountEvent[] = [
  { type: 'DEPOSIT', amount: 1000 },
  { type: 'DEPOSIT', amount: 500 },
  { type: 'WITHDRAW', amount: 200 },
  { type: 'DEPOSIT', amount: 300 },
];
```

Le solde peut être reconstruit à partir de ces événements :

```ts
function calculateBalance(events: AccountEvent[]): number {
  return events.reduce((balance, event) => {
    if (event.type === 'DEPOSIT') {
      return balance + event.amount;
    }

    if (event.type === 'WITHDRAW') {
      return balance - event.amount;
    }

    return balance;
  }, 0);
}

const balance = calculateBalance(events);

console.log(balance); // 1600
```

### Point important

La fonction `calculateBalance()` ne modifie pas le tableau `events`.

Elle reçoit :

```text
events
```

et produit :

```text
balance
```

Elle peut donc être considérée comme une fonction de transformation :

```text
Events → State
```

Cette caractéristique est particulièrement proche de la philosophie de la **programmation fonctionnelle**, où l’on privilégie les transformations de données plutôt que la mutation directe de l’état.

---

# 5. État mutable vs événements immuables

La différence fondamentale peut être résumée ainsi.

### Approche traditionnelle

L’application stocke directement l’état :

```text
Account
──────────────
balance: 1600
```

Puis elle le modifie :

```text
1600 → 1800 → 1500 → ...
```

L’état courant est la donnée principale.

### Event Sourcing

L’application conserve les événements :

```text
Deposit 1000
Deposit 500
Withdraw 200
Deposit 300
```

Puis reconstruit l’état :

```text
Events
   ↓
Apply Events
   ↓
1600
```

On peut donc retenir :

> **State = résultat de l’application des événements.**

Et dans une architecture Event Sourcing :

> **Les événements constituent la source de vérité ; l’état courant est reconstruit à partir d’eux.**

---

# 6. Pourquoi l’Event Sourcing est-il intéressant ?

Imaginons qu’un client demande :

> « Pourquoi mon compte affiche-t-il 1 600 € ? »

Dans une approche traditionnelle, la base de données peut simplement contenir :

```text
balance = 1600
```

La valeur actuelle est connue, mais l’historique précis ayant conduit à cette valeur peut avoir été perdu.

Avec Event Sourcing, nous conservons par exemple :

```text
10:00 → Deposit 1000
10:15 → Deposit 500
11:00 → Withdraw 200
12:30 → Deposit 300
```

Nous pouvons donc reconstruire l’histoire complète du compte.

### Principaux avantages

* **Audit** : savoir ce qui s’est produit.
* **Traçabilité** : connaître l’ordre des opérations.
* **Historique** : conserver les changements successifs.
* **Reconstruction** : pouvoir recalculer l’état.
* **Analyse** : exploiter l’historique des événements.
* **Debugging** : comprendre comment un état incorrect a été obtenu.

Selon le modèle d’événement utilisé, on peut également conserver :

```text
who       → qui a effectué l'opération
when      → quand
what      → quelle opération
where     → éventuellement depuis quel contexte
metadata  → informations complémentaires
```

---

# 7. « Nothing ever gets deleted or updated »

Un principe important de l’Event Sourcing est que les événements historiques sont généralement considérés comme **immutables**.

Dans une base CRUD traditionnelle, on peut effectuer :

```sql
UPDATE accounts
SET balance = 1600
WHERE id = 123;
```

Avec Event Sourcing, on évite généralement de modifier un événement historique.

Par exemple, si l’historique contient :

```text
Deposit 500
```

on ne le remplace pas directement par :

```text
Deposit 300
```

On ajoute plutôt un nouvel événement représentant la correction ou l’opération suivante.

Par exemple :

```text
Deposit 500
Withdraw 200
```

Les événements précédents restent donc présents.

Cela permet de préserver l’historique :

```text
Event 1
   ↓
Event 2
   ↓
Event 3
   ↓
Event 4
```

Chaque nouvel événement vient enrichir l’historique plutôt que remplacer les événements précédents.

> **L’historique est append-only : on ajoute principalement de nouveaux événements au lieu de modifier ou supprimer les anciens.**

---

# 8. Pourquoi parler de « CR » plutôt que de « CRUD » ?

Dans une application CRUD classique :

```text
C → Create
R → Read
U → Update
D → Delete
```

Avec Event Sourcing, le stockage des événements fonctionne principalement comme un journal auquel on ajoute de nouvelles entrées :

```text
Create Event
      ↓
Read Events
      ↓
Create Event
      ↓
Read Events
      ↓
...
```

On évite généralement les opérations :

```text
UPDATE
DELETE
```

sur l’historique métier.

Cela donne une structure conceptuellement proche de :

```text
Event 1
Event 2
Event 3
Event 4
Event 5
...
```

L’historique reste ainsi disponible pour reconstruire l’état.

⚠️ Il faut cependant nuancer : **Event Sourcing ne signifie pas qu’aucune donnée ne peut jamais être supprimée ou modifiée dans tout le système**. La règle concerne principalement le journal d’événements qui constitue la source de vérité métier.

---

# 9. Le problème : un nombre très important d’événements

L’Event Sourcing introduit naturellement une problématique de volumétrie.

Imaginons un compte bancaire ayant accumulé :

```text
1 000 événements
100 000 événements
10 000 000 événements
1 000 000 000 événements
```

Si nous devons reconstruire l’état depuis le premier événement à chaque lecture :

```text
Event 1
   ↓
Event 2
   ↓
Event 3
   ↓
...
   ↓
Event 1 000 000 000
   ↓
Balance
```

cela peut devenir coûteux en temps et en ressources.

Une stratégie courante consiste donc à utiliser des **snapshots**.

---

# 10. Solution : les snapshots

Un snapshot consiste à enregistrer périodiquement une représentation de l’état à un instant donné.

Par exemple :

```text
01 Janvier
    ↓
10 000 événements
    ↓
Snapshot
balance = 50 000 €
```

Puis de nouveaux événements sont enregistrés :

```text
Deposit 100
Withdraw 50
Deposit 200
```

Pour reconstruire l’état actuel, il n’est plus nécessaire de rejouer les 10 000 événements précédents.

On repart du snapshot :

```text
Snapshot
50 000
   +
100
   -
50
   +
200
────────
50 250 €
```

Le processus devient :

```text
Snapshot
    +
Événements récents
    ↓
État actuel
```

---

# 11. Exemple TypeScript avec snapshot

Nous pouvons représenter l’état du compte ainsi :

```ts
type AccountState = {
  balance: number;
};
```

Le snapshot :

```ts
const snapshot: AccountState = {
  balance: 50_000,
};
```

Les nouveaux événements :

```ts
const newEvents: AccountEvent[] = [
  { type: 'DEPOSIT', amount: 100 },
  { type: 'WITHDRAW', amount: 50 },
  { type: 'DEPOSIT', amount: 200 },
];
```

Nous pouvons alors reconstruire le solde :

```ts
const currentBalance =
  snapshot.balance + calculateBalance(newEvents);

console.log(currentBalance); // 50250
```

Le principe est donc :

```text
             ┌──────────────────┐
             │     Snapshot     │
             │   balance: 50k   │
             └────────┬─────────┘
                      │
                      +
                      │
             ┌────────▼─────────┐
             │   New Events     │
             │   +100           │
             │   -50            │
             │   +200           │
             └────────┬─────────┘
                      │
                      ▼
                  50 250 €
```

Le snapshot est donc une **optimisation de lecture/reconstruction**, et non un remplacement de l’historique des événements.

---

# 12. Le stockage et la conservation de l’historique

L’Event Sourcing repose sur l’idée qu’il peut être pertinent de conserver une quantité importante de données historiques.

Avec l’augmentation des capacités de stockage modernes, conserver un historique détaillé peut être acceptable selon le contexte et les contraintes du système.

On peut ainsi privilégier :

```text
Beaucoup de stockage
        ↓
Historique complet
        ↓
État reconstructible
```

plutôt que :

```text
État actuel uniquement
        ↓
Modifications successives
        ↓
Perte potentielle de l'historique
```

Cela ne signifie toutefois pas qu'il faut conserver **indéfiniment toutes les données sans politique de rétention**. Les contraintes de confidentialité, de réglementation, de coût et de performance doivent être prises en compte.

---

# 13. Et la concurrence ?

L’Event Sourcing peut également modifier la manière dont on raisonne sur les modifications concurrentes.

Dans une approche classique, deux processus peuvent essayer de modifier simultanément la même valeur :

```text
balance = 1000
```

Processus A :

```text
1000 → 1100
```

Processus B :

```text
1000 → 800
```

Sans mécanisme de concurrence approprié, l'une des modifications peut écraser l'autre.

Avec Event Sourcing, les opérations sont représentées comme des événements :

```text
Event A → Deposit 100
Event B → Withdraw 200
```

L'historique conserve alors les deux intentions métier.

Cependant, **Event Sourcing ne supprime pas les problèmes de concurrence**.

Il faut toujours gérer notamment :

* l'ordre des événements ;
* la concurrence optimiste ;
* les conflits ;
* les transactions ;
* la cohérence ;
* les versions des agrégats ;
* les règles métier empêchant certaines opérations.

Par exemple, deux retraits simultanés ne doivent pas nécessairement être acceptés simplement parce qu'ils sont représentés par deux événements.

---

# 14. Un exemple particulièrement parlant : Git

Un exemple concret permettant de comprendre l’idée est **Git**.

Imaginez que nous conservions uniquement l'état actuel d'un fichier :

```text
app.ts
   ↓
version actuelle
```

Nous perdons alors une grande partie de l'historique.

Git conserve au contraire une succession de commits :

```text
Commit 1
   ↓
Commit 2
   ↓
Commit 3
   ↓
Commit 4
```

Chaque commit représente une étape de l'évolution du projet.

Conceptuellement, cela ressemble à :

```text
Event Sourcing

Events
  ↓
Apply Events
  ↓
Current State
```

et :

```text
Git

Commits
  ↓
Apply Changes
  ↓
Current Files
```

⚠️ Git n'est pas simplement un système d'Event Sourcing au sens strict : son modèle de données et ses mécanismes sont plus complexes. Il constitue néanmoins une **bonne analogie conceptuelle** pour comprendre l'idée d'un historique permettant de retrouver des états antérieurs.

---

# 15. Event Sourcing et immutabilité

L’Event Sourcing constitue une application particulièrement intéressante du concept d’**immutabilité**.

Dans une approche mutable :

```text
balance = 1000

       ↓ mutation

balance = 1200

       ↓ mutation

balance = 900
```

La valeur précédente est remplacée.

Dans une approche Event Sourcing :

```text
Deposit 1000
      +
Deposit 200
      +
Withdraw 300
```

Les événements précédents restent disponibles.

L'état est ensuite calculé :

```text
Events
   ↓
Reducer / Aggregate
   ↓
State
```

On peut donc avoir une vision fonctionnelle :

```text
Stateₙ₊₁ = apply(Stateₙ, Eventₙ₊₁)
```

Chaque événement transforme logiquement l'état précédent en un nouvel état.

---

# 16. Event Sourcing et Projection

Dans une architecture réelle, on distingue souvent le **journal d'événements** de la manière dont les données sont présentées ou consommées par l'application.

On peut avoir :

```text
                 Event Store
                     │
                     │ Events
                     ▼
                Event Handler
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
       Account     Balance    Reporting
      Projection  Projection  Projection
```

Les événements constituent la source de vérité.

Les projections construisent ensuite des modèles optimisés pour différents besoins de lecture.

Par exemple :

```text
Events
  │
  ├──► Balance Projection
  │
  ├──► Transaction History
  │
  └──► Reporting Projection
```

C'est l'une des raisons pour lesquelles Event Sourcing est souvent associé à **CQRS (Command Query Responsibility Segregation)**, même si les deux concepts sont distincts et peuvent être utilisés séparément.

---

# 17. Architecture globale

Une représentation simplifiée d'une architecture Event Sourcing peut être :

```text
                 Command
                    │
                    ▼
              Domain / Aggregate
                    │
                    ▼
             Domain Events
                    │
                    ▼
              ┌─────────────┐
              │ Event Store │
              └──────┬──────┘
                     │
                     │ Events
                     ▼
               Projections
                     │
                     ▼
                Read Models
                     │
                     ▼
                Application
```

Le flux principal devient donc :

```text
Command
   ↓
Business Rules
   ↓
Event
   ↓
Event Store
   ↓
Projection
   ↓
Current State / Read Model
```

---

# 18. Résumé

La différence fondamentale entre une approche CRUD classique et Event Sourcing peut être résumée ainsi :

### Approche classique

```text
Transaction
     ↓
Modifier l'état
     ↓
balance = 1600
```

Le **state courant** est la donnée principale.

### Event Sourcing

```text
Deposit 1000
Deposit 500
Withdraw 200
Deposit 300
       ↓
Apply Events
       ↓
balance = 1600
```

Les **événements constituent la source de vérité** et l'état courant est reconstruit à partir d'eux.

---

# 🧠 À retenir

> **Event Sourcing consiste à conserver les événements qui représentent les changements métier comme source de vérité, puis à reconstruire l'état courant en appliquant ces événements.**

Les concepts essentiels sont donc :

```text
                 EVENT SOURCING
                       │
                       ▼
              ┌─────────────────┐
              │ Events immuables│
              └────────┬────────┘
                       │
                       ▼
                  Event Store
                       │
                       ▼
                  Projection
                       │
                       ▼
                 Current State
```

L'Event Sourcing est ainsi directement lié à l'immutabilité :

> **Au lieu de constamment écraser une valeur pour représenter le nouvel état, on conserve les changements sous forme d'événements et on reconstruit l'état à partir de cet historique.**

Cette approche apporte une forte **traçabilité**, mais introduit également des complexités supplémentaires : gestion de l'ordre des événements, concurrence, versionnement, projections, snapshots, évolution des événements et stratégie de rétention des données.

Elle doit donc être considérée comme un **choix architectural**, et non comme un remplacement systématique du CRUD.
