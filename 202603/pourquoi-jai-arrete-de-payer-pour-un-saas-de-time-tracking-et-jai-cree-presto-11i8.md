---
Title: Pourquoi j'ai arrêté de payer pour un SaaS de time-tracking et j'ai créé Presto
Description: 
Author: Walid Karray
Date: 2026-03-05T21:34:13.000Z
Robots: noindex,nofollow
Template: index
---
<p><strong>Vous êtes freelance. Vous suivez vos jours facturés. Vous envoyez des comptes rendus d'activité à vos clients. Pourquoi vos outils ressemblent-ils à ceux d'une boîte de 500 personnes ?</strong></p>

<p>La plupart des outils de suivi du temps sont conçus pour des équipes, des managers et des workflows d'entreprise. Ils viennent avec une tarification au siège, des tableaux de bord surchargés et des fonctionnalités que vous n'utiliserez jamais. Tout ce dont vous avez besoin, c'est d'un CRA mensuel propre indiquant les jours travaillés — et éventuellement un PDF à joindre à votre facture.</p>

<p>C'est exactement ce que fait <strong>Presto</strong>. Ni plus, ni moins.</p>




<h2>
  
  
  Qu'est-ce que Presto ?
</h2>

<p><a href="https://github.com/tux86/presto" rel="noopener noreferrer">Presto</a> est un générateur de comptes rendus d'activité open-source et auto-hébergé. Il vous permet de :</p>

<ul>
<li>Suivre vos jours facturés par client et mission sur un <strong>calendrier visuel</strong>
</li>
<li>Générer des <strong>CRA au format PDF</strong> professionnels, prêts à envoyer</li>
<li>Suivre vos <strong>revenus et votre taux d'occupation</strong> avec des tableaux de bord intégrés</li>
<li>Le faire tourner sur <strong>votre propre serveur</strong> avec une seule commande Docker</li>
</ul>

<p>Il est sous licence MIT, entièrement autonome, et se déploie en moins de 2 minutes.</p>




<h2>
  
  
  Le tableau de bord : votre année en un coup d'œil
</h2>

<p>À l'ouverture de Presto, vous accédez directement à votre <strong>tableau de bord d'activité</strong> — une vue en cartes de tous vos CRA de l'année, regroupés par client.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F7m1q8hnbmpgbwc34vadl.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F7m1q8hnbmpgbwc34vadl.png" alt="Tableau de bord Presto — Cartes de CRA regroupées par client, avec aperçu calendrier et chiffre d'affaires" width="800" height="500"></a></p>

<p>Chaque carte affiche le mois, le nom de la mission, un <strong>mini aperçu calendrier</strong> (pour visualiser d'un coup d'œil les jours travaillés), le nombre de jours facturés et le montant. Les rapports sont colorés par client et marqués comme Brouillon ou Validé.</p>

<p>Vous pouvez <strong>filtrer par société ou par client</strong>, naviguer entre les années et créer un nouveau CRA en deux clics.</p>

<p>Pas de surcharge. Pas de diagrammes de Gantt. Juste votre travail, organisé par mois.</p>




<h2>
  
  
  L'éditeur de CRA : cliquez, remplissez, c'est fait
</h2>

<p>Le cœur de Presto, c'est l'<strong>éditeur de CRA basé sur un calendrier</strong>. Chaque jour du mois est une case cliquable — un clic pour une journée complète, un deuxième pour une demi-journée, un troisième pour effacer. C'est tout.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftlo94xxrfu1z7a63i49w.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Ftlo94xxrfu1z7a63i49w.png" alt="Éditeur de CRA Presto — Vue calendrier avec journées complètes, demi-journées et jours fériés" width="800" height="500"></a></p>

<p>Le panneau latéral affiche un récapitulatif en temps réel : jours travaillés sur les jours ouvrés disponibles, votre TJM et le montant total. Quand un rapport est finalisé, passez-le en <strong>Validé</strong> et cliquez sur <strong>Exporter en PDF</strong> pour générer un document prêt à imprimer.</p>

<h3>
  
  
  Des fonctionnalités qui font gagner du temps :
</h3>

