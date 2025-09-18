---
Title: Créer un Univers avec Python : Une Simulation Inspirée par la Poésie et le Cosmos
Description: 
Author: Khadidja Bounadja
Date: 2025-09-18T22:00:01.000Z
Robots: noindex,nofollow
Template: index
---
<p>J'ai toujours pensé que les théories les plus complexes pouvaient être comprises avec des analogies simples. Guidé par mon intuition et inspiré par des textes anciens, j'ai voulu créer une simulation qui donne vie à une idée fascinante : et si l'univers était un organisme vivant, un immense cerveau ?<br>
Ce projet explore la naissance, la mort et le cycle de la matière, non pas avec des équations complexes, mais avec la logique d'un simple code Python.<br>
Le Concept : De la Poésie au Code<br>
Ma théorie repose sur quelques principes :<br>
La Matière Noire et l'Énergie Noire : La matière noire est la "structure" de l'univers, une obscurité dormante. L'énergie noire est le "déclencheur", la force qui "perturbe" cette obscurité pour créer la lumière.<br>
La Naissance de l'Étoile ("Pouf Chocapic") : Une étoile naît quand la matière "soupe" consomme assez d'énergie noire en un point précis. C'est le moment de la création.<br>
Le Cycle Infini : Une étoile est comme une plante qui naît, se nourrit, meurt et dont les résidus servent à créer de nouvelles planètes, perpétuant le cycle.<br>
Pour visualiser ces idées, j'ai utilisé Python pour modéliser une petite portion de mon univers.<br>
La Simulation : Comment le Code Fonctionne<br>
Le code utilise la librairie numpy pour créer une grille (notre univers) et matplotlib pour en faire une image. Le processus est simple :<br>
Création du Vide Cosmique : Une grille est remplie d'une valeur de base représentant le potentiel de l'énergie noire.<br>
L'Action de la Matière : Des particules de matière sont placées aléatoirement et commencent à "soustraire" de l'énergie noire.<br>
La Naissance : Lorsque le potentiel d'énergie noire d'un point atteint un seuil critique, une étoile est née ! Le code capture cet instant et crée une image.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fx018aqnj9ht6jd8yu2c3.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fx018aqnj9ht6jd8yu2c3.jpg" alt=" " width="800" height="800"></a></p>

<p>Voici le code complet. Les commentaires expliquent chaque étape.</p>

<p>[4]</p>

<h1>
  
  
  -<em>- coding: utf-8 -</em>-
</h1>

<p>import numpy as np<br>
import random<br>
import math</p>

<h1>
  
  
  --- Règles de l'univers ---
</h1>

<p>force_de_lumiere = 3 * 3<br>
nombre_particules_matiere = 200<br>
valeur_energie_noire_de_base = 20<br>
grid_size = 50</p>

<h1>
  
  
  --- Création du cosmos ---
</h1>

<p>cosmic_void = np.full((grid_size, grid_size), valeur_energie_noire_de_base)<br>
particules = [{'pos': (random.randint(0, grid_size - 1), random.randint(0, grid_size - 1)), 'force': 1} for _ in range(nombre_particules_matiere)]</p>

<p>print("Le cosmos est créé, avec un potentiel d'énergie noire élevé.")</p>

<h1>
  
  
  --- Simulation de la naissance ---
</h1>

<p>etoile_creee = False<br>
for temps in range(1, 101):<br>
    nouvelle_grille = np.copy(cosmic_void)</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>for p in particules:
    nouvelle_grille[p['pos'][0], p['pos'][1]] -= p['force']

for y in range(grid_size):
    for x in range(grid_size):
        if nouvelle_grille[y, x] &lt;= (valeur_energie_noire_de_base - force_de_lumiere):
            masse_etoile = valeur_energie_noire_de_base - nouvelle_grille[y, x]
            print(f"\nPOUF CHOCAPIC ! Une étoile est née à ({y}, {x}) avec une masse de {masse_etoile:.2f} !")
            etoile_creee = True
            break
    if etoile_creee: break
if etoile_creee: break

if temps % 10 == 0:
    print(f"Étape {temps} : La création continue...")
</code></pre>

</div>

<p>print("\nLa simulation de la naissance est terminée.")<br>
Le cosmos est créé, avec un potentiel d'énergie noire élevé.<br>
Étape 10 : La création continue...<br>
Étape 20 : La création continue...<br>
Étape 30 : La création continue...<br>
Étape 40 : La création continue...<br>
Étape 50 : La création continue...<br>
Étape 60 : La création continue...<br>
Étape 70 : La création continue...<br>
Étape 80 : La création continue...<br>
Étape 90 : La création continue...<br>
Étape 100 : La création continue...</p>

<p>La simulation de la naissance est terminée.</p>

<p>[2]<br>
1 s</p>

<h1>
  
  
  -<em>- coding: utf-8 -</em>-
</h1>

<p>import random<br>
import time</p>

<h1>
  
  
  --- Règles de l'univers ---
</h1>

<p>masse_critique = 50<br>
naine_blanche_seuil = 20<br>
etoile_neutrons_seuil = 40</p>

<h1>
  
  
  --- Création de l'étoile ---
</h1>

<p>etoile_masse = 100</p>

<p>print(f"Une étoile est née avec une masse de {etoile_masse}.")</p>

<h1>
  
  
  --- Simulation de la mort ---
</h1>

