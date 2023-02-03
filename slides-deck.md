---
author: François Freitag
footer: PyConFr 2023 ![height:40px](https://inclusion.beta.gouv.fr/images/logo-emplois.svg)
paginate: true
theme: gaia
title: REX analyse antivirus des fichiers de la plateforme des emplois de l’inclusion
---
<style>
footer {
    display: flex;
    justify-content: space-around;
}
</style>

# REX analyse antivirus
#### des fichiers de la plateforme des emplois de l’inclusion

---

# GIP de l’inclusion

<style scoped>
p > img {
    background-color: #222;
    padding: 7px;
    margin-right: 15px;
    border-radius: 10px;
}
</style>


> Faciliter la vie des personnes en insertion et de celles et ceux qui les accompagnent à travers de nouveaux services publics.

——————————————————————————————————————

#### François Freitag

[mail@franek.fr](mailto:mail@franek.fr)

![height:40px](https://www.python.org/static/img/python-logo.png) ![height:40px](https://infooptima.net/wp-content/uploads/2016/02/Django-logo.svg_.png) ![height:40px](https://www.sphinx-doc.org/en/master/_static/sphinxheader.png)



---
# Contexte

* 500 000+ fichiers PDF 📈
* Envoyés directement sur Cellar (S3)
* Pas de latence perceptible à l’envoi (exigence métier)
* Analyse quotidienne
* Analyse complète mensuelle (évolution signature virus)

---
# L’antivirus
![height:200px](https://www.clamav.net/assets/clamav-trademark.png)

- Gratuit et open-source
- CleverCloud `CC_CLAMAV=1`

---
# Performances

Échantillon de 10 000 fichiers aléatoires
Temps d’analyse par fichier :
- 1 seconde en moyenne
- 20 secondes au pire

⇒ Pas d’analyse directement lors de la requête HTTP

---
# Intégration à la plateforme

Informations utiles :

- horodatage de l’analyse
- résultat de l’analyse
- confirmation manuelle
- commentaire

---
# Intégration à la plateforme

L’`admin` Django :
<!-- Screenshot admin ? -->

---
# Analyse des fichiers

`cron`

* Identifie les fichiers à analyser
* Les télécharge : `ThreadPoolExecutor` :heart:
* Analyse avec ClamAV : `subprocess.run()` :heart:
* Enregistre le résultat : *ORM* Django :heart:

---
# Solution *a minima*

- Analyse quotidienne des nouveaux fichiers
    * Parcours des objets Cellar (S3) : environ 5 minutes
    * Analyse : environ 5 minutes
- Analyse mensuelle de tous les fichiers
    * Parcours des objets Cellar (S3) : environ 5 minutes
    * Analyse : environ 17 280 minutes *(ou 3 jours)*
* Oops : interruptions lors du déploiement (`SIGTERM` après 90 s)

---
#  Pas très satisfaisant… 🤔

Gestion des interruptions liées au `SIGTERM` du déploiement
- Pas de déploiement pendant 3 jours ? 🤨
* Mécanisme de reprise
    * Gestion de `SIGTERM` ?
        * Gestion de signal ⚠, quid crash sans `SIGTERM` ?
    * Acquittement ?
        * Sous quel délai ?

---
# Analyse des fichiers (mieux)
`cron`
- Identifie **mieux** les fichiers à analyser
- Les télécharge : `ThreadPoolExecutor` :heart:
- Analyse avec ClamAV : `subprocess.run()` :heart:
- Enregistre le résultat : *ORM* Django :heart:

---
# Analyse des fichiers (mieux)

**Une fois par jour**

`cron` synchro *Cellar* (S3) → base de données

**Fréquemment**

Sélection d’un lot de fichiers récents, ou horodatage >= un mois

```python
select_for_update(skip_locked=True, no_key=True)
```

---
# Analyse des fichiers (mieux)

Que nous apporte la base de données ?
* Nettoyage automatique du verrou en cas d’échec
* Gestion de la concurrence
* Le meilleur ?
    * Elle est déjà en place.

---
# Je veux voir le code !

C’est open-source.
https://github.com/betagouv/itou/blob/master/itou/antivirus/management/commands/scan_s3_files.py

---

# Aller plus loin ?

- Stocker les fichiers via Django
- Gestion des fichiers infectés : *API* VirusTotal
- Zone de quarantaine Cellar
- Parallélisation des analyses

---
<style scoped>
h1 {
    font-size: 280px;
    text-align: center;
}
h2 {
    text-align: center;
}
</style>
# Merci
## Avez-vous des questions ?
