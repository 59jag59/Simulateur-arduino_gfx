# Simulateur-arduino_gfx
simulateur pour librairie arduino_gfx
J’ai fait un simulateur en HTML pour tester rapidement de petits bouts de code à l’écran.
GFX_SIM, est un simulateur d'écran LCD qui tourne entièrement dans le navigateur, sans installation.

C'est un fichier HTML unique (pas de build, pas de dépendances serveur) qui simule par défaut un écran JC3248W535 (480×320, RGB565) avec la librairie Arduino_GFX, mais la résolution, le nom de l'écran et le pin backlight sont configurables — il peut donc simuler n'importe quel afficheur (ILI9341, ST7789, ST7735, SSD1306, etc.). Il suffit de cliquer sur les chips en haut de l'interface pour changer.
Le principe : vous écrivez du code Arduino C++ dans l'éditeur intégré, le simulateur le transpile en JavaScript à la volée, et l'exécute sur un canvas HTML5. Le résultat s'affiche en temps réel — sans avoir besoin du hardware.

Comment ça marche sous le capot
Deux modes d'utilisation
Mode rapide (sans setup/loop) : vous écrivez directement des appels GFX et le résultat s'affiche en temps réel à chaque Run. Idéal pour tester une seule fonction, ajuster un positionnement ou prototyper un écran rapidement — pas besoin d'écrire void setup() / void loop().
Mode complet (setup/loop) : le simulateur détecte automatiquement void setup() et void loop(), active le mode boucle, et gère la sauvegarde/restauration des variables entre chaque itération. Parfait pour les animations, horloges, interfaces interactives avec delay().
Transpiler C++ → JavaScript
Le cœur du simulateur est un transpiler qui convertit le code Arduino en JS exécutable :
Fonctions GFX : gfx->fillRect(...) → await gfxAPI.fillRect(...) — chaque fonction GFX de la librairie Arduino_GFX est réimplémentée en JS pour dessiner sur le canvas
Structures C++ : parsing en 3 passes (définition struct → conversion des instanciations en objets JS → suppression des noms de type dans les paramètres)
Types C++ : int, float, double, uint8_t, uint16_t, etc. sont convertis en let, avec gestion des cast entiers (_u8(), _u16(), _i32(), etc.)
#define : les macros avec arguments deviennent des fonctions JS, les constantes deviennent des const
setup() / loop() : détection automatique, sauvegarde/restauration des variables globales entre les itérations de loop
delay() : converti en await _asyncDelay() pour ne pas bloquer le navigateur
Boucles infinies : des gardes automatiques empêchent les while et do...while de geler le navigateur (limite 100 000 itérations)
Fonctions GFX supportées
Toutes les primitives classiques d'Arduino_GFX sont implémentées :
fillScreen, fillRect, drawRect, fillRoundRect, drawRoundRect
drawCircle, fillCircle, drawLine, drawFastHLine, drawFastVLine
drawTriangle, fillTriangle, drawPixel
setCursor, setTextColor, setTextSize, print, println, printf
drawBitmap, draw16bitRGBBitmap
setRotation, width(), height()
Font CP437 5×7 intégrée (avec table de conversion Unicode ↔ CP437)
Couleurs RGB565 nommées (RGB565_RED, RGB565_GREEN, etc.) + valeurs hex
Simulation hardware
Touch screen : clic sur le canvas = simulation du contrôleur tactile AXS15231B avec coordonnées corrigées selon la rotation (getTouchPoint, touchX, touchY)
GPIO : popup avec LEDs visuelles, sliders analogiques et toggles digitaux — digitalWrite, analogRead, analogWrite, ledcWrite/Setup/AttachPin
Serial Monitor : popup avec sortie Serial.print/println/printf et envoi vers Serial.read()
Backlight : analogWrite(GFX_BL, val) ajuste la luminosité via filtre CSS
Sleep/Wake : enterSleep() / wakeUp() simulés
millis(), delay(), random(), map(), constrain() — tout est là
Mode Édition visuelle ✏️
Après exécution du code, un mode Edit permet de :
Sélectionner les objets graphiques directement sur le canvas (cadre jaune + poignées)
Déplacer / Redimensionner par drag & drop
Modifier les propriétés (x, y, w, h, couleur, texte) dans un panneau
Ajouter de nouveaux objets graphiques
Supprimer des objets (la ligne source est commentée)
Synchronisation bidirectionnelle : toute modification visuelle met à jour le code source, et vice versa
Undo/Redo dédié au mode GFX
Éditeur de code
Coloration syntaxique
Autocomplétion des fonctions GFX, mots-clés et variables utilisateur
Numéros de ligne
Raccourcis clavier (Ctrl+Enter pour exécuter, Tab pour indenter)
Sauvegarde automatique dans localStorage
Autres features
Zoom du canvas (molette, pinch sur mobile, slider)
Tout afficheur : résolution, nom et backlight configurables en un clic — pas limité au JC3248W535
11 exemples intégrés : bar chart, test C++, touch screen, animation avec delay, cercles concentriques, table CP437, horloge setup/loop, calculatrice, bitmap, serial menu, gauge avec sliders
Chargement de fichiers .h (bitmaps, constantes)
Mode portrait/paysage
Breakpoints et inspection de variables pour le debug
Fonctionne sur mobile (touch, pinch-to-zoom)

Ce que je cherche
Le transpiler couvre beaucoup de cas mais il y a forcément des limites. J'aimerais que des développeurs Arduino/ESP32 testent avec leur propre code pour voir :
Erreurs de transpilation — du code C++ valide qui ne compile pas ou donne un résultat différent du hardware réel
Fonctions GFX manquantes — des appels Arduino_GFX que j'aurais oublié d'implémenter
Bugs d'affichage — rendu différent entre le simulateur et l'écran réel (couleurs, positions, taille texte...)
Problèmes d'ergonomie — interface pas intuitive, trucs qui manquent
Cas limites du C++ — pointeurs, templates, certaines syntaxes que le transpiler ne gère pas (et que je devrais documenter comme limitations)
⚠️ Limitations connues : ce n'est pas un compilateur C++ complet. Les pointeurs, les classes complexes, les templates, le multi-fichier (au-delà des .h simples) ne sont pas supportés. C'est fait pour du code Arduino typique d'interface graphique.

Comment tester
Téléchargez le fichier HTML
Ouvrez-le dans un navigateur (Chrome/Firefox/Edge)
Choisissez un exemple dans le menu déroulant, ou collez votre propre code
Cliquez ▶ Run
Testez le touch en cliquant sur l'écran
Aucune installation, aucun serveur, tout est dans le fichier.

Merci d'avance pour vos retours ! Si ça tient la route, je publierai sur GitHub avec une licence CC BY-NC 4.0.
Développé par 59jag — 59jag59@gmail.com
