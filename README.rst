Quand se passe-t-il...
======================

Ce référentiel est une tentative de répondre à la vieille question d'entrevue : "Que se passe-t-il lorsque vous tapez google.com dans la barre d'adresse de votre navigateur et appuyez sur Entrée ?"

Sauf qu'au lieu de la version habituelle, nous allons essayer de répondre à cette question en détail autant que possible. Pas de saut d'informations.

C'est un processus collaboratif, alors plongez-vous et essayez d'aider ! Il y a de nombreuses informations manquantes, attendant que vous les ajoutiez ! Alors, envoyez-nous une demande de tirage, s'il vous plaît !

Tout cela est sous licence selon les termes de la licence `Creative Commons Zero`_.

Lisez ceci en `简体中文`_ (chinois simplifié), en `日本語`_ (japonais), en `한국어`_ (coréen) et en `espagnol`_. NOTE : ces traductions n'ont pas été examinées par les responsables du projet alex/what-happens-when.

Table des matières
======================

.. contents::
   :backlinks: none
   :local:

La touche "g" est pressée
--------------------------
Les sections suivantes expliquent les actions physiques du clavier et les interruptions du système d'exploitation. Lorsque vous appuyez sur la touche "g", le navigateur reçoit l'événement et les fonctions d'auto-complétion entrent en jeu.
En fonction de l'algorithme de votre navigateur et de votre mode de navigation privée/incognito, diverses suggestions vous seront présentées dans le menu déroulant en dessous de la barre d'URL. La plupart de ces algorithmes trient et priorisent les résultats en fonction de l'historique de recherche, des signets, des cookies et des recherches populaires sur Internet dans son ensemble. Au fur et à mesure que vous tapez "google.com", de nombreux blocs de code s'exécutent et les suggestions se peaufinent à chaque pression de touche. Il peut même suggérer "google.com" avant que vous n'ayez fini de le taper.

La touche "Entrée" atteint le fond
-------------------------------------

Pour choisir un point zéro, choisissons la touche "Entrée" du clavier qui atteint le bas de sa plage. À ce moment-là, un circuit électrique spécifique à la touche "Entrée" est fermé (directement ou capacitivement). Cela permet à un faible courant de circuler dans la logique du clavier, qui scanne l'état de chaque commutateur de touche, élimine le bruit électrique résultant de la fermeture intermittente rapide du commutateur et le convertit en un entier de code, dans ce cas 13. Le contrôleur du clavier encode ensuite le code pour le transmettre à l'ordinateur. Cela se fait désormais presque universellement via une connexion Universal Serial Bus (USB) ou Bluetooth, mais historiquement, cela se faisait via des connexions PS/2 ou ADB.

*Dans le cas du clavier USB :*

- La partie USB du clavier est alimentée par l'alimentation 5V fournie sur la broche 1 par le contrôleur USB de l'ordinateur.

- Le code généré est stocké dans la mémoire du circuit interne du clavier dans un registre appelé "endpoint".

- Le contrôleur USB de l'ordinateur interroge cet "endpoint" toutes les ~10 ms (valeur minimale déclarée par le clavier), de sorte qu'il obtient la valeur du code stockée.

- Cette valeur passe au SIE USB (Serial Interface Engine) pour être convertie en un ou plusieurs paquets USB suivant le protocole USB de bas niveau.

