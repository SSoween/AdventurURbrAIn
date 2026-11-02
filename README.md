# Résumé de discussion — Jeu d’aventure textuel type Zork en PWA

## 1. Durée possible d’une aventure textuelle

Une aventure textuelle n’est pas limitée à 1–2 h. La durée dépend surtout de l’architecture et de la quantité de contenu.

| Format | Durée typique | Nombre d’échanges | Prérequis |
|---|---:|---:|---|
| Aventure courte / jam | 30 min–2 h | 30–100 | Petit scénario, quelques lieux, 2–3 puzzles |
| Aventure classique | 4–12 h | 150–500 | Chapitres, inventaire, flags, quêtes, embranchements |
| Campagne scénarisée | 20–50 h | 1 000+ | Beaucoup d’écriture ou génération contrôlée, état robuste |
| Monde persistant / épisodique | 50–500 h+ | quasi illimité | Mémoire longue, quêtes génératives, PNJ persistants, économie, événements |
| Théorique | pas de maximum | infini | Génération + mémoire + validation + cohérence |

Zork lui-même se joue plutôt en une dizaine d’heures pour un joueur normal, et la trilogie représente des dizaines d’heures. Le texte pur n’impose donc aucune limite de durée.

### La vraie limite : la cohérence

Sans mémoire structurée, un simple chat dérive souvent après 1–2 h : oublis, contradictions, PNJ incohérents, quêtes abandonnées, répétitions.

Pour aller loin, il faut :
- un état structuré : lieux, inventaire, flags, quêtes, relations, temps ;
- des résumés hiérarchiques de l’histoire ;
- une mémoire des PNJ et du monde ;
- des scripts/validateurs pour empêcher les incohérences ;
- éventuellement des chapitres/épisodes avec fins et relances.

### Réponse courte
- Sans ingénierie particulière : 1–2 h, voire quelques heures.
- Avec un jeu bien conçu : 10–30 h très faisable.
- En mode monde persistant/épisodique : dizaines ou centaines d’heures, voire un jeu “sans fin” pratique.
- Maximum théorique : illimité, mais la qualité et la cohérence sont le vrai plafond.

---

## 2. Peut-on faire un jeu persistant/illimité en PWA sur mobile ?

Oui, c’est possible et adapté. Une PWA peut être installée sur l’écran d’accueil, fonctionner hors-ligne pour l’interface et stocker localement l’état du jeu.

### Atouts de la PWA
- Installation sans passer par les stores.
- Fonctionnement hors-ligne grâce au Service Worker.
- Stockage local généreux via IndexedDB.
- Sur Chrome/Android, le quota peut être important (jusqu’à une fraction élevée de l’espace disque).

### Point critique : la persistance des données sur iOS
- Safari applique l’ITP : une PWA non installée peut voir ses données locales supprimées après environ 7 jours sans ouverture.
- Solution : inciter fortement l’utilisateur à installer la PWA sur l’écran d’accueil.
- Utiliser `navigator.storage.persist()` pour demander une persistance de stockage.
- Sur Android/Chrome, le stockage est plus prévisible et généreux.
- Pour une tranquillité totale : sauvegarde optionnelle chiffrée sur le cloud.

### Architecture recommandée pour un jeu “sans fin”
1. État du monde persistant dans IndexedDB (via Dexie.js par exemple).
2. Mémoire hiérarchique : résumés réguliers plutôt que tout l’historique.
3. Séparation entre moteur de jeu et narration.
4. Sauvegarde cloud optionnelle pour restauration/synchronisation.

---

## 3. Poids d’une telle PWA

### Poids de l’application
- Généralement 1–5 Mo, souvent 1–3 Mo.
- Code/interface : souvent quelques centaines de Ko compressés.
- Polices et icônes : souvent le poste le plus lourd (100–500 Ko ou plus).
- Service Worker/manifeste : négligeable.

### Poids des sauvegardes
- Pour un jeu purement textuel : quelques Mo après une très longue partie.
- IndexedDB gère sans problème environ 20 Mo de sauvegardes.
- Les quotas modernes sont bien supérieurs.
- Les illustrations, musiques ou polices lourdes sont les principaux facteurs de gonflement.

