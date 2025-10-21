# hackbench-cybernova-2025
Projet Hackathon Ynov 2025

Hackathon


L’entreprise CYBERNOVA, spécialisée dans l’audit de sécurité et la réponse à incident, vient d’obtenir un nouveau contrat pour tester la robustesse d’un réseau interne simulé.
Nous sommes donc invités à participer à un hackathon interne de cybersécurité :
une épreuve de 48 heures durant laquelle vous devrez concevoir, attaquer, défendre et documenter un environnement technique complet.
Objectif : Reproduire le cycle complet d’un audit de sécurité sur un intranet vulnérable, en conditions réelles, depuis la découverte initiale jusqu’à la remédiation.

Étape 1 : Création du repository sur Github.

a) Créez le repository sur Github, en cliquant sur “New” 

b) Donnez un nom à votre repository, une description

**ATTENTION : Si vous utilisez Replit version gratuite, Veillez à mettre votre repository en visibilité “Public”**
c) Pour terminer, “Create repository”
Étape 2 : Création du projet sur VS Code.
Dans un premier temps, vous aller devoir lier votre github a votre projet dans Vs code. 

Pour ce faire, vous devez : 
 a) aller sur VS Code et appuyer sur Ctrl+Shift+P

 b) Tapez : “Git:Clone”

 c) Coller l’URL : https://github.com/VOTRE-NOM/hackbench-cybernova-2025.git

Quand votre projet sera ouvert, Reproduisez cette arborescence dans VS Code.



d) Copier-coller ces lignes de code aux endroits indiqué : 

Dans le dossier replit_template/ mettre les fichiers suivants.
index.js
// index.js - Intranet RH demo 
// But : fournir une petite appli vulnérable (non destructive)
const express = require('express');
const fs = require('fs');
const path = require('path');
const app = express();

app.use(express.urlencoded({ extended: true }));
app.use(express.json());
app.use(express.static('public'));

// Page d'accueil
app.get('/', (req, res) => {
  res.send(`
    <h1>Intranet RH - Demo</h1>
    <p>Bienvenue sur l'intranet de démonstration. Utilisez le formulaire pour rechercher un employé.</p>
    <form method="POST" action="/search">
      <input name="q" placeholder="Nom ou partie du nom" />
      <button>Search</button>
    </form>
    <p>Endpoints utiles : <code>/search</code> (POST), <code>/admin</code> (protected), <code>/flag</code> (secret)</p>
  `);
});

// Route "search" - vulnérable par conception (recherche naïve)
app.post('/search', (req, res) => {
  const q = (req.body.q || '').toLowerCase();
  // Lecture simple du fichier users.txt (pédagogique)
  const users = fs.readFileSync(path.join(__dirname, 'data', 'users.txt'), 'utf8')
                  .split(/\r?\n/).filter(Boolean);
  // Naïve filtering - pas d'échappement ni de limitation
  const hits = users.filter(u => u.toLowerCase().includes(q));
  res.send(`<h2>Résultats</h2><p>Query: <code>${escapeHtml(q)}</code></p><pre>${hits.join('\n') || 'Aucun'}</pre>`);
});

// Endpoint admin - simulation d'un accès protégé par token faible
app.get('/admin', (req, res) => {
  // token transmis en query ? exemple : /admin?token=admintoken
  const token = req.query.token || '';
  // token en clair dans le code (vulnérable volontairement)
  if (token === 'admintoken123') {
    res.send(`<h1>Console Admin</h1><p>Bienvenue, administrateur.</p>`);
  } else {
    res.status(401).send(`<h1>401 Unauthorized</h1><p>Token manquant ou invalide.</p>`);
  }
});

// Endpoint flag (preuve pédagogique)
app.get('/flag', (req, res) => {
  // on sert le fichier flag si demandé
  res.download(path.join(__dirname, 'public', 'flag.txt'), 'flag.txt', (err) => {
    if (err) res.status(500).send('Erreur lecture flag');
  });
});

