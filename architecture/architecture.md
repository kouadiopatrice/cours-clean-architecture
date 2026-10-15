# Chapitre 15 — Qu’est-ce que l’architecture ?

Le mot « architecture » évoque des images de puissance et de mystère. Il nous fait penser à des décisions lourdes de conséquences et à une grande maîtrise technique. L’architecture logicielle représente le sommet de l’accomplissement technique. Lorsque nous pensons à un architecte logiciel, nous imaginons quelqu’un qui détient du pouvoir et qui inspire le respect. Quel jeune développeur logiciel en devenir n’a jamais rêvé de devenir un jour architecte logiciel ?

Mais qu’est-ce que l’architecture logicielle ? Que fait un architecte logiciel, et à quel moment le fait-il ?

Tout d’abord, un architecte logiciel est un programmeur, et il continue de programmer. Ne croyez jamais cette idée trompeuse selon laquelle les architectes logiciels s’éloignent du code pour se concentrer sur des sujets de plus haut niveau. Ce n’est pas le cas ! Les architectes logiciels sont les meilleurs programmeurs et continuent de prendre en charge des tâches de programmation, tout en guidant le reste de l’équipe vers une conception qui maximise la productivité. Ils n’écrivent peut-être pas autant de code que les autres programmeurs, mais ils continuent à programmer. Ils le font parce qu’ils ne peuvent pas accomplir correctement leur travail s’ils ne font pas eux-mêmes l’expérience des problèmes qu’ils créent pour les autres programmeurs.

L’architecture d’un système logiciel est la forme que lui donnent ceux qui le construisent. Cette forme se manifeste dans la division du système en composants, dans l’organisation de ces composants et dans les moyens par lesquels ils communiquent entre eux.

Cette organisation a pour objectif de faciliter le développement, le déploiement, l’exploitation et la maintenance du système logiciel.

**La stratégie qui permet d’y parvenir consiste à conserver autant d’options ouvertes que possible, aussi longtemps que possible.**

Cette affirmation vous a peut-être surpris. Vous pensiez peut-être que l’objectif de l’architecture logicielle était de faire fonctionner correctement le système. Bien entendu, nous voulons que le système fonctionne correctement, et son architecture doit absolument soutenir cet objectif, qui compte parmi les priorités les plus importantes.

Cependant, l’architecture d’un système a très peu d’influence sur le fait qu’il fonctionne ou non. Il existe de nombreux systèmes dont l’architecture est médiocre, mais qui fonctionnent parfaitement. Leurs difficultés ne se situent pas au niveau de leur fonctionnement, mais plutôt au niveau du déploiement, de la maintenance et du développement continu.

Cela ne signifie pas que l’architecture ne joue aucun rôle dans le bon comportement du système. Elle en joue bien un, et ce rôle est essentiel. Toutefois, ce rôle est passif et d’ordre structurel, plutôt qu’actif ou indispensable au comportement lui-même. Il existe peu, voire aucune, option liée au **comportement** qu’une architecture puisse laisser ouverte.

L’objectif principal de l’architecture est de soutenir le cycle de vie du système. Une bonne architecture rend le système facile à comprendre, à développer, à maintenir et à déployer. Le but ultime est de réduire au minimum le coût total du système sur toute sa durée de vie et de maximiser la productivité des programmeurs.

## Développement

Un système logiciel difficile à développer n’a probablement pas devant lui une longue durée de vie saine. L’architecture doit donc faciliter le développement du système par les équipes qui en ont la charge.

La structure des équipes entraîne des choix architecturaux différents. D’un côté, une petite équipe de cinq développeurs peut travailler efficacement ensemble sur un système monolithique, sans composants ni interfaces clairement définis. En réalité, au début du développement, cette équipe pourrait considérer les contraintes imposées par une architecture stricte comme un obstacle. C’est probablement l’une des raisons pour lesquelles tant de systèmes sont dépourvus d’une bonne architecture : ils ont été lancés sans architecture, parce que l’équipe était petite et ne voulait pas être gênée par une structure supplémentaire.

À l’inverse, un système développé par cinq équipes différentes, chacune composée de sept développeurs, ne peut progresser que si le système est divisé en composants bien définis, dotés d’interfaces fiables et stables. Si l’on ne tient compte d’aucun autre facteur, l’architecture de ce système évoluera probablement vers cinq composants, un par équipe.

Une architecture prévoyant un composant par équipe n’est pas nécessairement la meilleure pour le déploiement, l’exploitation et la maintenance du système. Néanmoins, c’est vers cette architecture que les équipes tendront si elles sont guidées uniquement par les délais de développement.