<p>while etoile_masse &gt; masse_critique:<br>
    perte_masse = random.uniform(1, 5)<br>
    etoile_masse -= perte_masse<br>
    print(f"L'étoile s'use... Masse restante : {etoile_masse:.2f}")<br>
    time.sleep(0.1)</p>

<h1>
  
  
  --- Le destin final ---
</h1>

<p>print(f"\nL'étoile a atteint la masse critique de {masse_critique} !")</p>

<p>if etoile_masse &lt; naine_blanche_seuil:<br>
    print("L'étoile a terminé son cycle et est devenue : Naine Blanche")<br>
elif etoile_masse &lt; etoile_neutrons_seuil:<br>
    print("L'étoile a terminé son cycle et est devenue : Étoile à Neutrons")<br>
else:<br>
    print("L'étoile a terminé son cycle et est devenue : Trou Noir")</p>

<p>print("\nLe cycle de vie et de mort de l'étoile est terminé.")<br>
Une étoile est née avec une masse de 100.<br>
L'étoile s'use... Masse restante : 97.76<br>
L'étoile s'use... Masse restante : 95.43<br>
L'étoile s'use... Masse restante : 93.23<br>
L'étoile s'use... Masse restante : 89.64<br>
L'étoile s'use... Masse restante : 88.06<br>
L'étoile s'use... Masse restante : 84.97<br>
L'étoile s'use... Masse restante : 79.97<br>
L'étoile s'use... Masse restante : 77.70<br>
L'étoile s'use... Masse restante : 74.58<br>
L'étoile s'use... Masse restante : 70.17<br>
L'étoile s'use... Masse restante : 67.67<br>
L'étoile s'use... Masse restante : 63.38<br>
L'étoile s'use... Masse restante : 60.06<br>
L'étoile s'use... Masse restante : 56.44<br>
L'étoile s'use... Masse restante : 55.20<br>
L'étoile s'use... Masse restante : 53.17<br>
L'étoile s'use... Masse restante : 50.17<br>
L'étoile s'use... Masse restante : 48.06</p>

<p>L'étoile a atteint la masse critique de 50 !<br>
L'étoile a terminé son cycle et est devenue : Trou Noir</p>

<p>Le cycle de vie et de mort de l'étoile est terminé.</p>

<p>[3]<br>
0 s</p>

<h1>
  
  
  -<em>- coding: utf-8 -</em>-
</h1>

<p>import numpy as np<br>
import random<br>
import math<br>
import time</p>

<h1>
  
  
  --- Règles de l'univers ---
</h1>

<p>force_de_lumiere = 3 * 3<br>
nombre_particules_matiere = 200<br>
valeur_energie_noire_de_base = 20<br>
grid_size = 50<br>
masse_critique = 5</p>

<h1>
  
  
  --- Création du cosmos ---
</h1>

<p>cosmic_void = np.full((grid_size, grid_size), valeur_energie_noire_de_base)<br>
particules = [{'pos': (random.randint(0, grid_size - 1), random.randint(0, grid_size - 1)), 'force': 1} for _ in range(nombre_particules_matiere)]</p>

<p>print("Le cosmos est créé, avec un potentiel d'énergie noire élevé.")</p>

<h1>
  
  
  --- Fonctions pour le cycle ---
</h1>

<p>def vie_etoile(etoile_masse):<br>
    perte_masse = random.uniform(0.1, 0.5)<br>
    etoile_masse -= perte_masse<br>
    return etoile_masse</p>

<p>def mort_stellaire(masse_noyau):<br>
    if masse_noyau &lt; 3:<br>
        return "Naine Blanche"<br>
    elif masse_noyau &lt; 7:<br>
        return "Étoile à Neutrons"<br>
    else:<br>
        return "Trou Noir"</p>

<h1>
  
  
  --- Simulation du cycle complet ---
</h1>

<p>etoile_creee = False<br>
etoile_masse = 0</p>

<h1>
  
  
  Phase 1 : La naissance
</h1>

<p>for temps in range(1, 101):<br>
    nouvelle_grille = np.copy(cosmic_void)<br>
    for p in particules:<br>
        nouvelle_grille[p['pos'][0], p['pos'][1]] -= p['force']</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>for y in range(grid_size):
    for x in range(grid_size):
        if nouvelle_grille[y, x] &lt;= (valeur_energie_noire_de_base - force_de_lumiere):
            etoile_masse = valeur_energie_noire_de_base - nouvelle_grille[y, x]
            print(f"\nPOUF CHOCAPIC ! Une étoile est née à ({y}, {x}) avec une masse de {etoile_masse:.2f} !")
            etoile_creee = True
            break
    if etoile_creee: break
if etoile_creee: break
</code></pre>

</div>
<h1>
  
  
  Phase 2 : La mort
</h1>

<p>if etoile_creee:<br>
    while etoile_masse &gt; masse_critique:<br>
        etoile_masse = vie_etoile(etoile_masse)<br>
        time.sleep(0.1)</p>
<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>print(f"\nL'étoile a atteint la masse critique de {masse_critique} !")
destin_final = mort_stellaire(etoile_masse)
print(f"L'étoile a terminé son cycle et est devenue : {destin_final}")
</code></pre>

</div>

<p>print("\nLe cycle de l'étoile est terminé.")<br>
Le cosmos est créé, avec un potentiel d'énergie noire élevé.</p>

<p>Le cycle de l'étoile est terminé.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/..." class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/..." alt="Uploading image" width="800" height="400"></a></p>