<ul>
<li>
<strong>Remplissage automatique</strong> — Un clic remplit tous les jours ouvrés du mois. Les week-ends et jours fériés sont automatiquement exclus.</li>
<li>
<strong>Jours fériés par pays</strong> — Définissez un pays de jours fériés par client (France, Allemagne, Luxembourg… tous les pays sont supportés). Les jours fériés apparaissent sous forme de points jaunes sur le calendrier.</li>
<li>
<strong>Précision à la demi-journée</strong> — Parce que parfois, on ne travaille qu'une matinée.</li>
<li>
<strong>Notes par jour</strong> — Ajoutez du contexte à des jours spécifiques (réunions, livrables, déplacements sur site).</li>
<li>
<strong>Workflow Brouillon / Validé</strong> — Les rapports validés sont verrouillés. Plus de modifications accidentelles.</li>
</ul>




<h2>
  
  
  Reporting : connaître ses chiffres
</h2>

<p>La page <strong>Reporting</strong> vous donne une vue d'ensemble de votre activité freelance sur l'année.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fkp1owgpmfy0gu15gp5qz.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fkp1owgpmfy0gu15gp5qz.png" alt="Tableau de bord Reporting Presto — KPIs, graphiques de revenus et jauge d'occupation" width="800" height="500"></a></p>

<p>Quatre indicateurs clés en haut de page :</p>

<ul>
<li>
<strong>Jours travaillés</strong> — avec comparaison année sur année et courbe de tendance</li>
<li>
<strong>Chiffre d'affaires</strong> — montant total avec conversion multi-devises automatique</li>
<li>
<strong>Clients actifs</strong> — combien de clients facturés cette année</li>
<li>
<strong>Taux d'occupation</strong> — une jauge indiquant le pourcentage de jours ouvrés effectivement facturés</li>
</ul>

<p>En dessous, des graphiques interactifs détaillent le <strong>CA par client</strong> (barres empilées), la <strong>répartition clients</strong> (donut) et les tendances mensuelles en revenus et en jours.</p>

<p>Si vous travaillez avec des clients dans différentes devises (EUR, USD, GBP…), Presto convertit automatiquement tout dans votre <strong>devise de référence</strong> grâce aux taux de change en temps réel. Un seul chiffre, clair et net.</p>




<h2>
  
  
  Export PDF : des CRA professionnels en quelques secondes
</h2>

<p>Quand un rapport est validé, Presto génère un <strong>PDF côté serveur</strong> via React PDF. Le document inclut :</p>

<ul>
<li>Les coordonnées de votre société et les informations du client</li>
<li>Une grille calendrier propre avec les jours travaillés en surbrillance</li>
<li>Les totaux : jours travaillés, TJM, montant total</li>
<li>Une section notes</li>
<li>Disponible en <strong>5 langues</strong> : français, anglais, allemand, espagnol, portugais</li>
</ul>

<p>Le nom du fichier est généré automatiquement (ex. <code>Acme_Corp-Projet_API-2025-12.pdf</code>). Téléchargez, joignez à votre email de facturation, terminé.</p>




<h2>
  
  
  Multi-clients, multi-devises, multilingue
</h2>

<p>Presto est conçu pour les freelances qui jonglent entre plusieurs clients à l'international :</p>

<ul>
<li>
<strong>Sociétés</strong> — Enregistrez vos propres entités juridiques (utile si vous facturez sous plusieurs raisons sociales)</li>
<li>
<strong>Clients</strong> — Chaque client a sa propre devise, son pays de jours fériés, sa couleur et ses coordonnées</li>
<li>
<strong>Missions</strong> — Chaque mission (projet/contrat) a un TJM et est rattachée à un client</li>
<li>
<strong>Devises</strong> — Toutes les devises ISO 4217 supportées. Les tableaux de bord convertissent automatiquement.</li>
<li>
<strong>Langues</strong> — Interface complète en français, anglais, allemand, espagnol et portugais</li>
</ul>




<h2>
  
  
  Déploiement en 60 secondes
</h2>

