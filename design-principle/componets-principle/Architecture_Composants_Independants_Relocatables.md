Ce passage de *Clean Architecture* explique **pourquoi les composants logiciels modernes doivent être indépendants et relocalisables**. L'exemple du PDP-8 est ancien, mais l'idée est directement applicable à TypeScript/Angular/React.

Le point central est :

> **Un composant doit pouvoir évoluer, être compilé, testé et remplacé sans obliger tout le reste du système à changer.**

# 1. Le problème à l'époque : tout était physiquement lié

Dans l'ancien programme PDP-8, on avait quelque chose comme :

```
Adresse mémoire

0200 ┌─────────────────────┐
     │ Application         │
     │                     │
     │ START               │
     │                     │
     │ GETSTR              │
     │                     │
     │ PUTSTR              │
     ├─────────────────────┤
3000 │ BUFR                │
     ├─────────────────────┤
     │ GETSTR              │
     └─────────────────────┘
```

Le programme disait explicitement :

```
*200
```

Cela signifiait approximativement :

> « Charge mon programme à l'adresse mémoire 200. »

Le problème était que **les adresses étaient importantes pour le programme**.

Si `GETSTR` était à une certaine adresse, le programme devait savoir où il se trouvait.

# 2. Comment les bibliothèques fonctionnaient

À cette époque, si tu voulais utiliser une fonction `GETSTR`, tu prenais son code source et tu le mettais avec ton application.

Conceptuellement :

```
application.pdp8
    +
GETSTR.pdp8
    +
PUTSTR.pdp8
    ↓
compilation
    ↓
un seul programme
```

En termes modernes, imagine :

```
// application.ts

function getString(): string {
  // énorme bibliothèque copiée ici
}

function putString(value: string): void {
  // autre bibliothèque copiée ici
}

function main() {
  const value = getString();
  putString(value);
}
```

Tout est mélangé dans le même programme.

# 3. Pourquoi c'était un problème ?

À l'époque :

- la mémoire était très limitée ;
- les compilateurs étaient lents ;
- les périphériques étaient lents ;
- les programmes devenaient de plus en plus gros.

Donc si tu avais :

```
Application
+
Bibliothèque de fonctions
+
Encore une bibliothèque
+
Encore une bibliothèque
```

le temps de compilation augmentait énormément.

L'auteur explique donc qu'ils ont commencé à **compiler les bibliothèques séparément**.

# 4. Première évolution : compiler séparément

Au lieu de :

```
Source application
       +
Source library
       ↓
   compilation
       ↓
    programme
```

on fait :

```
Library source
      ↓
  compilation
      ↓
Library binary

Application source
      ↓
  compilation
      ↓
Application binary
```

Puis au moment de l'exécution :

```
Library binary
      +
Application binary
      ↓
     RAM
```

C'est déjà très proche de notre monde moderne.

# 5. Le problème de l'adresse fixe

Supposons que la bibliothèque soit chargée à :

```
2000
```

et que ton application soit chargée autour de :

```
0000 → 1777
```

On a donc :

```
0000
┌────────────────────┐
│ Application        │
│                    │
│                    │
└────────────────────┘
1777

2000
┌────────────────────┐
│ Library            │
│                    │
└────────────────────┘
```

Ça fonctionne.

Mais ton application grandit.

Elle dépasse :

```
1777
```

Elle commence donc à entrer dans l'espace de la bibliothèque.

Tu arrives à une situation du genre :

```
0000
┌────────────────────┐
│ Application        │
│                    │
└────────────────────┘

2000
┌────────────────────┐
│ Library            │
└────────────────────┘

?????
┌────────────────────┐
│ Suite application  │
└────────────────────┘
```

L'application doit être **coupée en morceaux** pour éviter la bibliothèque.

C'est exactement le problème décrit dans le livre.

# 6. Le concept important : Relocatability

Le chapitre arrive donc à un concept important :

> **Relocatability**

Une bibliothèque/composant ne devrait pas dépendre d'une adresse physique particulière.

Autrement dit :

```
❌ Library doit être à l'adresse 2000

✅ Library peut être chargée n'importe où
```

Aujourd'hui, les systèmes modernes ont des mécanismes beaucoup plus sophistiqués pour résoudre cela.

Mais le principe architectural reste intéressant :

> **Un composant ne doit pas être inutilement couplé à son environnement.**

# 7. Maintenant, faisons le parallèle avec TypeScript

Imagine une application React/Angular.

Tu as :

```
Application
   ↓
Auth
   ↓
API
```

Une mauvaise architecture pourrait être :

```
export class LoginUseCase {
  async execute(email: string, password: string) {

    const response = await axios.post(
      'https://api.example.com/login',
      {
        email,
        password
      }
    );

    localStorage.setItem(
      'token',
      response.data.token
    );

    return response.data;
  }
}
```

Ici `LoginUseCase` connaît :

- Axios
- l'URL de l'API
- localStorage
- le format HTTP
- le format du backend

Donc :

```
LoginUseCase
     ↓
   Axios
     ↓
HTTP API
```

Le composant métier est fortement dépendant de son environnement.

# 8. Approche par composants indépendants

On peut créer une interface :

```
export interface AuthRepository {
  login(
    email: string,
    password: string
  ): Promise<AuthUser>;
}
```

Le use case utilise uniquement cette interface :

