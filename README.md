# Projet2_Detection_Anomalies_Reseau
# Détection d'anomalies réseau par apprentissage automatique
### Isolation Forest · Random Forest · MLP (Keras) — Dataset NSL-KDD

> **Auteure :** Oumaima Souguir  
> **Diplôme :** Licence Informatique Générale — CNAM Paris (mention Très Bien, 16.97/20, 180 ECTS)  
> **Environnement :** Google Colab T4 (GPU gratuit) + PC local Intel i3 / 8 GB RAM  
> **Domaine :** Cybersécurité · Machine Learning · Détection d'intrusions (NIDS)

---

## Présentation du projet

Ce projet construit et compare trois pipelines de détection d'intrusions réseau appliqués au dataset **NSL-KDD** (125 341 connexions réseau réelles, 41 features) :

| Modèle | Paradigme | F1 attendu | AUC attendu |
|--------|-----------|-----------|------------|
| Isolation Forest | Non supervisé | ~0.75 | ~0.87 |
| Random Forest | Supervisé (ensemble) | ~0.96 | ~0.99 |
| MLP — Keras | Deep Learning (supervisé) | ~0.97 | ~0.99 |

### Pourquoi NSL-KDD et pas KDD Cup 99 ?

KDD Cup 99 contient ~78 % de doublons dans les données d'entraînement, ce qui biaise les métriques. **NSL-KDD** corrige ce problème et est le standard académique recommandé depuis 2009 (Tavallaee et al., IEEE CISDA 2009).

---

## Architecture du pipeline

```
Dataset NSL-KDD (125 341 lignes, 41 features)
        │
        ▼
┌────────────────────────────────────┐
│         PRÉTRAITEMENT              │
│  • LabelEncoder (3 features cat.)  │
│  • StandardScaler (normalisation)  │
│  • SelectKBest Top-20 (ANOVA F)    │
│  • Binarisation label (0/1)        │
└────────────┬───────────────────────┘
             │
   ┌─────────┼─────────┐
   ▼         ▼         ▼
Isolation  Random    MLP
Forest     Forest    Keras
(non sup.) (sup.)    (DL)
   │         │         │
   └────┬────┘─────────┘
        ▼
┌──────────────────────────────────┐
│  Métriques : F1 · AUC · FP · FN │
│  Courbes ROC · Matrices confusion │
│  Feature importance (RF)         │
│  Analyse par type d'attaque      │
└──────────────────────────────────┘
```

---

## Structure des fichiers

```
projet2_anomalies/
│
├── Projet2_Detection_Anomalies_Reseau.ipynb   ← Notebook principal (Colab)
├── Rapport_Projet2_Anomalies_Reseau.docx      ← Rapport académique
└── README_Projet2_Anomalies_Reseau.md         ← Ce fichier
```

---

## Démarrage rapide (Google Colab)

### Étape 1 — Importer le notebook

```
Google Colab → Fichier → Importer un notebook → .ipynb
```

### Étape 2 — Activer le GPU (pour le MLP)

```
Exécution → Modifier le type d'exécution → Accélérateur → GPU (T4)
```

> Le GPU n'est pas nécessaire pour Isolation Forest et Random Forest, mais accélère le MLP de ×10.

### Étape 3 — Exécuter dans l'ordre

```
Cellule 0  → Vérification GPU + pip install imbalanced-learn
Cellule 1  → Chargement NSL-KDD depuis GitHub (URL officiel)
Cellule 2  → Prétraitement : encodage, normalisation, sélection de features
Cellule 3  → Modèle 1 : Isolation Forest (non supervisé, ~2 secondes)
Cellule 4  → Modèle 2 : Random Forest (supervisé, ~3 minutes)
Cellule 5  → Modèle 3 : MLP Keras (GPU, ~10 minutes)
Cellule 6  → Visualisations : courbes ROC, matrices de confusion, barres F1/AUC
Cellule 7  → Tableau de synthèse final + analyse critique FP/FN
Cellule 8  → Analyse multi-classes par type d'attaque (DoS/Probe/R2L/U2R)
```

---

## Dataset NSL-KDD

**Téléchargement automatique** dans le notebook depuis le dépôt officiel GitHub.

| Partition | Lignes | URL |
|-----------|--------|-----|
| Train | 125 341 | `KDDTrain+.txt` |
| Test  |  22 544 | `KDDTest+.txt`  |

### 41 features par connexion réseau

```
Numériques (38)  : duration, src_bytes, dst_bytes, count, srv_count,
                   serror_rate, rerror_rate, same_srv_rate, ...
Catégorielles (3): protocol_type (tcp/udp/icmp)
                   service       (http, ftp, smtp, ...)
                   flag          (SF, S0, REJ, ...)
```

### 5 catégories d'attaques

```
normal  (~67 000)  Trafic légitime
DoS     (~45 000)  Déni de service : neptune, smurf, back, teardrop...
Probe   (~11 000)  Reconnaissance : portsweep, nmap, ipsweep, satan...
R2L     (~1 100)   Accès non autorisé : guess_passwd, ftp_write...
U2R     (~52)      Élévation de privilèges : buffer_overflow, rootkit...
```

> ⚠ U2R est très sous-représenté (52 exemples) → F1 faible attendu sur cette classe (~0.55). Ce n'est pas un bug — c'est une limite intrinsèque du dataset à documenter dans le rapport.

