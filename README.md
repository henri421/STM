# Bielle-Tirant

Application web autonome (PWA) qui calcule les efforts normaux dans un treillis plan, pour construire et exploiter un modele bielle-tirant sans recourir a un logiciel de structure ni resoudre la trigonometrie a la main. Tout tourne dans le navigateur, sans serveur ni envoi reseau, ce qui la rend installable et utilisable hors ligne, et compatible avec un traitement local de donnees sensibles.

Le calcul repose sur la methode des rigidites (treillis plan articule, effort axial seul). Convention retenue : effort positif en traction (tirant), negatif en compression (bielle). Reference du modele : EN 1992-1-1:2004 section 6.5, reecrite en section 8.5 dans la deuxieme generation EN 1992-1-1:2023.

## Contenu

Le depot est un site statique. `index.html` est autonome : il embarque le noyau de calcul, l'interface et le style. `manifest.webmanifest`, `service-worker.js` et le dossier `icons/` assurent l'installation et le fonctionnement hors ligne.

## Deploiement sur GitHub Pages

1. Creer un depot GitHub et y deposer le contenu de ce dossier a la racine.
2. Dans Settings puis Pages, choisir la source Deploy from a branch, la branche `main` et le dossier `/root`.
3. Ouvrir l'adresse fournie, du type `https://utilisateur.github.io/nom-du-depot/`. Les chemins sont relatifs, l'application fonctionne donc sous ce sous-repertoire.
4. Sur mobile ou sur ordinateur, le navigateur proposera d'installer l'application ; un bouton Installer apparait aussi dans l'en-tete lorsque le navigateur le permet.

Pour un essai local avant mise en ligne, servir le dossier par un serveur statique, par exemple `python3 -m http.server`, puis ouvrir `http://localhost:8000`. Ouvrir `index.html` par un double-clic suffit pour le calcul, mais le service worker ne s'enregistre que via `http` ou `https`.

## Utilisation

Un menu Modele charge des configurations de depart : poutre-cloison a charge centree, console courte dissymetrique, semelle sur deux pieux, ou gabarit vide. Chaque donnee est editable dans le panneau de droite et le schema se recalcule immediatement.

Les coordonnees des noeuds et les charges acceptent des expressions parametriques faisant intervenir les variables declarees, par exemple `a/2`, `-P`, ou `h*tan(pi/6)`. Les fonctions `sin cos tan asin acos atan sqrt abs exp ln` et la constante `pi` sont disponibles ; une variable de meme nom qu'une constante prime sur elle, ce qui permet d'utiliser `e` comme excentricite.

Les appuis se definissent degre de liberte par degre de liberte, en bloquant independamment la translation horizontale et verticale, ce qui couvre l'appui fixe, l'appui glissant et tout blocage partiel. Le tableau de resultats donne l'effort de chaque barre avec sa nature, les reactions d'appui, le degre d'hyperstaticite et le residu d'equilibre.

Lorsqu'un modele est instable, l'outil ne se contente pas de le signaler : il calcule les modes de mecanisme comme noyau de la matrice de rigidite et en deduit un diagnostic detaille. Chaque mode est decrit en clair, en nommant les noeuds concernes et la direction du deplacement libre, en distinguant un mecanisme local, dû a un noeud insuffisamment tenu, d'un mode d'ensemble dû a des appuis insuffisants, et en suggerant la correction (ajouter une barre non alignee ou un appui, retablir un blocage). Un bilan compare le nombre de barres et de reactions au nombre de degres de liberte, et les axes libres sont figures en orange sur le schema, a chaque noeud pouvant se deplacer. Toute correction peut etre annulee.

La carte Generateur de topologie propose un modele complet a partir du seul contour de beton, en reutilisant les appuis et charges du modele courant. On fixe un domaine rectangulaire, une densite de grille et le rapport des contraintes admissibles traction sur compression, puis l'outil seme des noeuds candidats, relie toutes les barres non redondantes et resout un programme lineaire de volume minimal sous la seule condition d'equilibre, ce qui produit un treillis proche de l'optimum de Michell. Les points d'appui et de charge sont injectes exactement dans la structure de sol, de sorte qu'un appui peut se trouver n'importe ou, hors de la trame. On peut aussi declarer un appui lineaire le long d'une arete (bas, haut, gauche ou droite), en blocage vertical seul ou vertical et horizontal ; les appuis inutilises sont ensuite elagues et un blocage horizontal est ajoute au besoin pour la stabilite. Les tirants sont restreints aux directions horizontale, verticale et 45 degres afin de rester ferraillables, tandis que les bielles peuvent prendre toute inclinaison. Les chaines colineaires sont recollees pour eviter les noeuds intermediaires instables. Le resultat n'est jamais une verite mais une proposition : il est ecrit dans les tableaux et se corrige comme n'importe quel modele, ce qui est la raison d'etre de l'editeur. Le rapport de contraintes gouverne la topologie ; une valeur de 1 correspond au treillis de Michell symetrique, une valeur superieure favorise les tirants.

Le modele de depart Voile, charge excentree illustre le cas d'un mur charge hors de son axe : la charge descendant vers deux appuis decales engendre un tirant, comme dans les modeles a chemin de charge de Schlaich. Un appui lineaire a la base ou des appuis places librement permettent d'explorer les variantes.