// Simple helper to avoid XSS in displayed query (very minimal)
function escapeHtml(str) {
  return String(str).replace(/[&<>"'`=\/]/g, s => ({
    '&':'&amp;', '<':'&lt;','>':'&gt;','"':'&quot;', "'":'&#39;','/':'&#x2F;','`':'&#x60;','=':'&#x3D;'
  })[s]);
}

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => console.log(`Intranet demo listening on port ${PORT}`));

package.json
{
  "name": "intranet-rh-demo",
  "version": "1.0.0",
  "main": "index.js",
  "scripts": {
    "start": "node index.js"
  },
  "dependencies": {
    "express": "^4.18.2"
  }
}

data/users.txt
Alice Martin - alice.martin@novatech.local - RH
Bob Durand - bob.durand@novatech.local - Dev
Charlie Lefevre - charlie.lefevre@novatech.local - Ops
David Bernard - david.bernard@novatech.local - Finance
Eve Dupont - eve.dupont@novatech.local - QA
Frank Leroy - frank.leroy@novatech.local - Support

public/flag.txt
FLAG-NOVATECH-48H-DEMO-2025

README.md
# Intranet RH - Demo (HackBench)

## But
Application : intranet RH minimal volontairement vulnérable.  
Utilisez-la uniquement dans le cadre du HackBench (48h). N'attaquez que votre propre instance.

## Endpoints
- `GET /` : page d'accueil + formulaire de recherche
- `POST /search` : recherche naïve dans `data/users.txt`
- `GET /admin?token=...` : endpoint admin simulé (token en clair)
- `GET /flag` : télécharge le flag pédagogique

## Objectifs pédagogiques
1. Trouver et documenter les vulnérabilités (ex : recherche non sécurisée, token en clair).  
2. Récupérer le flag (preuve) et documenter la méthode (commands.txt, captures).  
3. Proposer et implémenter des correctifs (validation input, stockage sécurisé du token, auth).

## Déploiement sur Replit
1. Fork / Importer ce repo dans Replit (Import from GitHub).  
2. Cliquer sur **Run**.  
3. Récupérer l'URL publique fournie par Replit (ex : `https://<votre-projet>.<votre-compte>.repl.co`).

## Règles
- Vous ne testez que votre instance.  
- Documentez **toutes** les commandes exécutées dans `evidence/commands.txt`.  
- Déclarez toute utilisation d'IA (outil + prompt) dans votre rapport.

## Remédiations attendues (suggestions)
- Ajout d'une authentification véritable (session + mot de passe hashed).  
- Validation stricte des entrées/sanitization.  
- Ne pas stocker token admin en clair dans l'URL.

Étape 3 : Push le projet sur votre repository Github.
Pour pouvoir “push” votre projet sur votre repository, vous devez : 
a) aller dans “source control” et appuyer sur “commit”

b) donner un nom a votre commit et appuyer sur valider en haut à droite 




Dès que cela est fini, rafraichissez votre page github et vous pourrez apercevoir que votre projet est bien dans votre repository.


Étape 4 : Héberger le projet avec Replit. (Optionnel)

Pour éviter d’héberger votre application en local, il est possible de l’héberger par le site Replit.

a) Créer un compte Replit

b) Une fois dans l’accueil Cliquer sur “Import code or design”

c) Sélectionnez Github

d) Sélectionnez votre repository et faites “Import from Github”

e) Laissez Replit, installer les dépendances et démarrer l’application

 Vous devrez avoir ce résultat : 



f) Faites “Open a new tab” pour ouvrir l’application sur internet.

Étape 5 : L’attaque de l’application
Pour récupérer les informations de l’application, il suffit de se rendre sur son terminal (CMD)
et de taper cette commande : 

curl https://b452551e-a583-4f7f-9c74-32ac1b2fd606-00-105vtajd5ey1c.spock.replit.dev

(commande effectuer sur un pc externe)
curl est un outil en ligne de commande qui permet d’envoyer des requêtes HTTP(S) (et d’autres protocoles) et de récupérer la réponse.
En regardant le résultat, on peut voir en bas du code de la page index plusieurs autres pages: search, admin, flag.

Ce qui nous interesse, c’est le flag

Essayons un curl avec “/flag”



Le flag apparait bien, l’attaque est donc réussie.