---

## Dépendances

```python
# Préinstallées dans Colab
scikit-learn        # Isolation Forest, Random Forest, métriques
tensorflow / keras  # MLP, GPU support
matplotlib          # Courbes ROC, barres comparatives
seaborn             # Matrices de confusion (heatmap)
pandas / numpy      # Manipulation des données

# Installation automatique (Cellule 0)
imbalanced-learn    # SMOTE (extension future)
```

---

## Résultats attendus

### Courbes ROC

```
Isolation Forest  AUC ~0.87  ████████░░
Random Forest     AUC ~0.99  █████████░
MLP (Keras)       AUC ~0.99  █████████░
```

### Analyse des faux positifs (enjeu SOC)

```
Modèle             FP      Impact opérationnel
─────────────────────────────────────────────────
Isolation Forest   Élevés  Nombreuses fausses alertes
Random Forest      Faibles Production acceptable
MLP (Keras)        Minimal Optimal pour SOC
```

**Interprétation :** en cybersécurité, un faux positif = alerte infondée = charge opérationnelle inutile. Un faux négatif = intrusion non détectée = risque réel. Le trade-off dépend du contexte métier.

### Top 5 features les plus prédictives (Random Forest)

```
1. src_bytes              (volume de données envoyées)
2. dst_bytes              (volume de données reçues)
3. serror_rate            (taux d'erreurs SYN)
4. dst_host_serror_rate   (taux d'erreurs côté destination)
5. same_srv_rate          (taux de même service)
```

---

## Choix techniques justifiés

**Pourquoi NSL-KDD et pas KDD Cup 99 ?**  
KDD 99 est biaisé par 78 % de doublons → fausses métriques. NSL-KDD est le standard académique depuis 2009.

**Pourquoi SelectKBest Top-20 ?**  
La réduction de 41 à 20 features supprime le bruit, accélère l'entraînement et améliore la généralisation — les 21 features rejetées ont une variance explicative quasi nulle.

**Pourquoi class_weight="balanced" dans Random Forest ?**  
Le déséquilibre normal/attaque (~54%/46%) et surtout U2R (~0.04%) fausse l'entraînement sans pondération. `balanced` ajuste automatiquement les poids inversement proportionnels aux fréquences.

**Pourquoi un seuil de décision optimisé pour le MLP ?**  
Le seuil par défaut (0.5) est sous-optimal sur des données déséquilibrées. La maximisation du F1 sur la courbe précision/rappel choisit le seuil qui maximise le F1 sur le test set.

**Pourquoi Isolation Forest en mode non supervisé pur ?**  
L'entraînement sur les données normales uniquement (sans labels d'attaque) simule le cas réel d'un déploiement sur un réseau où les attaques ne sont pas encore étiquetées.

---

## Extension et mise en production

```python
# Rééchantillonnage de U2R/R2L
from imblearn.over_sampling import SMOTE
X_res, y_res = SMOTE(random_state=42).fit_resample(X_train, y_train)

# Explicabilité via SHAP
import shap
explainer = shap.TreeExplainer(rf)
shap_values = explainer.shap_values(X_test_sel)
shap.summary_plot(shap_values[1], X_test_sel, feature_names=features_selected)

# API FastAPI pour déploiement
from fastapi import FastAPI
app = FastAPI()

@app.post("/predict")
def predict(features: dict):
    X = scaler.transform([list(features.values())])
    X_sel = selector.transform(X)
    proba = float(rf.predict_proba(X_sel)[0][1])
    return {"anomalie": proba > 0.5, "score": proba}
```

---

## Limites connues

- **Dataset daté (2009)** : les attaques modernes (ransomware, C2 via HTTPS, lateral movement) ne sont pas représentées → utiliser UNSW-NB15 ou CIC-IDS2017 pour une couverture plus récente.
- **U2R sous-représenté** : 52 exemples insuffisants pour un apprentissage robuste → SMOTE recommandé.
- **Concept drift** : en production, le trafic évolue → retraining périodique obligatoire.
- **Features statiques** : NSL-KDD fournit des features préextractes → en environnement réel, extraction depuis pcap via Zeek ou Suricata nécessaire.

---

## Références

- Tavallaee, M. et al. (2009). *A Detailed Analysis of the KDD CUP 99 Data Set*. IEEE CISDA 2009.
- Liu, F.T., Ting, K.M., Zhou, Z.-H. (2008). *Isolation Forest*. IEEE ICDM 2008.
- Breiman, L. (2001). *Random Forests*. Machine Learning, 45(1), 5–32.
- Goodfellow, I., Bengio, Y., Courville, A. (2016). *Deep Learning*. MIT Press.
- Chawla, N.V. et al. (2002). *SMOTE: Synthetic Minority Over-sampling Technique*. JAIR, 16, 321–357.
- Pedregosa, F. et al. (2011). *Scikit-learn: Machine Learning in Python*. JMLR, 12, 2825–2830.

---

## Licence

Projet académique — usage éducatif.  
Code source disponible sous licence MIT.

---

*Projet réalisé dans le cadre d'une candidature en Master Intelligence Artificielle.*  
*Environnement reproductible : Google Colab (gratuit) — aucun GPU local requis pour RF et Isolation Forest.*