### Résumé
- Application : ~1–5 Mo.
- Sauvegardes : quelques Mo.
- Format idéal pour un jeu persistant : téléchargement quasi instantané et stockage négligeable.

---

## 4. Comment fonctionne ce genre d’app ?

Trois grands modèles existent.

### Modèle 1 : Tout est préétabli à l’avance
- Le monde, les énigmes et les réponses sont écrits par un auteur.
- Le moteur lit des fichiers de données et interprète les commandes.
- Avantages : cohérence parfaite, maîtrise totale, débogage facile.
- Limites : coût d’écriture élevé, contenu épuisable, embranchements exponentiels.

### Modèle 2 : Tout est créé à la volée par un moteur génératif
- Un LLM invente le monde, les PNJ et les conséquences en temps réel.
- Avantages : liberté et rejouabilité quasi infinies.
- Problème : dérive, oublis, contradictions. Un LLM est sans état et ne se souvient que de ce qu’on lui donne dans le prompt.

### Modèle 3 : Approche hybride
C’est le modèle recommandé pour un jeu persistant.

Principe :
- Un moteur déterministe gère la vérité objective du monde : points de vie, inventaire, lieu, quêtes, flags.
- Le LLM ne fait que narrer les conséquences des actions.
- Le LLM reçoit l’état à chaque tour et renvoie une narration + éventuellement une ligne d’état machine-lisible.
- Le moteur valide et applique les changements.
- Le LLM est un “directeur narratif”, pas le moteur du jeu.

Pourquoi c’est la clé :
- Le monde ne dérive pas.
- La narration reste variée.
- L’histoire peut durer des centaines d’heures car l’état est stocké de manière fiable.

---

## 5. LLM et poids de l’app : clarification

Quand on parle de “quelques Mo”, il s’agit de la PWA, pas du LLM.

| Architecture | Poids de l’app | LLM où ? | Internet requis ? | Coût |
|---|---:|---|---|---|
| LLM côté serveur / API | 1–5 Mo | Serveur distant | Oui pour la narration | Payant à l’usage |
| Petit modèle local dans l’app | 100 Mo–2 Go+ | Sur le téléphone | Non | Gratuit mais lourd/lent |
| Tout statique + moteur local | < 1–5 Mo | Pas de LLM | Non | Gratuit |

### Fonctionnement réaliste pour une PWA légère
1. La PWA contient l’interface, le moteur de règles, l’état du monde, la sauvegarde IndexedDB, les prompts et la logique de mémoire.
2. La PWA envoie au serveur l’état courant, le résumé de l’histoire et l’action du joueur.
3. Le serveur appelle le LLM, filtre/valide la réponse et renvoie la narration + les changements d’état.
4. La PWA applique les changements via son moteur déterministe.

### Pourquoi ne pas embarquer le LLM dans l’app ?
- Un LLM de qualité pour le jeu textuel pèse souvent des centaines de Mo à plusieurs Go.
- Il faut de la RAM, du CPU/GPU, et l’expérience mobile devient lourde : téléchargement, batterie, chauffe, latence.
- Les petits modèles embarqués existent, mais la qualité pour une aventure longue et cohérente est généralement insuffisante.
- Mettre une clé API dans le client est une mauvaise pratique de sécurité : il faut un backend proxy.

### Conséquences
- Poids de l’app : quelques Mo.
- Poids réel du système : le LLM est externalisé ; le coût est côté serveur/API.
- Mode hors-ligne : possible pour le moteur et les sauvegardes, pas pour la narration LLM, sauf à embarquer un petit modèle local — et alors le poids explose.
- Jeu “illimité” : oui, avec une architecture client léger + serveur/API + état local structuré.

---

## Conclusion

- Un jeu d’aventure textuel peut durer bien plus que 1–2 h.
- Un jeu persistant/illimitié sur mobile est possible en PWA.
- Une PWA légère de quelques Mo est réaliste.
- Le LLM n’est pas embarqué dans ces quelques Mo : il est appelé via une API.
- L’architecture idéale est hybride : moteur déterministe local + LLM narrateur côté serveur + état structuré et sauvegardé localement.
- La cohérence, la mémoire et la persistance des données sont les vrais défis, pas la durée théorique.