<p>Presto est distribué sous forme d'une <strong>image Docker unique</strong>. Pas de microservices, pas de conteneur frontend séparé, pas de Redis, pas de file de messages. Juste l'application et PostgreSQL.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="c1"># docker-compose.yml</span>
<span class="na">services</span><span class="pi">:</span>
  <span class="na">presto</span><span class="pi">:</span>
    <span class="na">image</span><span class="pi">:</span> <span class="s">axforge/presto:latest</span>
    <span class="na">ports</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s2">"</span><span class="s">8080:8080"</span>
    <span class="na">environment</span><span class="pi">:</span>
      <span class="na">DATABASE_URL</span><span class="pi">:</span> <span class="s">postgresql://presto:secret@db:5432/presto</span>
      <span class="na">JWT_SECRET</span><span class="pi">:</span> <span class="s">votre-cle-secrete-min-32-caracteres</span>
    <span class="na">depends_on</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">db</span>

  <span class="na">db</span><span class="pi">:</span>
    <span class="na">image</span><span class="pi">:</span> <span class="s">postgres:17-alpine</span>
    <span class="na">environment</span><span class="pi">:</span>
      <span class="na">POSTGRES_USER</span><span class="pi">:</span> <span class="s">presto</span>
      <span class="na">POSTGRES_PASSWORD</span><span class="pi">:</span> <span class="s">secret</span>
      <span class="na">POSTGRES_DB</span><span class="pi">:</span> <span class="s">presto</span>
    <span class="na">volumes</span><span class="pi">:</span>
      <span class="pi">-</span> <span class="s">pgdata:/var/lib/postgresql/data</span>

<span class="na">volumes</span><span class="pi">:</span>
  <span class="na">pgdata</span><span class="pi">:</span>
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight shell"><code>docker compose up <span class="nt">-d</span>
</code></pre>

</div>



<p>Ouvrez <code>http://localhost:8080</code>, créez un compte et commencez à saisir vos CRA.</p>

<h3>
  
  
  Mode mono-utilisateur
</h3>

<p>Si vous êtes le seul utilisateur, désactivez l'authentification :<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">environment</span><span class="pi">:</span>
  <span class="na">AUTH_DISABLED</span><span class="pi">:</span> <span class="s2">"</span><span class="s">true"</span>
</code></pre>

</div>



<p>Pas d'écran de connexion. Pas de mot de passe. Ouvrez l'application et c'est parti.</p>

<h3>
  
  
  Verrouiller après la mise en place
</h3>

<p>Votre compte est créé ? Désactivez l'inscription pour que personne d'autre ne puisse s'inscrire :<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight yaml"><code><span class="na">environment</span><span class="pi">:</span>
  <span class="na">REGISTRATION_ENABLED</span><span class="pi">:</span> <span class="s2">"</span><span class="s">false"</span>
</code></pre>

</div>






<h2>
  
  
  La stack technique
</h2>

<p>Pour les curieux :</p>

<ul>
<li>
<strong>Runtime</strong> — Bun</li>
<li>
<strong>Frontend</strong> — React 19, Vite 6, Tailwind CSS 4, Zustand, TanStack Query</li>
<li>
<strong>Backend</strong> — Hono, Drizzle ORM</li>
<li>
<strong>Base de données</strong> — PostgreSQL</li>
<li>
<strong>PDF</strong> — @react-pdf/renderer (rendu côté serveur)</li>
<li>
<strong>Langage</strong> — TypeScript 5.7, mode strict</li>
<li>
<strong>Tests</strong> — 222 tests E2E API</li>
</ul>

<p>Les migrations s'exécutent automatiquement au démarrage. L'image Docker est un build multi-stage produisant un conteneur léger basé sur Alpine.</p>




<h2>
  
  
  À qui s'adresse Presto ?
</h2>

<p>Presto est fait pour les <strong>freelances et consultants indépendants</strong> qui :</p>

<ul>
<li>Facturent à la journée (ou demi-journée)</li>
<li>Doivent soumettre des CRA mensuels</li>
<li>Veulent suivre leur CA sur plusieurs clients et devises</li>
<li>Préfèrent les outils auto-hébergés aux abonnements SaaS</li>
<li>Privilégient la simplicité à l'usine à gaz</li>
</ul>

<p>Si vous utilisez des tableurs, des templates Notion ou des outils de time tracking hors de prix pour faire ce qui est fondamentalement une tâche simple — Presto est votre porte de sortie.</p>




<h2>
  
  
  Pour commencer
</h2>

<ul>
<li>
<strong>GitHub</strong> : <a href="https://github.com/tux86/presto" rel="noopener noreferrer">github.com/tux86/presto</a>
</li>
<li>
<strong>Docker Hub</strong> : <a href="https://hub.docker.com/r/axforge/presto" rel="noopener noreferrer">hub.docker.com/r/axforge/presto</a>
</li>
<li>
<strong>Licence</strong> : MIT — gratuit pour toujours, sans conditions</li>
</ul>

<p>Mettez une étoile au repo si vous trouvez ça utile. Les contributions et retours sont les bienvenus.</p>




<p><em>Presto est open-source et activement maintenu. Conçu par un freelance, pour les freelances.</em></p>