## Déploiement

Pour être efficace, un système logiciel doit pouvoir être déployé. Plus le déploiement coûte cher, moins le système est utile. L’architecture logicielle devrait donc viser à rendre le système facilement déployable **en une seule action**.

Malheureusement, la stratégie de déploiement est rarement prise en compte au début du développement. Cela conduit à des architectures qui facilitent le développement, mais rendent le déploiement très difficile.

Par exemple, au début du développement d’un système, les développeurs peuvent décider d’utiliser une « architecture microservices ». Ils peuvent constater que cette approche facilite le développement, car les frontières entre les composants sont très nettes et les interfaces relativement stables. Cependant, au moment du déploiement, ils peuvent découvrir que le nombre de microservices est devenu difficile à gérer. La configuration des connexions entre eux et la gestion de l’ordre dans lequel ils doivent démarrer peuvent également devenir une importante source d’erreurs.

Si les architectes avaient pris en compte les questions de déploiement dès le début, ils auraient peut-être choisi un nombre plus réduit de services, une combinaison de services et de composants exécutés dans le même processus, ainsi qu’un moyen plus intégré de gérer leurs interconnexions.

## Exploitation

L’influence de l’architecture sur l’exploitation du système est généralement moins spectaculaire que son influence sur le développement, le déploiement et la maintenance. Presque toutes les difficultés d’exploitation peuvent être résolues en ajoutant davantage de matériel, sans modifier radicalement l’architecture logicielle.

Nous avons d’ailleurs vu cette situation se reproduire de nombreuses fois. Des systèmes logiciels dont l’architecture est inefficace peuvent souvent être rendus performants simplement en ajoutant du stockage et des serveurs. Le matériel étant peu coûteux et le travail humain étant cher, les architectures qui compliquent l’exploitation coûtent moins cher que celles qui entravent le développement, le déploiement et la maintenance.

Cela ne veut pas dire qu’une architecture adaptée aux besoins d’exploitation n’est pas souhaitable. Bien au contraire ! Cela signifie simplement que l’équation des coûts donne davantage de poids au développement, au déploiement et à la maintenance.

Cela étant dit, l’architecture joue un autre rôle dans l’exploitation du système : une bonne architecture rend visibles les besoins opérationnels du système.

On pourrait dire plus précisément que l’architecture rend le fonctionnement du système évident pour les développeurs. **L’architecture doit révéler le fonctionnement opérationnel.** Elle doit mettre au premier plan les cas d’utilisation, les fonctionnalités et les comportements attendus, afin qu’ils deviennent des repères clairement visibles pour les développeurs. Cela facilite la compréhension du système et contribue donc fortement au développement et à la maintenance.

## Maintenance

Parmi tous les aspects d’un système logiciel, la maintenance est le plus coûteux. Le flot incessant de nouvelles fonctionnalités, auquel s’ajoutent inévitablement les défauts et leurs corrections, consomme énormément de ressources humaines.

Les principaux coûts de maintenance sont liés à l’exploration du code existant et aux risques. L’exploration consiste à parcourir le logiciel existant pour déterminer où et comment ajouter une fonctionnalité ou corriger un défaut. Lorsqu’on effectue ces modifications, il existe toujours un risque de provoquer involontairement de nouveaux défauts, ce qui augmente le coût lié aux risques.

Une architecture soigneusement pensée réduit considérablement ces coûts. En divisant le système en composants et en isolant ces composants au moyen d’interfaces stables, on peut rendre les chemins d’évolution plus évidents et réduire fortement le risque de régressions involontaires.

## Préserver les options ouvertes

Comme nous l’avons expliqué dans un chapitre précédent, le logiciel possède deux types de valeur : la valeur de son comportement et la valeur de sa structure. La seconde est la plus importante, car c’est elle qui rend le logiciel **souple et modifiable**.

Le logiciel a été inventé parce que nous avions besoin d’un moyen de modifier rapidement et facilement le comportement des machines. Mais cette flexibilité dépend étroitement de la forme du système, de l’organisation de ses composants et de la manière dont ils sont reliés entre eux.

Pour préserver la souplesse du logiciel, il faut conserver autant d’options ouvertes que possible, aussi longtemps que possible. Quelles options devons-nous garder ouvertes ? **Ce sont les détails qui n’ont pas d’importance.**

Tous les systèmes logiciels peuvent être décomposés en deux grands éléments : les politiques métier et les détails. Les politiques regroupent les règles métier et les procédures. C’est là que réside la véritable valeur du système.

