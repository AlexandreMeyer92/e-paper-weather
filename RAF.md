# RAF - Station meteo e-paper

Derniere mise a jour : 2026-10-27
Depot personnel : https://github.com/AlexandreMeyer92/e-paper-weather
Version preparee : `1.8.1-crowpanel` (`e-paper-weather-v1.8.1`)

## Etat

- [x] Fork personnel cree; branche `main` et tag `e-paper-weather-v1.8.1` publies.
- [x] Emplacement initial regle sur Issy-les-Moulineaux, France (`48.82104, 2.27718`), fuseau Europe/Paris.
- [x] Interface configuree en francais (`fr_FR`) et unites metriques : degres C, km/h, hPa, km et mm.
- [x] Compilation locale reussie pour l'environnement CrowPanel courant avec PlatformIO Core 6.2.0 et Arduino-ESP32 3.3.12.
- [x] Geocodage Open-Meteo verifie pour Issy-les-Moulineaux, code postal 92130.
- [ ] Aucun essai sur appareil reel n'a encore ete effectue : la carte n'est pas encore disponible.

## Materiel et revision

- [ ] Acheter/recevoir le Elecrow CrowPanel ESP32-S3 E-Paper 4.2 pouces, reference DIE07300S001 / DIE07300S, dalle noir et blanc 400 x 300, controleur SSD1683.
- [ ] Verifier la revision au dos avant compilation et flash : pastille verte ronde = `crowpanel_esp32s3_42_epaper`; ancienne revision sans pastille verte = `crowpanel_esp32s3_42_epaper_legacy`.
- [ ] Prevoir un cable USB-C de donnees et une alimentation USB stable. Aucune batterie n'est requise pour le premier essai.
- [ ] Ne pas activer Secure Boot, chiffrement flash ou eFuses dans ce deploiement.

## Configuration locale

- [ ] Copier `platformio/include/secrets.h.example` vers `platformio/include/secrets.h` et renseigner le SSID et le mot de passe Wi-Fi localement.
- [ ] Ne jamais committer `secrets.h` ni partager le binaire compile avec ces identifiants.
- [x] Aucune cle meteo n'est necessaire : les donnees proviennent d'Open-Meteo.
- [ ] Confirmer que le routeur offre du Wi-Fi 2,4 GHz et que la carte aura acces a Internet.

## Premier flash USB

A executer dans PowerShell depuis `platformio/`, avec le nom de port remplace par celui de la carte :

```powershell
pio run -e crowpanel_esp32s3_42_epaper
pio device list
pio run -e crowpanel_esp32s3_42_epaper -t upload --upload-port COMx
pio device monitor -p COMx -b 115200
```

- [ ] Confirmer le port serie COM avec `pio device list`.
- [ ] Au premier flash, verifier que PlatformIO ecrit le bootloader, la table de partitions CrowPanel et le firmware.
- [ ] Si necessaire, entrer en mode telechargement avec BOOT/RST selon les instructions Elecrow.
- [ ] Ne pas flasher `firmware.bin` a l'adresse 0 : l'image combinee usine, si elle est utilisee, commence a l'adresse 0; l'OTA utilise seulement `firmware.bin`.
- [ ] Relever les messages serie, l'adresse IP et `[ota] ready`.

La table `platformio/partitions_crowpanel_ota.csv` contient deux emplacements applicatifs necessaires a l'OTA. Ne pas la remplacer par `huge_app.csv`.

## Validation sur la carte

- [ ] Ouvrir `http://<IP-de-la-carte>/`, puis verifier `/api/status` et `/screenshot.bmp` depuis le meme reseau local.
- [ ] Confirmer sur le panneau physique que la ville, la date et les valeurs meteo sont lisibles; la capture BMP seule ne prouve pas que la dalle a change.
- [ ] Verifier que la ville affichee est bien Issy-les-Moulineaux, France. Si le geocodage renvoie un resultat incorrect, ne pas valider le code postal seul sans verifier le pays et les coordonnees.
- [ ] Changer de ville depuis la page web, verifier le redemarrage, puis couper et remettre l'alimentation pour confirmer la persistance NVS (memoire non volatile de l'ESP32), sans reflash.
- [ ] Laisser l'appareil connecte et alimente au moins 65 a 75 minutes sans rafraichissement manuel ni redemarrage. Le firmware vise un cycle toutes les 30 minutes apres la fin du precedent.
- [ ] Surveiller le serie a 115200 : nouvelles requetes Open-Meteo, reponses HTTP reussies, `[epd] weather frame refresh complete`, absence de timeout BUSY et hausse de l'uptime sans reboot.
- [ ] Comparer les photos du panneau et les horodatages avant/apres. Ne pas conclure a un echec simplement parce que la temperature est restee stable.
- [ ] Repeter apres avoir restaure l'intervalle de 30 minutes si celui-ci a ete temporairement reduit pour les essais.

Note : les alertes du firmware proviennent du National Weather Service americain. Pour Issy-les-Moulineaux, ne pas considerer cette fonction comme une source d'alertes officielles francaises.

## OTA et securite acceptee

- [ ] Apres validation USB, compiler le meme environnement et televerser uniquement `platformio/.pio/build/<environnement>/firmware.bin` depuis la page web; garder l'alimentation jusqu'au redemarrage.
- [ ] Verifier apres OTA que l'appareil redemarre, conserve sa ville et reprend les cycles meteo.
- [ ] Ne pas envoyer `bootloader.bin`, `partitions.bin` ou `firmware.factory.bin` par la route OTA.
- [ ] Garder l'appareil sur un reseau local de confiance et ne pas rediriger ses ports depuis Internet.

Le firmware actuel conserve volontairement les mecanismes OTA sans protection par mot de passe : `OTA_PASSWORD` est vide et la route web `POST /update` ne demande pas d'authentification. En cas d'echec Wi-Fi, le portail de provisioning WiFiManager ouvre aussi un point d'acces local sans mot de passe et propose son propre televersement firmware. L'utilisateur accepte ce risque pour son usage domestique; aucune correction de ces mecanismes n'a ete faite.

## Versions et publication

- Le code firmware annonce `1.8.1-crowpanel`; le tag publie est `e-paper-weather-v1.8.1`.
- Le workflow GitHub de release ecoute les tags `v*`; le tag prefixe `e-paper-weather-` ne declenche donc pas ce workflow. Le build local a produit `firmware.bin` et `firmware.factory.bin`, mais aucune release GitHub de binaires n'a ete creee par ce tag.
- Les warnings `-Wformat` observes dans le scan Wi-Fi de `client_utils.cpp` restent non bloquants et hors du changement de configuration.
