# TwinCAT Event Logger – Traçabilité et Export CSV

Ce dépôt contient un exemple permet de créer une traçabilité des actions réalisées dans une IHM TwinCAT HMI via l’**Event Logger TwinCAT**. Le code PLC envoie automatiquement des événements périodiques, ainsi que des événements déclenchés par l’IHM. Un mécanisme d’export au format **CSV** est également intégré.
Ce dépot n'utilise pas les fonctionnalité de l'audit Trail de TE2000
---

## 🎯 Objectifs du projet

* Générer des événements TwinCAT à intervalle régulier (toutes les 10s).
* Générer des événements déclenchés par des actions IHM.
* Exporter l’historique des événements en CSV.
* Ouvrir automatiquement l’explorateur Windows après export.

---

## 📁 Structure du code PLC

Le programme PLC comporte trois parties principales :

### 1. **Initialisation**

* Réinitialisation du compteur d’événements.
* Instanciation du message via `fbMsg.CreateEx()`.

```iecst
IF bInit THEN
    wCount := 0;
    bInit := FALSE;
    fbMsg.CreateEx(TC_EVENTS.EventClass.Logger, 0);
END_IF
```

---

### 2. **Génération d’un log automatique toutes les 10 secondes**

* Utilisation d’un temporisateur `TON`.
* Création du message enrichi par `{0}`.

```iecst
TonDef(IN := TRUE, PT := T#10S, Q => , ET => );
IF TonDef.Q THEN
    wCount := wCount + 1;
    TonDef(IN := FALSE);
    sLog := CONCAT('Event num: ', WORD_TO_STRING(wCount));
    fbMsg.ipArguments.Clear().AddString(sLog);
    fbMsg.Send(0);
END_IF
```

---

### 3. **Génération d’un événement depuis l’IHM**

* Une chaîne `sLog_IHM_Action` est envoyée en événement si elle est enrichie via l’IHM.

```iecst
IF sLog_IHM_Action <> '' THEN
    fbMsg.ipArguments.Clear().AddString(sLog_IHM_Action);
    fbMsg.Send(0);
    sLog_IHM_Action := '';
END_IF
```

---

### 4. **Export CSV des événements**

* Export via `fbTcEventLogger.ExportLoggedEvents()`.
* Ouverture automatique de l’explorateur Windows.

```iecst
IF bExport THEN
    bExportResult := fbTcEventLogger.ExportLoggedEvents(
        sFileName := sExportPathFile,
        ipExportSettings := fbTcEventCsvExportSettings,
        bError => bExportError,
        hrErrorCode => hrExportErrorCode
    );

    IF bExportResult THEN
        bExport := FALSE;
        sCommand := CONCAT('C:\Windows\explorer.exe ', sExportPath);
        fbNT_StartProcess(NETID := '', PATHSTR := sCommand, START := FALSE);
        fbNT_StartProcess(START := TRUE);
    END_IF
END_IF
```

---

## 📌 Points importants

* `{0}` dans la définition de l’Event TwinCAT permet d'injecter dynamiquement un argument enrichi.
* `ExportLoggedEvents` nécessite un chemin complet vers le fichier CSV.
* Deux appels successifs à `fbNT_StartProcess` sont nécessaires pour générer un front montant sur `START`.

---

## 🛠 Pré-requis

* TwinCAT 3.1.x
* Bibliothèques nécessaires :

  * **Tc2_Utilities** (pour `NT_StartProcess`)
  * **TcEventLogger**
  * **TcSystem**

---

## 🚀 Intégration dans un projet Git

Ce fichier Markdown peut servir :

* de documentation pour le dépôt,
* de guide rapide pour les développeurs TwinCAT,
* d'explication pour la logique d’export et de traçabilité.

N’hésitez pas à compléter avec vos captures TwinCAT, configurations Event Logger ou exemples d'IHM !

---

## 📄 Licence

Projet libre d’utilisation interne – à adapter selon votre entreprise.
