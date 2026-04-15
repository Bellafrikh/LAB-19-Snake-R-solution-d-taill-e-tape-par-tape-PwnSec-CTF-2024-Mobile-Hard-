# LAB-19-Snake-R-solution-d-taill-e-tape-par-tape-PwnSec-CTF-2024-Mobile-Hard-

# Rapport de Laboratoire 19 : Analyse et Exploitation Android
**Sujet :** Reverse Engineering et Exploitation de vulnérabilité SnakeYAML
**Auteur :** Zaynab Bellafrikh
**Date :** 15 Avril 2026

---

## 1. Objectif du Lab
L'objectif est de réaliser un audit de sécurité sur l'application `snake.apk` pour extraire un flag caché. Le défi combine deux aspects :
1.  **Bypass de sécurité :** Contourner les mécanismes anti-root/anti-debug.
2.  **Exploitation :** Utiliser une vulnérabilité de désérialisation YAML pour instancier une classe malveillante.

## 2. Analyse Statique
L'ouverture de l'APK dans **JADX-GUI** a permis d'identifier la structure suivante :
* **MainActivity :** Contient une méthode `isDeviceRooted()` qui bloque l'application.
* **Classe BigBoss :** Une classe non appelée par le flux normal du programme, dont le constructeur imprime le flag dans le Logcat.
* **Vulnérabilité :** L'application utilise `SnakeYAML` pour charger des fichiers sans validation, permettant une instanciation de classe arbitraire.

<img width="808" height="102" alt="image" src="https://github.com/user-attachments/assets/b2e722be-83b4-48d8-8813-4b03412dd24e" />

<img width="1361" height="893" alt="image" src="https://github.com/user-attachments/assets/820a1886-b94d-40c9-bdf4-df69c6be5f20" />

<img width="1348" height="738" alt="image" src="https://github.com/user-attachments/assets/16c08dfb-553c-41df-b65a-9b3c7c5262e8" />

<img width="1292" height="262" alt="image" src="https://github.com/user-attachments/assets/ce5c3700-be13-40ac-b484-c78687bb7077" />

<img width="938" height="652" alt="image" src="https://github.com/user-attachments/assets/6f6dc0af-88d1-4fbe-a7b1-97251a43857e" />

<img width="1186" height="532" alt="image" src="https://github.com/user-attachments/assets/84ec4066-6ce7-4146-aace-ed0aedf08092" />

## 3. Étapes de Réalisation

### A. Modification du Bytecode (Patching Smali)
Pour supprimer la barrière du Root, l'APK a été décompilé avec `apktool` :
1.  Modification de `MainActivity.smali`.
2.  Neutralisation du saut conditionnel après l'appel à `isDeviceRooted`.
3.  Recompilation et signature avec `uber-apk-signer`.

<img width="1185" height="361" alt="WhatsApp Image 2026-04-15 at 19 56 56" src="https://github.com/user-attachments/assets/aa63852d-67bb-4417-925c-b879bd8a914a" />

<img width="1600" height="548" alt="image" src="https://github.com/user-attachments/assets/5821ae4a-6703-4262-8196-8e114a0b09ea" />

<img width="1243" height="208" alt="image" src="https://github.com/user-attachments/assets/a1017491-1c22-456d-8e4e-b32ca20ae9cc" />

<img width="1343" height="255" alt="image" src="https://github.com/user-attachments/assets/b1608166-a786-4954-92b0-dcecb3074956" />

<img width="957" height="367" alt="image" src="https://github.com/user-attachments/assets/1ebeb454-0740-4d89-9196-33bf3709bd99" />

<img width="1387" height="596" alt="image" src="https://github.com/user-attachments/assets/1e58a430-5efc-4e71-a4a5-9809d94fa230" />

<img width="896" height="131" alt="image" src="https://github.com/user-attachments/assets/b9570b21-d03b-4872-b102-6111c3c214f9" />

### B. Création du Payload
Un fichier YAML a été conçu pour forcer l'application à instancier la classe `BigBoss`.
**Fichier :** `Skull_Face.yml`
```yaml
!!com.pwnsec.snake.BigBoss ["Snaaaaaaaaaaaaaake"]
```