```
export class LoginUseCase {

  constructor(
    private readonly authRepository: AuthRepository
  ) {}

  execute(
    email: string,
    password: string
  ): Promise<AuthUser> {

    return this.authRepository.login(
      email,
      password
    );
  }
}
```

Maintenant :

```
             CORE
┌────────────────────────────┐
│                            │
│      LoginUseCase           │
│             │              │
│             ↓              │
│      AuthRepository        │
│        interface           │
│                            │
└─────────────┬──────────────┘
              │
              │ implementation
              ↓
┌──────────────────────────────┐
│ AuthRepositoryBackend        │
│                              │
│ Axios                        │
│ HTTP                         │
│ API                          │
└──────────────────────────────┘
```

Le `LoginUseCase` ne sait même pas si tu utilises :

```
Axios
fetch
Angular HttpClient
GraphQL
REST
mock
local database
```

# 9. C'est ça l'idée moderne derrière le texte

L'ancien monde avait :

```
Application
    ↓
Adresse mémoire précise
    ↓
Library
```

Le problème était :

```
Application dépend physiquement de Library
```

Dans une architecture moderne, on cherche plutôt :

```
Application
    ↓
Interface
    ↑
Adapter
    ↓
Infrastructure
```

Donc :

```
        BUSINESS
           │
           │
           ▼
     ┌─────────────┐
     │ Interface   │
     └──────┬──────┘
            ▲
            │
            │ implements
            │
     ┌──────┴──────┐
     │   Adapter   │
     └──────┬──────┘
            │
            ▼
       Infrastructure
```

# 10. Exemple concret avec Angular 17

Prenons une API de contrats.

### ❌ Mauvais couplage

```
@Injectable()
export class ContractService {

  constructor(
    private readonly http: HttpClient
  ) {}

  getContract(
    contractNumber: string
  ) {
    return this.http.get(
      `/api/contracts/${contractNumber}`
    );
  }
}
```

Ici ton domaine dépend directement d'Angular.

### ✅ Avec une interface

Dans le domaine :

```
export interface ContractRepository {
  getContract(
    contractNumber: string
  ): Observable<Contract>;
}
```

Puis l'adapter Angular :

```
@Injectable()
export class ContractRepositoryHttp
  implements ContractRepository {

  constructor(
    private readonly http: HttpClient
  ) {}

  getContract(
    contractNumber: string
  ): Observable<Contract> {

    return this.http.get<Contract>(
      `/api/contracts/${contractNumber}`
    );
  }
}
```

Ton use case :

```
export class GetContractUseCase {

  constructor(
    private readonly repository: ContractRepository
  ) {}

  execute(
    contractNumber: string
  ) {
    return this.repository.getContract(
      contractNumber
    );
  }
}
```

Maintenant :

```
Angular HttpClient
       │
       ▼
ContractRepositoryHttp
       │
       │ implements
       ▼
ContractRepository
       ▲
       │
       │ depends on
       │
GetContractUseCase
```

# 11. Pourquoi l'auteur dit "independently developable"

C'est probablement la partie la plus importante de ton extrait.

Un composant bien conçu devrait pouvoir être développé indépendamment.

Par exemple :

```
Team A
  ↓
LoginUseCase
```

et :

```
Team B
  ↓
AuthRepositoryBackend
```

peuvent travailler séparément si le contrat est défini :

```
interface AuthRepository {
  login(
    email: string,
    password: string
  ): Promise<AuthUser>;
}
```

L'équipe A n'a pas besoin de connaître l'implémentation de l'équipe B.

# 12. Et ça améliore énormément les tests

Grâce à l'interface :

```
class FakeAuthRepository
  implements AuthRepository {

  async login(
    email: string,
    password: string
  ): Promise<AuthUser> {

    return {
      id: '1',
      email,
      name: 'Test User'
    };
  }
}
```

Test :

```
describe('LoginUseCase', () => {

  it('should login a user', async () => {

    const repository =
      new FakeAuthRepository();

    const useCase =
      new LoginUseCase(repository);

    const user =
      await useCase.execute(
        'test@test.com',
        'password'
      );

    expect(user.email)
      .toBe('test@test.com');
  });

});
```

Le test ne nécessite :

```
❌ Backend
❌ API
❌ Axios
❌ Angular HttpClient
❌ Database
```

Le composant métier est donc **indépendamment testable**.

# 13. Résumé du passage

Le texte raconte en réalité l'évolution suivante :

```
ANNÉES 1960
──────────────────────────────

Application
     +
Library
     ↓
tout compilé ensemble

        ↓

Library compilée séparément
        +
Application compilée séparément
        ↓
mais avec des adresses fixes

        ↓

Problème :
les applications grossissent
et les bibliothèques grossissent

        ↓

Besoin de composants
indépendants et relocatables

        ↓

ARCHITECTURE MODERNE
──────────────────────────────

       Application
            │
            ▼
        Interface
            ▲
            │
         Adapter
            │
            ▼
      Infrastructure
```

### La leçon à retenir pour ton architecture TypeScript

Quand tu construis ton architecture `core / application / details` :

```
core
 ├── entities
 ├── useCases
 └── repositories (interfaces)
                 ▲
                 │
details           │
 ├── http         │
 ├── storage      │
 └── repositories ─┘
```

**Le `core` dit ce dont il a besoin.**

**L'adapter `details` explique comment le fournir.**

C'est exactement l'esprit de l'**Interface Adapter / Ports & Adapters**, et c'est une des bases qui permettent d'obtenir des composants **indépendamment développables, testables et remplaçables**.