Les détails sont les éléments nécessaires pour permettre aux humains, aux autres systèmes et aux programmeurs de communiquer avec les politiques métier, mais qui n’influencent pas leur comportement. Ils comprennent notamment les périphériques d’entrée-sortie, les bases de données, les systèmes Web, les serveurs, les frameworks, les protocoles de communication, etc.

L’objectif de l’architecte est de donner au système une structure qui reconnaît les politiques métier comme son élément essentiel, tout en rendant les détails **sans incidence** sur ces politiques. Cela permet de **retarder et de différer** les décisions concernant ces détails.

Par exemple :

- Il n’est pas nécessaire de choisir un système de base de données dès les premières étapes du développement, car les politiques métier de haut niveau ne devraient pas dépendre de la base de données utilisée. Si l’architecte est prudent, ces politiques ne se soucieront pas de savoir si la base de données est relationnelle, distribuée, hiérarchique ou constituée de simples fichiers plats.

- Il n’est pas nécessaire de choisir un serveur Web dès le début, car les politiques métier de haut niveau ne devraient pas savoir qu’elles sont exposées sur le Web. Si elles ignorent HTML, AJAX, JSP, JSF et tous les autres sigles du développement Web, le choix de la technologie Web peut être repoussé à une étape ultérieure. En réalité, **il n’est même pas nécessaire de décider si le système sera accessible sur le Web**.

- Il n’est pas nécessaire d’adopter REST dès le début, car les politiques métier de haut niveau devraient rester indépendantes de l’interface avec le monde extérieur. Il n’est pas non plus nécessaire d’adopter immédiatement un framework de microservices ou de SOA (architecture orientée services). Là encore, les politiques métier de haut niveau ne devraient pas dépendre de ces choix.

- Il n’est pas nécessaire d’adopter un framework d’injection de dépendances dès le début, car les politiques métier de haut niveau ne devraient pas dépendre de la manière dont les dépendances sont résolues.

Vous comprenez l’idée : si vous pouvez développer les politiques métier de haut niveau sans vous engager trop tôt dans les détails qui les entourent, vous pouvez reporter ces décisions pendant longtemps. Et plus vous attendez avant de les prendre, plus vous disposez d’informations pour faire les bons choix.

Cela vous permet également d’expérimenter différentes solutions. Si une partie des politiques métier de haut niveau fonctionne déjà et ne dépend pas de la base de données, vous pouvez essayer de la connecter à plusieurs bases de données pour en vérifier l’adéquation et les performances. La même démarche s’applique aux systèmes Web, aux frameworks Web, voire au choix d’utiliser le Web lui-même.

Plus vous maintenez les options ouvertes longtemps, plus vous pouvez réaliser d’expériences, essayer de solutions et recueillir d’informations avant d’atteindre le moment où les décisions ne peuvent plus être différées.

Et si les décisions ont déjà été prises par quelqu’un d’autre ? Et si votre entreprise s’est engagée à utiliser une base de données, un serveur Web ou un framework particulier ? **Un bon architecte fait comme si la décision n’avait pas encore été prise** et conçoit le système de façon à ce qu’elle puisse toujours être différée ou modifiée aussi longtemps que possible.

**Un bon architecte maximise le nombre de décisions qu’il n’a pas encore prises.**

## Indépendance vis-à-vis des périphériques

Pour illustrer cette façon de penser, remontons aux années 1960, lorsque les ordinateurs étaient encore jeunes et que la plupart des programmeurs étaient des mathématiciens ou des ingénieurs issus d’autres disciplines — et qu’un tiers, voire davantage, étaient des femmes.

À cette époque, nous commettions beaucoup d’erreurs. Bien sûr, nous ne savions pas qu’il s’agissait d’erreurs. Comment aurions-nous pu le savoir ?

L’une de ces erreurs consistait à lier directement notre code aux périphériques d’entrée-sortie. Si nous voulions imprimer quelque chose, nous écrivions du code utilisant les instructions d’entrée-sortie qui contrôlaient l’imprimante. Notre code dépendait donc directement du périphérique.

Par exemple, lorsque j’écrivais des programmes pour le PDP-8 afin d’imprimer sur un téléimprimeur, j’utilisais un ensemble d’instructions machine de ce type :

```text
PRTCHR, 0
TSF
JMP .-1
TLS
JMP I PRTCHR
```

PRTCHR est une sous-routine qui imprime un caractère sur le téléimprimeur. Le zéro initial servait à stocker l’adresse de retour. (Ne me demandez pas pourquoi.)

