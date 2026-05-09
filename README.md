# 📱 LAB 9 — Analyse de Surface d'Attaque Android avec Drozer

---

## 🧾 Informations générales

| Champ | Détail |
|-------|--------|
| Application | InsecureBankv2 |
| Package | `com.android.insecurebankv2` |
| Outil utilisé | Drozer |
| Type d'audit | Audit défensif — analyse de surface d'attaque |

---

## 🎯 Objectif

Identifier les composants Android exposés (Activities, Broadcast Receivers, Content Providers) et évaluer les risques de sécurité associés à l'aide de l'outil **Drozer**.

---

## ⚙️ Étape 1 — Connexion à Drozer

<img width="948" height="460" alt="etap2" src="https://github.com/user-attachments/assets/5d0e1556-affe-4c22-8321-fabe4e991d8a" />


---

## 📦 Étape 2 — Inventaire des applications installées

```
dz> run app.package.list
```
<img width="948" height="460" alt="etap2" src="https://github.com/user-attachments/assets/408a8621-5e45-4f46-8ec8-755b0c08e44b" />


---

## 🔍 Étape 3 — Informations détaillées sur l'application

```
dz> run app.package.info -a com.android.insecurebankv2
```

<img width="1108" height="556" alt="etap3" src="https://github.com/user-attachments/assets/29d8abba-8740-487b-9109-b7441240293c" />


---

## 📱 Étape 4 — Analyse des Activities

```
dz> run app.activity.info -a com.android.insecurebankv2
```

<img width="1108" height="556" alt="etap4" src="https://github.com/user-attachments/assets/19828259-de6a-4003-aa2d-f69ad77d9874" />


---

## 📡 Étape 5 — Broadcast Receivers exposés

```
dz> run app.broadcast.info -a com.android.insecurebankv2
```

<img width="1070" height="481" alt="etap5" src="https://github.com/user-attachments/assets/73cbed69-59ae-409f-ba81-b769107ccfbb" />


---

## 🗄️ Étape 6 — Content Providers

```
dz> run app.provider.info -a com.android.insecurebankv2
```

<img width="1056" height="282" alt="6" src="https://github.com/user-attachments/assets/acd2c111-aa47-4260-9fa4-122e1127e477" />


---

## 📜 Étape 7 — Lecture du Manifest

```
dz> run app.package.manifest com.android.insecurebankv2
```

<img width="1016" height="740" alt="7" src="https://github.com/user-attachments/assets/d56973de-25ac-4025-ae9d-896aae4c4bdb" />


---

## 🔎 Étape 8 — Scan des URI accessibles

```
dz> run scanner.provider.finduris -a com.android.insecurebankv2
```

<img width="1286" height="433" alt="8" src="https://github.com/user-attachments/assets/3bc73b5b-8221-40c2-a246-b6b70802d53f" />


---

## 🚨 Vulnérabilités identifiées

### 🔴 1. Activities exportées sans protection

| Activity | Exportée | Permission |
|----------|----------|------------|
| PostLogin | ✅ Oui | ❌ Aucune |
| DoTransfer | ✅ Oui | ❌ Aucune |
| ViewStatement | ✅ Oui | ❌ Aucune |
| ChangePassword | ✅ Oui | ❌ Aucune |

> ⚠️ **Risque :** contournement de l'authentification — un attaquant peut lancer ces écrans directement via un intent

---

### 🔴 2. Broadcast Receiver exposé

- `MyBroadCastReceiver` — aucune permission requise

> ⚠️ **Risque :** injection d'intents malveillants depuis une application tierce

---

### 🔴 3. Content Provider accessible sans permission

```
content://com.android.insecurebankv2.TrackUserContentProvider/trackerusers
content://com.android.insecurebankv2.TrackUserContentProvider/trackerusers/
```

> ⚠️ **Risque :** lecture des données utilisateur sans authentification

---

### 🔴 4. Permissions sensibles déclarées

| Permission | Impact |
|------------|--------|
| `READ_CONTACTS` | Accès au répertoire |
| `SEND_SMS` | Envoi de SMS à l'insu de l'utilisateur |
| `READ_CALL_LOG` | Accès à l'historique d'appels |
| `ACCESS_LOCATION` | Suivi géographique |

> ⚠️ **Risque :** abus de données personnelles

---

### 🔴 5. Mode debug activé

```xml
android:debuggable="true"
```

> ⚠️ **Risque :** facilite le reverse engineering et l'attachement d'un débogueur externe

---

## 📊 Tableau récapitulatif des composants exposés

| Type | Composant | Exporté | Protection |
|------|-----------|---------|------------|
| Activity | PostLogin | ✅ Oui | ❌ Aucune |
| Activity | DoTransfer | ✅ Oui | ❌ Aucune |
| Activity | ViewStatement | ✅ Oui | ❌ Aucune |
| Activity | ChangePassword | ✅ Oui | ❌ Aucune |
| Receiver | MyBroadCastReceiver | ✅ Oui | ❌ Aucune |
| Provider | TrackUserContentProvider | ✅ Oui | ❌ Aucune |

---

## 🛡️ Recommandations

**1. Désactiver l'export des composants non publics :**
```xml
android:exported="false"
```

**2. Protéger les composants exposés avec des permissions custom :**
```xml
android:permission="com.android.insecurebankv2.CUSTOM_PERMISSION"
```

**3. Restreindre l'accès au Content Provider :**
```xml
android:readPermission="..."
android:writePermission="..."
```

**4. Désactiver le mode debug en production :**
```xml
android:debuggable="false"
```

---

## 🧠 Conclusion

L'analyse de **InsecureBankv2** avec Drozer met en évidence une surface d'attaque très large, caractérisée par :

- Des **composants Android exportés** sans aucun contrôle d'accès
- Une **absence totale de permissions** sur les Activities et Providers critiques
- Une **mauvaise gestion des permissions** système (SMS, contacts, localisation)
- Un **mode debug actif** facilitant le reverse engineering

Ces failles permettraient à une application malveillante installée sur le même appareil d'accéder à des fonctionnalités sensibles et d'exfiltrer des données utilisateur sans aucune interaction de la victime.<img width="948" height="460" alt="etap2" src="https://github.com/user-attachments/assets/34f3b615-b59a-4025-b7cd-de736311d731" />
