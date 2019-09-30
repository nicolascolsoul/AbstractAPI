# API
<p align="center">
  <img alt="Les fleuristes" src="http://www.base-nikko.info/upload/images/1566762949guindaille.png"><br>
  Premier API pour les prototypes Glask's
</p>

## Mise à jour
- 25/08/2019 : première release de l'API et première édition de ce README.

# Usage
## Dépendances
Quelques lignes à ajouter dans l'*AndroidManifest* pour autoriser l'application à gérer le Bluetooth :
```
<uses-permission android:name="android.permission.BLUETOOTH"/>
<uses-permission android:name="android.permission.BLUETOOTH_ADMIN"/>
```
Les fichiers élémentaires :
- BluetoothDataService.java
- Glask.java
- GlaskGame.java
- GlaskEvents.java

## Connection Bluetooth
Le mieux est d'utiliser l'exemple du *MainActivity* qui permet de chercher facilement dans les périphériques Bluetooth déjà appairés. Il faut sortir un *BluetoothDevice* correspondant à celui de la base.
Une fois fait, il faut, une seule fois, passer le device à la classe *BluetoothDataService* en instanciant un objet (notez la nécessité d'un contexte).
```
bluetoothDataService = new BluetoothDataService(Example.this);
bluetoothDataService.connect(device, true);
```

## Récupération du jeu
Dans n'importe quelle activité, il est alors possible de récupérer le jeu et son état simplement.
```
game = BluetoothDataService.game.getInstance();
```
## Gestion des évènements
Il peut être judicieux de définir une série d'évènements liés à l'activité. On utilise la classe abstraite *GlaskEvents* pour définir les évènements.
```
GlaskEvents evts = new GlaskEvents() {
            @Override
            public void Filled(int index) {

            }

            @Override
            public void Drinking(int index) {

            }

            @Override
            public void Affoning(int index) {

            }

            @Override
            public void Knocked(int index) {

            }

            @Override
            public void NewPlayers(int index) {

            }

            @Override
            public void NewTime(int index) {

            }

            @Override
            public void Empty(int index) {

            }
        };
 ```
 Chaque fonction sera appelée par rapport à l'évènement auquel elle correspond. En argument se trouve l'index du Glask concerné, permettant ainsi de traiter l'information.
 - Filled : lorsqu'un gobelet vient de se remplir (a plus de 2/3).
 - Drinking : lorsqu'un utilisateur boit le contenu de son gobelet.
 - Affoning : lorsqu'un utilisateur a vidé l'intégralité de son gobelet.
 - Knocked : lorsqu'un utilisateur trinque ou frappe son gobelet.
 - NewPlayers : lorsqu'un nouvel utilisateur se connecte à la base, il est ajouté au jeu.
 - NewTime : lorsqu'un utilisateur consomme le contenu du gobelet, celui est chronométré. L'information est ensuite envoyée de manière unique.
 - Empty : lorsque le contenu du gobelet passe en dessous du dernier capteur (environ 2/3).
 
On peut alors configurer le jeu avec cette suite d'évènements.
```
game.setEvent(evts);
```
Attention aux changements d'activités avec des gestions d'évènements différents, il faut reconfigurer le jeu dans le `onResume()`.
 
## Implémentation

### RGB
Plusieurs façons de définir une couleur à un ou des gobelets :
```
game.RGB(int[] indexs, int R, int G, int B);
game.RGB(int[] index, int R, int G, int B);
game.RGB(int[] index, game.COLOR);
```
Soit un index, soit un tableau d'indexs permettant d'allumer plusieurs Glasks de la même façon. Il existe plusieurs couleurs prédéfinies dans `game.COLOR` :
```
game.RED
game.GREEN
game.BLUE
game.WHITE
game.PURPLE
game.YELLOW
game.NONE
```
Le simple fait d'exécuter cette fonction est suffisant pour allumer un Glask à la couleur concernée quasi immédiatement. Il existe une fonction permettant de *reset* les couleurs de tous les gobelets de manière optimisée :
```
game.resetAllRGB();
```
### Vibreur
Bien qu'il ne soit pas encore à 100% fonctionnel d'un point de vue hardware, on peut, de manière équivalente, définir l'état du vibreur d'un ou de plusieurs Glasks.
```
game.vibrate(int[] indexs, int state);
game.vibrate(int index, int state);
```
Où la variable `state` définit une animation selon cette syntaxe :
> xy : fait vibrer x fois pendant y * 200 ms avec pause de 200 ms

Exemple : `state = 34` fera vibrer 3 fois de manière consécutive pendant 800 ms avec une pause de 200 ms entre chaque.

Cette méthode n'est pas bloquante dans le sens ou l'arrêt du vibreur est automatique. Il n'y a pas de possibilité d'interrompre un vibreur.

### Autres
```
<int> game.getLength();
```
Obtenir le nombre de Glask(s) connecté(s).

```
<int> game.randomSelect();
```
Obtenir l'index d'un Glask aléatoire. (?)

```
<int> game.getStatus(); 
```
Retourne 1 si la base est ONLINE, 0 si OFFLINE. (?)

```
<void> game.sleepMillis(long millis); 
```
Permet de faire des actions temporisées de manière non bloquante à l'émission/réception des données. *millis* est la longueur en millisecondes du temps à attendre.

## Exemple
On fait une petite application qui permet de faire une roulette russe : les gobelets s'allument à tour de rôle et de manière aléatoire, le jeu s'arrête sur un gobelet en particulier. On pourrait, par la suite, imaginer que le jeu se bloque tant que l'utilisateur n'a pas bu son verre.

La première chose à faire est de lister les indexs (locaux) connectés au jeu. On peut simplement les ajouter lors de leur première connection. 
```
@Override
public void NewPlayers(int index) {
    // On ajoute chaque Glask qui se connecte
    playersLocalIndex.add(index);
}
```
On peut alors passer à l'algorithme en définissant un nombre aléatoire de gobelets qui vont s'allumer et le joueur par lequel l'animation va commencer.
```
Random rand = new Random();
int size = playersLocalIndex.size(); // = game.getLength();

// On définit une longueur de loop variable
int random_select = 4 * size + rand.nextInt(size);

// On définit un joueur au hasard qui va commencer l'animation
int random_first = playersLocalIndex.get(rand.nextInt(size));
```
On peut alors lancer la boucle :
```
int playerSelected = random_first;

for(int i = 0; i < random_select; i++)
{
    // On éteint le précédent
    if(i != 0) { game.RGB(playerSelected, game.NONE); }

    // Algorithme pour incrémenter/boucler sur les indexs
    if (playersLocalIndex.indexOf(playerSelected) == size - 1)
    {
        playerSelected = 0;
    }
    else
    {
        playerSelected = playerSelected + 1;
    }

    // On allume le concerné
    game.RGB(playerSelected, game.RED);
    game.sleepMillis(800);
}
// Animation du joueur qui a perdu
game.RGB(playerSelected, game.RED);
game.vibrate(playerSelected, 33);
```
En quelques lignes, on a déjà fait un jeu ludique et original sans se préoccuper de la partie hardware.