L’instruction TSF sautait l’instruction suivante si le téléimprimeur était prêt à imprimer. S’il était occupé, TSF passait simplement à l’instruction `JMP .-1`, qui revenait à l’instruction TSF. Si le téléimprimeur était prêt, TSF sautait jusqu’à l’instruction TLS, qui envoyait le caractère contenu dans le registre A au téléimprimeur. L’instruction `JMP I PRTCHR` retournait ensuite à l’appelant.

Au début, cette stratégie fonctionnait parfaitement. Si nous devions lire des cartes perforées, nous écrivions du code qui communiquait directement avec le lecteur de cartes. Si nous devions perforer des cartes, nous écrivions du code qui pilotait directement la perforatrice. Les programmes fonctionnaient parfaitement. Comment aurions-nous pu savoir que c’était une erreur ?

Mais les grandes piles de cartes perforées étaient difficiles à gérer. Elles pouvaient être perdues, abîmées, mélangées, pliées ou tomber. Des cartes individuelles pouvaient disparaître et des cartes supplémentaires pouvaient être insérées. L’intégrité des données est donc devenue un problème important.

La bande magnétique a apporté une solution. Nous pouvions transférer sur bande les images des cartes. Si une bande magnétique tombe, les enregistrements ne se mélangent pas. On ne peut pas perdre accidentellement un enregistrement ni en insérer un vide simplement en manipulant la bande. La bande est beaucoup plus sûre. Elle est également plus rapide à lire et à écrire, et il est très facile d’en faire des copies de sauvegarde.

Malheureusement, tous nos logiciels avaient été écrits pour manipuler des lecteurs et des perforatrices de cartes. Ces programmes ont dû être réécrits pour utiliser des bandes magnétiques. Ce fut un travail considérable.

À la fin des années 1960, nous avions retenu la leçon et inventé **l’indépendance vis-à-vis des périphériques**. Les systèmes d’exploitation de l’époque ont abstrait les périphériques d’entrée-sortie sous la forme de fonctions logicielles qui manipulaient des enregistrements unitaires semblables à ceux des cartes. Les programmes appelaient des services du système d’exploitation qui travaillaient avec des périphériques abstraits d’enregistrements unitaires. Les opérateurs pouvaient indiquer au système d’exploitation si ces services abstraits devaient être reliés à des lecteurs de cartes, à des bandes magnétiques ou à tout autre périphérique de ce type.

Dès lors, un même programme pouvait lire et écrire des cartes ou des bandes **sans aucune modification**. Le principe ouvert/fermé (Open–Closed Principle) venait de naître, même s’il ne portait pas encore ce nom.

## Courrier publicitaire

À la fin des années 1960, j’ai travaillé pour une entreprise qui imprimait des courriers publicitaires pour ses clients. Ceux-ci nous envoyaient des bandes magnétiques contenant des enregistrements unitaires avec les noms et adresses de leurs clients. Nous écrivions ensuite des programmes qui imprimaient des publicités personnalisées.

Vous connaissez ce genre de courrier :

> Bonjour Monsieur Martin,
>
> Félicitations !
>
> Vous avez été **CHOISI**, parmi toutes les personnes qui vivent sur Witchwood Lane, pour participer à notre nouvelle offre exceptionnelle, disponible pour une durée limitée…

Les clients nous envoyaient de longs rouleaux de lettres préimprimées, contenant tout le texte à l’exception du nom, de l’adresse et de tout autre élément qu’ils souhaitaient personnaliser. Nous écrivions des programmes qui extrayaient les noms, les adresses et les autres éléments depuis la bande magnétique, puis les imprimaient exactement aux emplacements prévus sur les formulaires.

Ces rouleaux de lettres pesaient environ 225 kg et contenaient des milliers de courriers. Les clients nous en envoyaient des centaines, et nous imprimions chaque courrier individuellement.

Au départ, nous utilisions un IBM 360 pour imprimer sur son unique imprimante de ligne. Nous pouvions imprimer quelques milliers de lettres par poste. Malheureusement, cela immobilisait une machine extrêmement coûteuse pendant très longtemps. À cette époque, la location d’un IBM 360 coûtait des dizaines de milliers de dollars par mois.

Nous avons donc demandé au système d’exploitation d’utiliser une bande magnétique plutôt que l’imprimante de ligne. Nos programmes n’avaient pas besoin d’être modifiés, puisqu’ils utilisaient les abstractions d’entrée-sortie fournies par le système d’exploitation.