<img width="1461" height="92" alt="image" src="https://github.com/user-attachments/assets/b44d9996-a9fc-41df-9b6f-688b31267d20" />

### C. Injection et Exécution
L'exploitation finale a été réalisée via **ADB** (Android Debug Bridge) en suivant une séquence précise de commandes pour préparer l'environnement et déclencher la vulnérabilité.

```powershell
# 1. Préparation du répertoire de travail sur l'émulateur
adb shell mkdir -p /sdcard/snake

# 2. Transfert du fichier YAML malveillant (Payload)
adb push Skull_Face.yml /sdcard/snake/Skull_Face.yml

# 3. Attribution manuelle des permissions de stockage pour l'application
adb shell pm grant com.pwnsec.snake android.permission.READ_EXTERNAL_STORAGE

# 4. Désactivation de SELinux pour éviter le blocage du processus natif
adb shell setenforce 0

# 5. Déclenchement de l'exploitation via un Intent avec paramètres (Extras)
adb shell am start -n com.pwnsec.snake/.MainActivity -e SNAKE BigBoss
```
<img width="875" height="27" alt="image" src="https://github.com/user-attachments/assets/68394a18-7070-47a6-9090-2a8f7d80f510" />

<img width="1451" height="122" alt="image" src="https://github.com/user-attachments/assets/e4672f93-cd20-4054-87ed-15f72b9e4619" />

<img width="1462" height="51" alt="image" src="https://github.com/user-attachments/assets/e0bfb1dc-037f-4b0d-9ac3-5a33dccbb64d" />

## 4. Capture du Flag

L'extraction du flag ne s'effectue pas via l'interface graphique de l'application, mais à travers les flux de logs du système Android. En utilisant l'outil `logcat` et en appliquant un filtre sur le tag spécifique `BigBoss`, nous avons pu intercepter la chaîne de caractères générée par la bibliothèque native lors de l'instanciation de la classe.

**Commande de surveillance en temps réel :**

```powershell
adb logcat -s BigBoss
```


## 4. Capture du Flag

L'extraction du flag ne s'effectue pas via l'interface graphique de l'application, mais à travers les flux de logs du système Android. En utilisant l'outil `logcat` et en appliquant un filtre sur le tag spécifique `BigBoss`, nous avons pu intercepter la chaîne de caractères générée par la bibliothèque native lors de l'instanciation de la classe via le payload YAML.

**Commande de surveillance en temps réel :**

```powershell
adb logcat -s BigBoss
```

<img width="942" height="102" alt="image" src="https://github.com/user-attachments/assets/efc75f2d-3f0a-411e-ac7c-08e22c03425a" />

<img width="902" height="61" alt="image" src="https://github.com/user-attachments/assets/92ef486b-9abc-4631-a305-ff1973b80f73" />

## 5. Conclusion

Ce laboratoire a permis de mettre en pratique une chaîne d'attaque complète et réaliste sur un environnement Android, couvrant plusieurs domaines critiques de la cybersécurité mobile :

1.  **Ingénierie inverse :** Utilisation de **JADX-GUI** pour identifier les mécanismes de protection anti-root et localiser les vecteurs d'attaque dans le code Java décompilé.
2.  **Patching binaire :** Modification directe du bytecode **Smali** via **Apktool** pour neutraliser les contrôles de sécurité, démontrant que les protections côté client ne sont jamais une barrière infranchissable pour un attaquant déterminé.
3.  **Exploitation logicielle :** Utilisation de la vulnérabilité de désérialisation **SnakeYAML (CVE-2022-1471)** pour forcer l'instanciation de classes Java arbitraires (`BigBoss`), illustrant le danger des bibliothèques traitant des données externes sans validation.

**Leçon apprise :** La sécurité ne doit jamais reposer uniquement sur des vérifications locales. Il est impératif pour les développeurs d'utiliser des parseurs sécurisés (comme le `SafeConstructor` de SnakeYAML 2.0+) et de suivre le principe du moindre privilège lors de la manipulation de fichiers sur le stockage externe.

---