Les unites sont libres mais coherentes ; l'affichage suppose des forces en kN et des longueurs en m.

Toute action est reversible : les boutons Annuler et Retablir de l'en-tete parcourent l'historique, y compris apres une generation ou une optimisation, de sorte qu'un modele n'est jamais perdu. La carte Sauvegarde et export permet d'enregistrer le modele courant sous un nom dans le navigateur puis de le recharger, d'exporter et de reimporter le modele complet au format .json pour l'archiver ou le transferer, d'exporter les efforts et reactions en .csv pour un tableur, et d'exporter le schema en .svg vectoriel ou en .png. Les sauvegardes nommees reposent sur le stockage local du navigateur ; si celui-ci est indisponible, l'export .json prend le relais.

Le domaine du generateur peut etre ajuste au modele courant par un bouton dedie, et il est de toute facon elargi automatiquement au moment de la generation pour englober tous les appuis et charges, ce qui evite un modele sans rapport lorsque la geometrie a ete editee.

La carte Optimisation de geometrie recherche, a topologie fixee, les coordonnees qui minimisent un objectif. On declare une ou plusieurs variables libres avec leurs bornes, on choisit l'objectif (effort du tirant critique, volume d'acier au sens de Schlaich, ou volume total, tous fondes sur l'equilibre donc exacts en isostatique) et on fixe la bande d'angle admissible entre bielles et tirants. Pour une seule variable libre, l'outil balaie le domaine et trace la courbe de l'objectif, en grisant la zone inadmissible et en reperant l'optimum ; au-dela, une recherche de type Nelder-Mead localise l'optimum et une coupe est tracee. L'optimum est reporte dans la geometrie, qui se redessine, et la contrainte active est nommee. La bande d'angle est une valeur de projet a fixer selon le cas et l'annexe nationale ; pour une console, l'annexe J informative de l'EN 1992-1-1:2004 suggere couramment une inclinaison de bielle telle que la tangente reste entre 1,0 et 2,5, soit environ 45 a 68 degres.

## Rapport de verification

Le noyau a ete confronte a des calculs manuels independants avant livraison. Les efforts d'un treillis isostatique etant independants des rigidites, ces valeurs sont exactes.

Poutre-cloison symetrique, charge de 100 kN, portee 4 m, hauteur 2 m : tirant inferieur +50,00 kN, bielles inclinees -70,71 kN, reactions verticales 50 kN, reaction horizontale nulle. Residu d'equilibre de l'ordre de 1e-14.

Console courte dissymetrique, charge de 300 kN, porte-a-faux 0,30 m, bras de levier 0,45 m : tirant superieur +200,00 kN, bielle inclinee -360,56 kN, reaction d'ancrage du tirant -200 kN, reaction au pied de bielle 200 kN horizontale et 300 kN verticale.

Un cas non tenu confirme la detection de mecanisme, et un controle dedie verifie qu'une variable nommee `e` prime sur la constante d'Euler. L'application assemblee a ensuite ete testee de bout en bout : lecture des efforts affiches, changement de modele, recalcul parametrique (diviser par deux la hauteur de semelle double l'effort du tirant) et signalement d'instabilite depuis l'interface. Onze controles, tous positifs.

L'optimiseur a ete verifie de la meme maniere. Sur la console, bras de levier libre et angle non contraint, il retrouve l'optimum analytique a la borne superieure et la courbe de l'objectif coincide avec la loi T = F.a/z. Avec une bande d'angle limitee a 60 degres, il s'arrete exactement sur cette contrainte, a z = 0,516 m, tirant ramene de 200 a 174,4 kN, valeur conforme au calcul direct. Le solveur Nelder-Mead a ete valide sur une fonction a minimum connu, et sept controles de bout en bout confirment le report de l'optimum dans la geometrie et la signalisation de la contrainte active.

Le generateur repose sur un simplexe deux phases, verifie sur des programmes lineaires a solution connue et sur la detection d'un systeme incompatible. Il restitue les deux modeles canoniques : sur la console, un tirant a +200 kN et une bielle a -360,55 kN ; sur la poutre-cloison, deux bielles a 45 degres a -70,71 kN et un tirant inferieur unique a +50 kN apres recollement des chaines. Chaque modele genere, re-resolu par la methode des rigidites, est stable et redonne les memes efforts, et dix controles de bout en bout confirment la generation depuis l'interface, y compris le refus d'une grille trop dense.

## Feuille de route

Cette version couvre les trois briques prevues : le solveur parametrique avec son editeur, l'optimiseur de geometrie a topologie fixee, et le generateur de topologie par structure de sol. Elle fournit les efforts, la geometrie optimisee et une proposition de modele corrigeable. Restent hors de son perimetre, et volontairement du ressort de l'ingenieur, la verification des contraintes de bielles et de noeuds et le dimensionnement des armatures selon l'EN 1992-1-1 (2004 section 6.5 ; 2023 section 8.5) et ses annexes nationales belge et luxembourgeoise.

## Reserves

Cet outil est une aide au calcul. Il fournit l'equilibre du treillis saisi, non une justification reglementaire. Le choix du modele, son admissibilite, la verification des bielles, tirants et noeuds selon l'Eurocode et ses annexes nationales, ainsi que la responsabilite de conception, restent celles de l'ingenieur du projet.