L’IBM 360 pouvait produire une bande complète en une dizaine de minutes, suffisamment pour imprimer plusieurs rouleaux de lettres. Les bandes étaient ensuite sorties de la salle informatique et montées sur des lecteurs de bandes reliés à des imprimantes hors ligne. Nous en avions cinq, que nous faisions fonctionner 24 heures sur 24, sept jours sur sept, pour imprimer des centaines de milliers de courriers publicitaires chaque semaine.

La valeur de l’indépendance vis-à-vis des périphériques était considérable ! Nous pouvions écrire nos programmes sans connaître ni nous soucier du périphérique qui serait utilisé. Nous pouvions les tester sur l’imprimante de ligne locale connectée à l’ordinateur. Ensuite, nous pouvions demander au système d’exploitation d’« imprimer » sur bande magnétique et produire des centaines de milliers de formulaires.

Nos programmes avaient une structure qui séparait les politiques métier des détails. La politique métier consistait à formater les enregistrements contenant les noms et les adresses. Le détail était le périphérique utilisé. Nous avions différé la décision concernant le périphérique à employer.

## Adressage physique

Au début des années 1970, j’ai travaillé sur un grand système comptable pour un syndicat local de camionneurs. Nous disposions d’un lecteur de disque de 25 Mo sur lequel nous stockions les enregistrements des agents, des employeurs et des membres. Comme ces différents enregistrements avaient des tailles différentes, nous avions formaté les premiers cylindres du disque de manière à ce que chaque secteur corresponde à la taille d’un enregistrement d’agent. Les cylindres suivants contenaient des secteurs adaptés aux enregistrements d’employeurs. Les derniers cylindres étaient réservés aux enregistrements des membres.

Nous avions écrit notre logiciel de façon à ce qu’il connaisse la structure détaillée du disque. Il savait que le disque comportait 200 cylindres et 10 têtes, et que chaque cylindre contenait plusieurs dizaines de secteurs par tête. Il savait quels cylindres contenaient les agents, les employeurs et les membres. Toutes ces informations étaient inscrites en dur dans le code.

Nous avions conservé un index sur le disque pour retrouver chacun des agents, employeurs et membres. Cet index se trouvait dans un autre ensemble de cylindres spécialement formatés. L’index des agents était constitué d’enregistrements contenant l’identifiant d’un agent ainsi que les numéros de cylindre, de tête et de secteur de son enregistrement. Les employeurs et les membres disposaient d’index similaires. Les membres étaient également organisés dans une liste doublement chaînée sur le disque. Chaque enregistrement de membre contenait les numéros de cylindre, de tête et de secteur des enregistrements des membres précédent et suivant.

Que se serait-il passé si nous avions dû remplacer le disque par un nouveau modèle doté de davantage de têtes, de cylindres ou de secteurs par cylindre ? Nous aurions dû écrire un programme spécial pour lire les anciennes données et les transférer vers le nouveau disque, en convertissant tous les numéros de cylindre, de tête et de secteur. Nous aurions également dû modifier toutes les informations inscrites en dur dans notre code — et elles se trouvaient partout ! Toutes les règles métier connaissaient en détail le système d’adressage par cylindre, tête et secteur.

Un jour, un programmeur plus expérimenté a rejoint notre équipe. Lorsqu’il a découvert ce que nous avions fait, il a blêmi et nous a regardés avec stupeur, comme si nous étions des extraterrestres. Puis il nous a gentiment conseillé de remplacer notre système d’adressage par un système utilisant des adresses relatives.

Notre collègue, plus sage que nous, nous a suggéré de considérer le disque comme un immense tableau linéaire de secteurs, chacun étant identifié par un entier séquentiel. Nous pouvions alors écrire une petite routine de conversion connaissant la structure physique du disque et capable de traduire à la volée une adresse relative en un numéro de cylindre, de tête et de secteur.

Heureusement, nous avons suivi son conseil. Nous avons modifié les politiques métier de haut niveau du système afin qu’elles ne dépendent plus de la structure physique du disque. Cela nous a permis de dissocier le choix de la structure du lecteur de disque de l’application.

## Conclusion

Ces deux récits illustrent, à petite échelle, un principe que les architectes appliquent à grande échelle. Les bons architectes séparent soigneusement les détails des politiques métier, puis les découplent si complètement que les politiques ne connaissent pas les détails et n’en dépendent d’aucune manière. Ils conçoivent les politiques métier de telle sorte que les décisions relatives aux détails puissent être retardées et reportées aussi longtemps que possible.
