---
Title: Git et GitHub pour Débutants en 2026 : Guide Complet
Description: 
Author: Lucas M Dev
Date: 2026-03-28T21:49:00.000Z
Robots: noindex,nofollow
Template: index
---
<p>Git et GitHub pour Débutants en 2026 : Guide Complet avec Commandes </p>

<p>Git est l'outil de contrôle de version que tout développeur doit maîtriser. En 2026, ne pas savoir utiliser Git est un dealbreaker en entreprise. Ce guide vous explique tout de zéro, avec les commandes que vous utiliserez vraiment au quotidien. </p>

<h2>
  
  
  Pourquoi Git est indispensable
</h2>

<p>Git permet de : </p>

<ul>
<li>Sauvegarder l'historique complet de votre code</li>
<li>Travailler à plusieurs sans se marcher dessus</li>
<li>Revenir à une version précédente si quelque chose casse</li>
<li>Expérimenter dans des branches séparées</li>
<li>Contribuer à des projets open-source</li>
</ul>

<h2>
  
  
  Installation et configuration
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Linux</span>
<span class="nb">sudo </span>apt <span class="nb">install </span>git

<span class="c"># Mac (via Homebrew)</span>
brew <span class="nb">install </span>git

<span class="c"># Windows</span>
<span class="c"># Télécharger Git for Windows : git-scm.com</span>

<span class="c"># Configuration initiale (obligatoire)</span>
git config <span class="nt">--global</span> user.name <span class="s2">"Votre Nom"</span>
git config <span class="nt">--global</span> user.email <span class="s2">"vous@example.com"</span>
git config <span class="nt">--global</span> init.defaultBranch main <span class="c"># Branche par défaut</span>

<span class="c"># Vérifier la config</span>
git config <span class="nt">--list</span>
</code></pre>

</div>



<h2>
  
  
  Les concepts fondamentaux
</h2>

<h3>
  
  
  Le Repository (Repo)
</h3>

<p>Un repo Git est un dossier suivi par Git. Il contient votre code + un historique de toutes les modifications. </p>

<h3>
  
  
  Les 3 zones de Git
</h3>

<ul>
<li>
<strong>Working Directory</strong> : vos fichiers tels qu'ils sont sur le disque</li>
<li>
<strong>Staging Area (Index)</strong> : ce que vous avez préparé pour le prochain commit</li>
<li>
<strong>Repository</strong> : l'historique des commits sauvegardés</li>
</ul>

<h2>
  
  
  Les commandes essentielles
</h2>

<h3>
  
  
  Initialiser et créer un repo
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Initialiser Git dans un dossier existant</span>
<span class="nb">cd </span>mon-projet
git init

<span class="c"># Ou cloner un repo existant</span>
git clone https://github.com/user/repo.git
git clone https://github.com/user/repo.git mon-dossier-local
</code></pre>

</div>



<h3>
  
  
  Le cycle basique add → commit → push
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Voir l'état de votre repo</span>
git status

<span class="c"># Ajouter des fichiers au staging</span>
git add fichier.txt <span class="c"># Un fichier</span>
git add src/ <span class="c"># Un dossier entier</span>
git add <span class="nb">.</span> <span class="c"># Tous les fichiers modifiés</span>

<span class="c"># Créer un commit</span>
git commit <span class="nt">-m</span> <span class="s2">"feat: ajouter la page de connexion"</span>

<span class="c"># Voir l'historique</span>
git log
git log <span class="nt">--oneline</span> <span class="c"># Format compact</span>
git log <span class="nt">--oneline</span> <span class="nt">--graph</span> <span class="c"># Avec branches visuelles</span>

<span class="c"># Envoyer sur GitHub</span>
git push origin main
</code></pre>

</div>



<h3>
  
  
  Messages de commit — Bonne pratique
</h3>

<p>La convention <strong>Conventional Commits</strong> est la plus répandue en 2026 :<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>feat: ajouter la fonctionnalité de recherche
fix: corriger le bug de connexion sur mobile
docs: mettre à jour le README
style: formater le code avec Prettier
refactor: simplifier la logique d'authentification
test: ajouter les tests unitaires pour userService
chore: mettre à jour les dépendances
</code></pre>

</div>



<h2>
  
  
  Travailler avec des branches
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Lister les branches</span>
git branch

<span class="c"># Créer une branche</span>
git branch feature/page-contact

<span class="c"># Créer ET basculer dessus</span>
git checkout <span class="nt">-b</span> feature/page-contact
<span class="c"># Ou en moderne Git :</span>
git switch <span class="nt">-c</span> feature/page-contact

<span class="c"># Basculer entre branches</span>
git switch main
git switch feature/page-contact

<span class="c"># Fusionner une branche dans main</span>
git switch main
git merge feature/page-contact