- Ces paquets sont envoyés par un signal électrique différentiel sur les broches D+ et D- (les deux du milieu) à une vitesse maximale de 1,5 Mb/s, car un périphérique HID (Human Interface Device) est toujours déclaré comme un "périphérique à faible vitesse" (conforme à l'USB 2.0).

- Ce signal série est ensuite décodé par le contrôleur USB hôte de l'ordinateur et interprété par le pilote du clavier universel Human Interface Device (HID) de l'ordinateur. La valeur de la touche est ensuite transmise à la couche d'abstraction matérielle du système d'exploitation.

*Dans le cas du clavier virtuel (comme sur les appareils à écran tactile) :*

- Lorsque l'utilisateur pose son doigt sur un écran tactile capacitif moderne, une petite quantité de courant est transférée au doigt. Cela complète le circuit à travers le champ électrostatique de la couche conductrice et crée une chute de tension à cet endroit de l'écran. Le "contrôleur d'écran" déclenche alors une interruption signalant les coordonnées de l'appui sur la touche.

- Ensuite, le système d'exploitation mobile notifie l'application actuellement focalisée d'un événement de pression dans l'un de ses éléments graphiques (qui est maintenant l'application du clavier virtuel des boutons).

- Le clavier virtuel peut maintenant déclencher une interruption logicielle pour envoyer un message de "touche pressée" à l'OS.

- Cette interruption notifie l'application actuellement focalisée d'un événement "touche pressée".

Interruption déclenchée [NON pour les claviers USB]
----------------------------------------------

Le clavier envoie des signaux sur sa ligne de demande d'interruption (IRQ), qui est cartographiée vers un ``vecteur d'interruption`` (entier) par le contrôleur d'interruption. Le CPU utilise la ``Table des Descripteurs d'Interruption`` (IDT) pour mapper les vecteurs d'interruption vers des fonctions (``gestionnaires d'interruption``) fournies par le noyau. Lorsqu'une interruption arrive, le CPU indexe l'IDT avec le vecteur d'interruption et exécute le gestionnaire approprié. Ainsi, le noyau est appelé.

(Sous Windows) Un message ``WM_KEYDOWN`` est envoyé à l'application
--------------------------------------------------------

Le transport HID passe l'événement de touche enfoncée au pilote ``KBDHID.sys``, qui convertit l'utilisation HID en un code de balayage. Dans ce cas, le code de balayage est ``VK_RETURN`` (``0x0D``). Le pilote ``KBDHID.sys`` interagit avec le pilote de classe de clavier ``KBDCLASS.sys``. Ce pilote est responsable de la gestion de toutes les entrées au clavier et au pavé numérique de manière sécurisée. Il fait ensuite appel à ``Win32K.sys`` (après éventuellement avoir fait passer le message par des filtres de clavier tiers installés). Tout cela se produit en mode noyau.

``Win32K.sys`` détermine quelle fenêtre est la fenêtre active grâce à l'API ``GetForegroundWindow()``. Cette API fournit la poignée de fenêtre de la barre d'adresse du navigateur. Ensuite, la principale "pompe à messages" de Windows appelle ``SendMessage(hWnd, WM_KEYDOWN, VK_RETURN, lParam)``. ``lParam`` est un masque de bits qui indique des informations supplémentaires sur la frappe de touche : nombre de répétitions (0 dans ce cas), le code de balayage réel (peut dépendre du constructeur OEM, mais en général pas pour ``VK_RETURN``), si des touches étendues (par exemple, alt, shift, ctrl) ont également été pressées (elles ne l'ont pas été), et d'autres informations.

L'API Windows ``SendMessage`` est une fonction simple qui ajoute le message à une file d'attente pour la poignée de fenêtre particulière (``hWnd``). Plus tard, la fonction principale de traitement des messages (appelée ``WindowProc``) assignée à la poignée de fenêtre (``hWnd``) est appelée pour traiter chaque message dans la file d'attente.

La fenêtre (``hWnd``) qui est active est en réalité un contrôle d'édition, et le ``WindowProc`` dans ce cas a un gestionnaire de messages pour les messages ``WM_KEYDOWN``. Ce code examine le troisième paramètre qui a été passé à ``SendMessage`` (``wParam``) et, parce qu'il s'agit de ``VK_RETURN``, sait que l'utilisateur a appuyé sur la touche Entrée.

(Sous OS X) Un événement ``KeyDown`` NSEvent est envoyé à l'application
----------------------------------------------------------

Le signal d'interruption déclenche un événement d'interruption dans le pilote de clavier du noyau I/O Kit. Le pilote traduit le signal en un code de touche, qui est transmis au processus ``WindowServer`` d'OS X. En conséquence, le ``WindowServer`` envoie un événement à toutes les applications appropriées (actives ou en écoute) via leur port Mach, où il est placé dans une file d'attente d'événements. Les événements peuvent ensuite être lus depuis cette file par des threads bénéficiant des autorisations nécessaires en appelant la fonction ``mach_ipc_dispatch``. Cela se produit le plus couramment via, et est géré par, une boucle d'événements principale ``NSApplication``, via un ``NSEvent`` de type ``KeyDown``.

(Sous GNU/Linux), le serveur Xorg écoute les codes de touche
-----------------------------------------------------------

Lorsqu'un serveur graphique ``X`` est utilisé, ``X`` utilisera le pilote d'événement générique ``evdev`` pour acquérir la frappe de touche. Une nouvelle cartographie des codes de touche en codes de balayage est effectuée avec des cartes et des règles spécifiques au serveur ``X``. Lorsque la cartographie de balayage de la touche pressée est terminée, le serveur ``X`` envoie le caractère au ``gestionnaire de fenêtres`` (DWM, metacity, i3, etc.), de sorte que le ``gestionnaire de fenêtres`` envoie à son tour le caractère à la fenêtre active. L'API graphique de la fenêtre qui reçoit le caractère imprime le symbole de police approprié dans le champ de texte actif.

Analyser l'URL
-------------

* Le navigateur dispose désormais des informations suivantes contenues dans l'URL (Uniform Resource Locator) :

    - ``Protocole``  "http"
        Utilise le protocole de transfert hypertexte

    - ``Ressource``  "/"
        Récupérer la page principale (page d'accueil)

S'agit-il d'une URL ou d'un terme de recherche ?
------------------------------------------------

Lorsqu'aucun protocole ou nom de domaine valide n'est donné, le navigateur procède à la recherche en envoyant le texte saisi dans la barre d'adresse au moteur de recherche web par défaut du navigateur. Dans de nombreux cas, l'URL contient un morceau de texte spécial ajouté pour indiquer au moteur de recherche qu'il provient de la barre d'adresse d'un navigateur particulier.

Conversion des caractères Unicode non-ASCII dans le nom d'hôte
-----------------------------------------------------------

* Le navigateur vérifie le nom d'hôte à la recherche de caractères qui ne sont pas dans ``a-z``, ``A-Z``, ``0-9``, ``-`` ou ``.``.
* Puisque le nom d'hôte est "google.com", il n'y en aura pas, mais s'il y en avait, le navigateur appliquerait le codage `Punycode`_ à la partie du nom d'hôte de l'URL.

Vérification de la liste HSTS (HTTP Strict Transport Security)
----------------

* Le navigateur vérifie sa liste "HSTS préchargée (HTTP Strict Transport Security)". Il s'agit d'une liste de sites web qui ont demandé à être contactés uniquement via HTTPS.
* Si le site web est dans la liste, le navigateur envoie sa demande via HTTPS au lieu de HTTP. Sinon, la demande initiale est envoyée via HTTP. (Notez qu'un site web peut toujours utiliser la politique HSTS *sans* figurer dans la liste HSTS. La première demande HTTP au site web par un utilisateur recevra une réponse demandant à l'utilisateur de n'envoyer que des demandes HTTPS. Cependant, cette unique demande HTTP pourrait potentiellement laisser l'utilisateur vulnérable à une `attaque de rétrogradation`_, c'est pourquoi la liste HSTS est incluse dans les navigateurs web modernes.)

Recherche DNS
----------

* Le navigateur vérifie si le domaine se trouve dans sa mémoire cache DNS. (pour voir le Cache DNS dans Chrome, allez à `chrome://net-internals/#dns <chrome://net-internals/#dns>`_).
* Si le domaine n'est pas trouvé, le navigateur appelle la fonction de bibliothèque ``gethostbyname`` (varie en fonction du système d'exploitation) pour effectuer la recherche.
* ``gethostbyname`` vérifie si le nom d'hôte peut être résolu en faisant référence au fichier ``hosts`` local (dont l'emplacement `varie en fonction du système d'exploitation`_) avant de tenter de résoudre le nom d'hôte via DNS.
* Si ``gethostbyname`` ne le trouve pas dans sa mémoire cache ni dans le fichier ``hosts``, il envoie une demande au serveur DNS configuré dans la pile réseau. Il s'agit généralement du routeur local ou du serveur DNS mis en cache par le FAI.
* Si le serveur DNS se trouve sur le même sous-réseau, la bibliothèque réseau suit le ``processus ARP`` ci-dessous pour le serveur DNS.
* Si le serveur DNS se trouve sur un sous-réseau différent, la bibliothèque réseau suit le ``processus ARP`` ci-dessous pour l'adresse IP de la passerelle par défaut.

Processus ARP
-----------

Pour envoyer une diffusion ARP (Address Resolution Protocol), la bibliothèque de la pile réseau a besoin de l'adresse IP cible à rechercher. Elle a également besoin de connaître l'adresse MAC de l'interface qu'elle utilisera pour envoyer la diffusion ARP.

L'ARP cache est d'abord vérifié pour une entrée ARP correspondant à notre adresse IP cible. Si elle se trouve dans le mémoire cache ARP, la fonction de bibliothèque renvoie le résultat : IP cible = MAC.

Si l'entrée n'est pas dans le mémoire cache ARP :

* La table de routage est consultée pour voir si l'adresse IP cible se trouve sur l'un des sous-réseaux de la table de routage locale. Si c'est le cas, la bibliothèque utilise l'interface associée à ce sous-réseau. Si ce n'est pas le cas, la bibliothèque utilise l'interface ayant le sous-réseau de la passerelle par défaut.

* L'adresse MAC de l'interface réseau sélectionnée est recherchée.

* La bibliothèque réseau envoie une demande ARP de la couche 2 (couche de liaison de données du modèle `OSI`_):

``Demande ARP``::

    MAC émetteur : interface:adresse:mac:ici
    IP émetteur : interface.ip.va.ici
    MAC cible : FF:FF:FF:FF:FF:FF (Diffusion)
    IP cible : adresse.ip.cible.ici

Selon le type de matériel entre l'ordinateur et le routeur :

Directement connecté :

* Si l'ordinateur est directement connecté au routeur, le routeur répondra avec une "Réponse ARP" (voir ci-dessous).

Hub :

* Si l'ordinateur est connecté à un concentrateur, le concentrateur diffusera la demande ARP sur tous les autres ports. Si le routeur est connecté sur le même "câble", il répondra avec une "Réponse ARP" (voir ci-dessous).

Commutateur :

* Si l'ordinateur est connecté à un commutateur, le commutateur vérifiera sa table CAM/MAC locale pour voir quel port a l'adresse MAC que nous recherchons. Si le commutateur n'a pas d'entrée pour l'adresse MAC, il réémettra la demande ARP sur tous les autres ports.

* Si le commutateur a une entrée dans la table MAC/CAM, il enverra la demande ARP au port contenant l'adresse MAC que nous recherchons.

* Si le routeur se trouve sur le même "câble", il répondra avec une "Réponse ARP" (voir ci-dessous)

``Réponse ARP``::

    MAC émetteur : adresse:mac:cible:ici
    IP émetteur : adresse.ip.cible.ici
    MAC cible : interface:adresse:mac:ici
    IP cible : interface.ip.va.ici

Maintenant que la bibliothèque réseau dispose de l'adresse IP de notre serveur DNS ou de la passerelle par défaut, elle peut reprendre son processus DNS :

* Le client DNS établit une socket vers le port UDP 53 du serveur DNS, en utilisant un port source supérieur à 1023.
* Si la taille de la réponse est trop grande, le protocole TCP sera utilisé à la place.
* Si le serveur DNS local/FAI ne le possède pas, une recherche récursive est demandée, ce qui remonte la liste des serveurs DNS jusqu'à atteindre l'entité SOA, et si elle est trouvée, une réponse est renvoyée

Ouverture d'un socket
-------------------
Une fois que le navigateur reçoit l'adresse IP du serveur de destination, il prend cette adresse ainsi que le numéro de port fourni dans l'URL (le protocole HTTP utilise par défaut le port 80, et HTTPS le port 443), et effectue un appel à la fonction de bibliothèque système appelée ``socket`` et demande un flux de socket TCP - ``AF_INET/AF_INET6`` et ``SOCK_STREAM``.

* Cette demande est d'abord transmise à la couche de transport, où un segment TCP est créé. Le port de destination est ajouté à l'en-tête, et un port source est choisi dans la plage dynamique du noyau (ip_local_port_range sous Linux).
* Ce segment est envoyé à la couche réseau, qui enveloppe un en-tête IP supplémentaire. L'adresse IP du serveur de destination ainsi que celle de la machine actuelle sont insérées pour former un paquet.
* Le paquet arrive ensuite à la couche de liaison. Un en-tête de trame est ajouté, qui inclut l'adresse MAC de la carte réseau de la machine ainsi que l'adresse MAC de la passerelle (routeur local). Comme précédemment, si le noyau ne connaît pas l'adresse MAC de la passerelle, il doit diffuser une requête ARP pour la trouver.

À ce stade, le paquet est prêt à être transmis par l'un des moyens suivants :

* `Ethernet`_
* `WiFi`_
* `Réseau de données cellulaire`_

Pour la plupart des connexions Internet domestiques ou de petites entreprises, le paquet passera de votre ordinateur, éventuellement à travers un réseau local, puis à travers un modem (MOdulateur/DEModulateur) qui convertit les 1 et les 0 numériques en un signal analogique adapté à la transmission sur des connexions téléphoniques, câble ou sans fil. À l'autre extrémité de la connexion se trouve un autre modem qui convertit le signal analogique en données numériques à traiter par le nœud réseau suivant où les adresses source et destination seront analysées plus en détail.

La plupart des grandes entreprises et certaines nouvelles connexions résidentielles disposent de connexions en fibre ou en Ethernet direct, auquel cas les données restent numériques et sont transmises directement au nœud réseau suivant pour le traitement.

Finalement, le paquet atteindra le routeur gérant le sous-réseau local. De là, il continuera à voyager vers les routeurs frontaliers du système autonome (AS), d'autres AS, et enfin vers le serveur de destination. Chaque routeur en cours de route extrait l'adresse de destination de l'en-tête IP et la route vers la prochaine étape appropriée. Le champ de temps de vie (TTL) de l'en-tête IP est décrémenté de un pour chaque routeur qui passe. Le paquet sera abandonné si le champ TTL atteint zéro ou si le routeur actuel n'a pas d'espace dans sa file d'attente (peut-être en raison de la congestion du réseau).

Cet envoi et cette réception se produisent plusieurs fois en suivant le flux de la connexion TCP :

* Le client choisit un numéro de séquence initial (ISN) et envoie le paquet au serveur avec le bit SYN réglé pour indiquer qu'il définit l'ISN.
* Le serveur reçoit le SYN et s'il est d'accord :
   * Le serveur choisit son propre numéro de séquence initial.
   * Le serveur définit SYN pour indiquer qu'il choisit son ISN.
   * Le serveur copie (ISN client +1) dans son champ ACK et ajoute le drapeau ACK pour indiquer qu'il accuse réception du premier paquet.
* Le client confirme la connexion en envoyant un paquet :
   * Augmente son propre numéro de séquence
   * Augmente le numéro d'accusé de réception du récepteur
   * Définit le champ ACK
* Les données sont transférées de la manière suivante :
   * Lorsqu'un côté envoie N octets de données, il augmente son SEQ de ce nombre
   * Lorsque l'autre côté accuse réception de ce paquet (ou d'une série de paquets), il envoie un

Si un paquet est perdu
----------------------

Parfois, en raison de la congestion du réseau ou de connexions matérielles instables, les paquets TLS seront perdus avant d'atteindre leur destination finale. L'expéditeur doit alors décider comment réagir. L'algorithme pour cela est appelé le `contrôle de congestion TCP`_. Cela varie en fonction de l'expéditeur ; les algorithmes les plus courants sont `cubic`_ sur les systèmes d'exploitation récents et `New Reno`_ sur presque tous les autres.

* Le client choisit une `fenêtre de congestion`_ en fonction de la `taille maximale de segment`_
  (MSS) de la connexion.
* Pour chaque paquet accusé de réception, la fenêtre double de taille jusqu'à atteindre le
  'seuil de démarrage lent'. Dans certaines implémentations, ce seuil est adaptatif.
* Après avoir atteint le seuil de démarrage lent, la fenêtre augmente de manière additive pour
  chaque paquet accusé de réception. Si un paquet est perdu, la fenêtre diminue de manière
  exponentielle jusqu'à ce qu'un autre paquet soit accusé de réception.

Protocole HTTP
-------------

Si le navigateur Web utilisé a été écrit par Google, au lieu d'envoyer une requête HTTP pour récupérer la page, il enverra une requête pour essayer de négocier avec le serveur une "mise à niveau" du protocole HTTP vers le protocole SPDY.

Si le client utilise le protocole HTTP et ne prend pas en charge SPDY, il envoie une requête au serveur sous la forme suivante::

    GET / HTTP/1.1
    Host: google.com
    Connection: close
    [autres en-têtes]

où ``[autres en-têtes]`` fait référence à une série de paires clé-valeur séparées par des deux-points formatées selon la spécification HTTP et séparées par une nouvelle ligne unique.
(Cela suppose que le navigateur Web utilisé n'a pas de bogues violant la spécification HTTP. Cela suppose également que le navigateur Web utilise ``HTTP/1.1``, sinon il peut ne pas inclure l'en-tête ``Host`` dans la requête et la version spécifiée dans la demande ``GET`` sera soit ``HTTP/1.0`` soit ``HTTP/0.9``.)

HTTP/1.1 définit l'option de connexion "close" pour que l'expéditeur signale que la connexion sera fermée après l'achèvement de la réponse. Par exemple,

    Connection: close

Les applications HTTP/1.1 qui ne prennent pas en charge les connexions persistantes DOIVENT inclure l'option de connexion "close" dans chaque message.

Après avoir envoyé la demande et les en-têtes, le navigateur Web envoie une seule ligne vide au serveur indiquant que le contenu de la demande est terminé.

Le serveur répond avec un code de réponse indiquant l'état de la demande et répond avec une réponse sous la forme suivante::

    200 OK
    [en-têtes de réponse]

Suivi d'une seule ligne vide, puis envoie une charge utile du contenu HTML de
``www.google.com``. Le serveur peut alors soit fermer la connexion, soit, si les en-têtes envoyés par le client le demandent, maintenir la connexion ouverte pour être réutilisée pour d'autres demandes.

Si les en-têtes HTTP envoyés par le navigateur Web contenaient suffisamment d'informations pour que le serveur Web puisse déterminer si la version du fichier mise en cache par le navigateur Web n'a pas été modifiée depuis la dernière récupération (c'est-à-dire si le navigateur Web incluait un en-tête ``ETag``), il peut répondre plutôt avec une demande de la forme::

    304 Not Modified
    [en-têtes de réponse]

et aucune charge utile, et le navigateur Web récupère le HTML depuis son cache.

Après avoir analysé le HTML, le navigateur Web (et le serveur) répète ce processus pour chaque ressource (image, CSS, favicon.ico, etc.) référencée par la page HTML, sauf que la demande sera ``GET /$(URL relative à www.google.com) HTTP/1.1`` au lieu de ``GET / HTTP/1.1``.

Si le HTML faisait référence à une ressource sur un domaine différent de ``www.google.com``, le navigateur Web revient aux étapes de résolution de l'autre domaine, et suit toutes les étapes jusqu'à ce point pour ce domaine. L'en-tête ``Host`` de la demande sera réglé sur le nom du serveur approprié au lieu de ``google.com``.

Gestion des requêtes du serveur HTTP
-------------------------------------
Le serveur HTTPD (HTTP Daemon) est celui qui gère les requêtes/réponses côté serveur. Les serveurs HTTPD les plus courants sont Apache ou nginx pour Linux et IIS pour Windows.

* Le serveur HTTPD (HTTP Daemon) reçoit la requête.
* Le serveur décompose la requête en les paramètres suivants :
   * Méthode de requête HTTP (soit ``GET``, ``HEAD``, ``POST``, ``PUT``,
     ``PATCH``, ``DELETE``, ``CONNECT``, ``OPTIONS``, ou ``TRACE``). Dans le cas d'une URL saisie directement dans la barre d'adresse, il s'agit de ``GET``.
   * Domaine, dans ce cas - google.com.
   * Chemin/page demandée, dans ce cas - / (car aucun chemin/page spécifique n'a été demandé, / est le chemin/page par défaut).
* Le serveur vérifie s'il existe un hôte virtuel configuré sur le serveur qui correspond à google.com.
* Le serveur vérifie si google.com peut accepter les requêtes GET.
* Le serveur vérifie si le client est autorisé à utiliser cette méthode
  (par IP, authentification, etc.).
* Si le serveur a un module de réécriture installé (comme mod_rewrite pour Apache ou
  URL Rewrite pour IIS), il essaie de faire correspondre la requête à l'une des
  règles configurées. Si une règle correspondante est trouvée, le serveur utilise cette règle pour
  réécrire la requête.
* Le serveur va chercher le contenu qui correspond à la requête,
  dans notre cas il s'agira du fichier index, car "/" est le fichier principal
  (certains cas peuvent remplacer cela, mais c'est la méthode la plus courante).
* Le serveur analyse le fichier en fonction du gestionnaire. Si Google
  fonctionne avec PHP, le serveur utilise PHP pour interpréter le fichier index, et
  envoie la sortie au client.

En coulisses du navigateur
--------------------------
Une fois que le serveur fournit les ressources (HTML, CSS, JS, images, etc.)
au navigateur, il passe par le processus suivant :

* Analyse - HTML, CSS, JS
* Rendu - Construction de l'arbre DOM → Arbre de rendu → Mise en page de l'arbre de rendu →
  Rendu de l'arbre de rendu

Navigateur
----------
La fonction du navigateur est de présenter la ressource Web que vous choisissez, en
la demandant au serveur et en l'affichant dans la fenêtre du navigateur.
La ressource est généralement un document HTML, mais peut également être un PDF,
une image ou un autre type de contenu. L'emplacement de la ressource est
spécifié par l'utilisateur à l'aide d'un URI (Uniform Resource Identifier).

La manière dont le navigateur interprète et affiche les fichiers HTML est spécifiée
dans les spécifications HTML et CSS. Ces spécifications sont maintenues
par l'organisation W3C (World Wide Web Consortium), qui est l'organisation de
normalisation pour le web.

Les interfaces utilisateur des navigateurs ont de nombreux éléments communs. Parmi les
éléments d'interface utilisateur courants, on trouve :

* Une barre d'adresse pour insérer un URI
* Des boutons de retour et d'avance
* Options de signet
* Des boutons de rafraîchissement et d'arrêt pour rafraîchir ou arrêter le chargement de
  documents en cours
* Un bouton d'accueil qui vous ramène à votre page d'accueil

**Structure de haut niveau du navigateur**

Les composants des navigateurs sont les suivants :

* **Interface utilisateur :** L'interface utilisateur comprend la barre d'adresse,
  les boutons de retour/avance, le menu de signets, etc. Chaque partie de l'affichage du navigateur, à l'exception de la fenêtre où vous voyez la page demandée.
* **Moteur de navigateur :** Le moteur de navigateur orchestre les actions entre l'interface utilisateur et le moteur de rendu.
* **Moteur de rendu :** Le moteur de rendu est responsable de l'affichage du contenu demandé. Par exemple, si le contenu demandé est en HTML, le moteur de rendu analyse le HTML et le CSS, puis affiche le contenu analysé à l'écran.
* **Réseau :** Le réseau gère les appels réseau tels que les requêtes HTTP, en utilisant différentes implémentations pour différentes plates-formes derrière une interface indépendante de la plate-forme.
* **Backend de l'interface utilisateur :** Le backend de l'interface utilisateur est utilisé pour dessiner des widgets de base tels que les zones de liste déroulante et les fenêtres. Ce backend expose une interface générique qui n'est pas spécifique à une plate-forme. En dessous, il utilise des méthodes d'interface utilisateur spécifiques au système d'exploitation.
* **Moteur JavaScript :** Le moteur JavaScript est utilisé pour analyser et
  exécuter du code JavaScript.
* **Stockage des données :** Le stockage des données est une couche de persistance. Le navigateur peut
  avoir besoin de sauvegarder toutes sortes de données localement, comme les cookies. Les navigateurs prennent également en charge des mécanismes de stockage tels que localStorage, IndexedDB, WebSQL et FileSystem.

Analyse HTML
------------

Le moteur de rendu commence à récupérer le contenu du document demandé depuis la couche réseau. Cela se fait généralement par tranches de 8 ko.

Le rôle principal du parseur HTML est d'analyser les balises HTML pour construire un arbre d'analyse.

L'arbre de sortie (l'« arbre d'analyse ») est un arbre composé de nœuds d'éléments et d'attributs du DOM. DOM est l'acronyme de Document Object Model. C'est la représentation objet du document HTML et l'interface des éléments HTML vers le monde extérieur, comme JavaScript. La racine de l'arbre est l'objet « Document ». Avant toute manipulation via un script, le DOM a une relation presque un à un avec la balise.

**L'algorithme d'analyse**

L'HTML ne peut pas être analysé à l'aide des parseurs classiques descendant ou ascendant.

Les raisons en sont les suivantes :

* La nature indulgente du langage.
* Le fait que les navigateurs aient une tolérance aux erreurs traditionnelle pour prendre en charge les cas bien connus d'HTML non valide.
* Le processus d'analyse est réentrant. Pour d'autres langages, la source ne change pas pendant l'analyse, mais en HTML, le code dynamique (tel que les éléments de script contenant des appels `document.write()`) peut ajouter des jetons supplémentaires, de sorte que le processus d'analyse modifie réellement l'entrée.

Incapables d'utiliser les techniques d'analyse régulières, les navigateurs utilisent un parseur personnalisé pour analyser l'HTML. L'algorithme d'analyse est décrit en détail dans la spécification HTML5.

L'algorithme se compose de deux étapes : la tokenisation et la construction de l'arbre.

**Actions lorsque l'analyse est terminée**

Le navigateur commence à récupérer les ressources externes liées à la page (CSS, images, fichiers JavaScript, etc.).

À ce stade, le navigateur marque le document comme interactif et commence à analyser les scripts en mode « différé » : ceux qui doivent être exécutés après l'analyse du document. L'état du document est défini comme « complet » et un événement « load » est déclenché.

Il est important de noter qu'il n'y a jamais d'erreur de « Syntaxe incorrecte » sur une page HTML. Les navigateurs corrigent tout contenu non valide et continuent.

Interprétation CSS
------------------

* Analyser les fichiers CSS, le contenu des balises ``<style>``, et les valeurs des attributs ``style`` en utilisant la « grammaire lexicale et syntaxique CSS ».
* Chaque fichier CSS est analysé en un « objet Feuille de style » où chaque objet
  contient des règles CSS avec des sélecteurs et des objets correspondant à la grammaire CSS.
* Un analyseur CSS peut être descendant ou ascendant lorsqu'un générateur de parseur spécifique est utilisé.

Rendu de la page
--------------

* Créer un « arbre de trame » ou un « arbre de rendu » en parcourant les nœuds DOM et
  en calculant les valeurs de style CSS pour chaque nœud.
* Calculer la largeur préférée de chaque nœud dans l'« arbre de trame » de bas en haut
  en additionnant la largeur préférée des nœuds enfants et les marges horizontales, les bordures et les espacements des nœuds.
* Calculer la largeur réelle de chaque nœud de haut en bas en attribuant la largeur disponible de chaque nœud à ses enfants.
* Calculer la hauteur de chaque nœud de bas en haut en appliquant la césure du texte et
  en additionnant les hauteurs des nœuds enfants et les marges, bordures et espacements du nœud.
* Calculer les coordonnées de chaque nœud à l'aide des informations calculées
  ci-dessus.
* Des étapes plus complexes sont nécessaires lorsque les éléments sont « flottants »,
  positionnés « absolument » ou « relativement », ou d'autres fonctionnalités complexes
  sont utilisées. Voir
  http://dev.w3.org/csswg/css2/ et http://www.w3.org/Style/CSS/current-work
  pour plus de détails.
* Créer des couches pour décrire quelles parties de la page peuvent être animées en groupe
  sans être ré-rasterisées. Chaque objet de trame/de rendu est attribué à une couche.
* Des textures sont allouées pour chaque couche de la page.
* Les objets de trame/de rendu de chaque couche sont parcourus et les commandes de dessin
  sont exécutées pour leur couche respective. Cela peut être rasterisé par le CPU ou dessiné sur le GPU directement en utilisant D2D/SkiaGL.
* Toutes les étapes ci-dessus peuvent réutiliser les valeurs calculées la dernière fois que
  la page web a été rendue, de sorte que les modifications progressives nécessitent moins de travail.
* Les couches de la page sont envoyées au processus de composition où elles sont combinées
  avec les couches pour d'autres contenus visibles tels que le navigateur chrome, les iframes
  et les panneaux d'extension.
* Les positions finales des couches sont calculées et les commandes de composition sont émises
  via Direct3D/OpenGL. Le tampon de commandes GPU est vidé vers le GPU pour un rendu asynchrone et l'image est envoyée au serveur de fenêtrage.

Rendu GPU
---------

* Pendant le processus de rendu, les couches informatiques graphiques peuvent utiliser le CPU général
  ou le processeur graphique (GPU) également.

* Lors de l'utilisation du GPU pour les calculs de rendu graphique, les couches logicielles graphiques
  divisent la tâche en plusieurs morceaux, de manière à tirer parti du parallélisme massif du GPU pour les calculs en virgule flottante requis pour le processus de rendu.


Serveur de fenêtrage
-------------------

Post-rendu et exécution induite par l'utilisateur
----------------------------------------

Une fois le rendu terminé, le navigateur exécute le code JavaScript en réponse à un mécanisme de temporisation (tel qu'une animation Google Doodle) ou à une interaction de l'utilisateur (saisir une requête dans la zone de recherche et recevoir des suggestions).
Des plugins tels que Flash ou Java peuvent également être exécutés, bien que ce ne soit pas le cas actuellement sur la page d'accueil de Google. Les scripts peuvent entraîner l'exécution de requêtes réseau supplémentaires, ainsi que la modification de la page ou de sa mise en page, provoquant une autre étape de rendu et de peinture de la page.

