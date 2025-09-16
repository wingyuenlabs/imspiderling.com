---
Title: 🚢 Titanic App Streamlit "Machine Learning Scikit Learn-Random Forest"
Description: 
Author: Sofiane Chehboune
Date: 2025-09-16T22:07:44.000Z
Robots: noindex,nofollow
Template: index
---
<p><strong><em>🚢 J'ai créé une application pour visualiser les données du Titanic et prédire vos chances de survie avec Streamlit et Scikit-Learn</em></strong></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3nn7naqw51lkmhbojrc6.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3nn7naqw51lkmhbojrc6.png" alt=" " width="800" height="383"></a></p>

<p>Bonjour la communauté dev.to !</p>

<p><em>Je suis ravi de partager mon dernier projet</em> : une application web interactive construite avec Streamlit qui explore le célèbre jeu de données du Titanic. Non seulement elle permet de visualiser les statistiques clés des passagers, mais elle intègre également un modèle de Machine Learning (RandomForest) pour prédire les chances de survie en fonction des caractéristiques que vous choisissez.</p>

<p>C'est un excellent projet pour quiconque souhaite voir comment transformer un script d'analyse de données en une application web fonctionnelle et interactive sans toucher du HTML, CSS ou JavaScript.</p>

<p><strong>✨ Démo et code source</strong><br>
Avant de plonger dans les détails, n'hésitez pas à l'essayer par vous-même !</p>

<p><strong>🚀 Essayez l'application en ligne</strong> : <a href="https://app-app-titanic-data-bdwwycbgdejsmtuv4ntkss.streamlit.app/" rel="noopener noreferrer">Titanic Survival Predictor</a><br>
<strong>👨‍💻 Explorez le code source sur GitHub</strong> : <a href="https://github.com/SofianeChehboune/Streamlit-App-Titanic-Data" rel="noopener noreferrer">Repo GitHub</a><br>
<em>(N'hésitez pas à laisser une étoile ⭐ sur le repo si vous aimez le projet !)<br>
(Conseil : Créez un petit GIF de votre application en action et hébergez-le sur un site comme Imgur pour l'insérer ici. C'est très efficace !)</em></p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fe21sc9dte0tbfpe2ntrm.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fe21sc9dte0tbfpe2ntrm.png" alt=" " width="800" height="383"></a></p>

<p><strong>🛠️ La Stack Technique</strong><br>
La beauté de ce projet réside dans sa simplicité et la puissance des outils utilisés :</p>

<p><strong><em>Streamlit</em></strong> : Pour construire l'interface utilisateur web de manière simple et rapide, entièrement en Python.<br>
Pandas : Pour la manipulation et le nettoyage des données du Titanic.<br>
<em><strong>Scikit-learn</strong></em> : Pour l'entraînement du modèle <code>RandomForestClassifier</code> un excellent choix pour ce type de problème de classification.<br>
<strong><em>Plotly &amp; Seaborn</em></strong> : Pour générer les graphiques interactifs et les visualisations de données.</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Faog6o8zu7gpp0859wz6n.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Faog6o8zu7gpp0859wz6n.png" alt=" " width="800" height="379"></a></p>

<p><strong>🔬 Comment ça marche ?</strong><br>
Le projet se décompose en trois étapes principales :</p>

<p><strong><em>1. Exploration et Visualisation des Données (EDA)</em></strong><br>
La première partie de l'application est dédiée à l'analyse exploratoire. J'ai utilisé Plotly et Seaborn pour créer des graphiques qui répondent à des questions comme:</p>

<ul>
<li>Le sexe avait-il un impact sur la survie ?</li>
<li>La classe du billet at-elle joué un rôle ?* Comment l'âge était-il distribué parmi les survivants et les non-survivants ?
Voici un petit extrait de code afficher pour le diagramme circulaire de la survie par sexe :
</li>
</ul>
<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>import plotly.express as px

# Création d'un DataFrame pour le graphique
gender_survival = df.groupby('Sex')['Survived'].value_counts(normalize=True).unstack().fillna(0)
gender_survival = (gender_survival * 100).round(2)

# Affichage avec Plotly
fig = px.pie(    names=['Non-survivants', 'Survivants'],
    values=gender_survival.loc[selected_gender],
    title=f"Pourcentage de survie pour le sexe : {selected_gender}"
)st.plotly_chart(fig)
</code></pre>

</div>


<p><strong><em>2. Entraînement du Modèle de Machine Learning</em></strong><br>
J'ai choisi un</p>

<p><code>RandomForestClassifier</code></p>

<p>pour sa robustesse et sa bonne performance "out-of-the-box". Le processus est classique :</p>

<p><strong><em>Nettoyage des données</em></strong> : Remplacer les valeurs manquantes (par exemple, l'âge médian pour les âges manquants).<br>
<strong><em>Feature Engineering</em></strong> : Conversion des variables catégorielles (comme le sexe ou le port d'embarquement) en valeurs numériques.<br>
<strong><em>Entraînement</em></strong> : Séparation des données en un ensemble d'entraînement et de test, puis entraînement du modèle sur les données d'entraînement.<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import train_test_split

# Définition des features (X) et de la cible (y)
features = ['Pclass', 'Sex', 'Age', 'SibSp', 'Parch', 'Fare']
X = df[features]
y = df['Survived']

# Encodage des variables catégorielles
X = pd.get_dummies(X, columns=['Sex'], drop_first=True)

# Division des données et entraînement
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)
model = RandomForestClassifier(n_estimators=100, random_state=42)
model.fit(X_train, y_train)```



#### 3. Création de l'Interface de Prédiction

C'est là que la magie de Streamlit opère. J'ai utilisé la barre latérale (`st.sidebar`) pour créer un formulaire où l'utilisateur peut entrer ses propres données (classe, sexe, âge, etc.).



```python
# Sidebar pour les inputs de l'utilisateur
st.sidebar.header("Faites votre propre prédiction :")
pclass = st.sidebar.selectbox('Classe du billet', [1, 2, 3])
sex = st.sidebar.selectbox('Sexe', ['male', 'female'])
age = st.sidebar.slider('Âge', 0, 100, 25)
# ... autres inputs

# Prédiction en temps réel
if st.sidebar.button("Prédire ma survie"):
    # Créer un DataFrame avec les inputs de l'utilisateur
    user_input = pd.DataFrame(...) 

    # Faire la prédiction
    prediction = model.predict(user_input)

    # Afficher le résultat
    if prediction[0] == 1:
        st.sidebar.success("Félicitations ! Vous auriez probablement survécu !")
    else:
        st.sidebar.error("Désolé... Vous n'auriez probablement pas survécu.")
</code></pre>

</div>



<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4pt67djabccnn6snba5m.png" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4pt67djabccnn6snba5m.png" alt=" " width="800" height="378"></a></p>

<p><strong>🏁 Conclusion</strong><br>
Ce projet a été une excellente occasion de combiner l'analyse de données, le machine learning et le développement web d'une manière très accessible. Streamlit continue de m'impressionner par sa capacité à créer des applications de données interactives avec une facilité déconcertante.</p>

<p>J'adorerais avoir vos retours ! Qu'en pensez-vous ? Y at-il des visualisations ou des fonctionnalités que vous aimeriez voir ajoutées ? Merci de m'avoir lu !</p>

<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fh4hix47tb4q0oyx9tofs.jpg" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fh4hix47tb4q0oyx9tofs.jpg" alt=" " width="800" height="420"></a></p>

<p>❤️</p>