<span class="c"># Supprimer une branche</span>
git branch <span class="nt">-d</span> feature/page-contact <span class="c"># Après merge</span>
git branch <span class="nt">-D</span> feature/page-contact <span class="c"># Force (sans merge)</span>
</code></pre>

</div>



<h2>
  
  
  Corriger des erreurs courantes
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Annuler les modifications non commitées</span>
git restore fichier.txt <span class="c"># Un fichier</span>
git restore <span class="nb">.</span> <span class="c"># Tout</span>

<span class="c"># Retirer un fichier du staging</span>
git restore <span class="nt">--staged</span> fichier.txt

<span class="c"># Annuler le dernier commit (garder les modifs)</span>
git reset <span class="nt">--soft</span> HEAD~1

<span class="c"># Modifier le dernier commit (message ou contenu)</span>
git commit <span class="nt">--amend</span> <span class="nt">-m</span> <span class="s2">"Nouveau message"</span>

<span class="c"># Voir les différences</span>
git diff <span class="c"># Modifs non stagées</span>
git diff <span class="nt">--staged</span> <span class="c"># Modifs stagées</span>
</code></pre>

</div>



<h2>
  
  
  Travailler avec GitHub
</h2>

<h3>
  
  
  Connecter votre repo local à GitHub
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># 1. Créer un repo sur github.com</span>
<span class="c"># 2. Connecter votre repo local</span>
git remote add origin https://github.com/vous/votre-repo.git
git branch <span class="nt">-M</span> main
git push <span class="nt">-u</span> origin main
</code></pre>

</div>



<h3>
  
  
  Pull Requests — Le workflow pro
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Workflow standard en équipe</span>
git switch main
git pull origin main <span class="c"># Mettre à jour local</span>
git switch <span class="nt">-c</span> feature/nouvelle-fonctionnalite

<span class="c"># ... travaillez sur votre feature ...</span>
git add <span class="nb">.</span>
git commit <span class="nt">-m</span> <span class="s2">"feat: ajouter la fonctionnalité"</span>
git push origin feature/nouvelle-fonctionnalite

<span class="c"># Ensuite, créez une Pull Request sur GitHub</span>
<span class="c"># Demandez une review à vos collègues</span>
<span class="c"># Après approbation : merge dans main</span>
</code></pre>

</div>



<h3>
  
  
  Récupérer les dernières modifications
</h3>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code><span class="c"># Récupérer et fusionner</span>
git pull origin main

<span class="c"># Récupérer sans fusionner</span>
git fetch origin

<span class="c"># Rebase — réappliquer vos commits sur la version la plus récente</span>
git fetch origin
git rebase origin/main
</code></pre>

</div>



<h2>
  
  
  Le .gitignore — Ce qu'on ne commit JAMAIS
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight properties"><code><span class="c"># Exemple de .gitignore pour un projet Node.js
</span><span class="err">node_modules/</span>
<span class="err">.env</span>
<span class="err">.env.local</span>
<span class="err">.env.production</span>
<span class="err">dist/</span>
<span class="err">build/</span>
<span class="err">*.log</span>
<span class="err">.DS_Store</span> <span class="c"># Mac
</span><span class="err">Thumbs.db</span> <span class="c"># Windows
</span><span class="err">.vscode/</span> <span class="c"># Configuration IDE (optionnel)
</span><span class="err">coverage/</span> <span class="c"># Rapports de tests
</span></code></pre>

</div>



<h2>
  
  
  Commandes Git que j'utilise quotidiennement
</h2>



<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>git status <span class="c"># Où j'en suis</span>
git add <span class="nb">.</span> <span class="c"># Tout stager</span>
git commit <span class="nt">-m</span> <span class="s2">"..."</span> <span class="c"># Committer</span>
git push <span class="c"># Envoyer sur GitHub</span>
git pull <span class="c"># Récupérer les dernières modifs</span>
git log <span class="nt">--oneline</span> <span class="c"># Historique rapide</span>
git switch <span class="nt">-c</span> branch <span class="c"># Créer une branche</span>
git diff <span class="c"># Voir les changements</span>
</code></pre>

</div>



<h2>
  
  
  Ressources pour aller plus loin
</h2>

<ul>
<li>
<strong>Pro Git</strong> (gratuit en ligne) : la référence absolue sur Git</li>
<li>
<strong>Oh My Git!</strong> : jeu interactif pour apprendre Git</li>
<li>
<strong>Learn Git Branching</strong> : visualisation interactive des branches</li>
<li>
<strong>GitHub Skills</strong> : cours officiels GitHub gratuits</li>
</ul>

<p><a href="//deployer-app-gratuitement-2026.html">→ Déployer votre app avec Railway, Render, Fly.io</a> <br>
 <a href="//github-pages-blog-gratuit-2026.html">→ Créer un blog gratuit avec GitHub Pages</a></p>

