# Code d'une des carte Esclave (dupliqué sur les deux autres)
## Codé sur NodeMCU embarquant un module ESP8266

```c
#include <ESP8266WiFi.h>
#include <ESP8266WiFiMulti.h>
#include <Servo.h> // Bibliothèque Servo pour contrôler les servos
#include <math.h> // Pour utiliser sin() et M_PI

#ifndef STASSID
#define STASSID "ESPap"
#define STAPSK  "thereisnospoon"
#endif

// Déclaration des broches pour chaque servomoteur
#define LeftLeg 14     // Broche pour la jambe gauche D5
#define RightLeg 5    // Broche pour la jambe droite D1
#define LeftFoot 4    // Broche pour le pied gauche D2
#define RightFoot 16 // Broche pour le pied droit D0

// Création des objets Servo pour chaque servomoteur
Servo leftLegServo;
Servo rightLegServo;
Servo leftFootServo;
Servo rightFootServo;

// Déclaration des angles de position "Home" (neutres)
int homeLeftLeg = 85;   // Angle neutre pour la jambe gauche
int homeRightLeg = 92;  // Angle neutre pour la jambe droite
int homeLeftFoot = 75;  // Angle neutre pour le pied gauche
int homeRightFoot = 90; // Angle neutre pour le pied droit

int emetteur = 12; // Emetteur -> TRIGGER
int recepteur = 13; // Recepteur -> ECHO
long duree_impulsion;
float dist_mesuree;
float temps_act;
int T_out=300000;

const char* ssid     = STASSID;
const char* password = STAPSK;

const char* host = "192.168.4.1";  // IP du maître
const uint16_t port = 80;           // Port du serveur maître

ESP8266WiFiMulti WiFiMulti;

const int ledPin = D2;  // Broche de la LED
WiFiClient client;      // Déclarer le client ici pour garder la connexion ouverte

int iEtat = 0;

void setup() {
  pinMode(emetteur, OUTPUT); // Emetteur 
  pinMode(recepteur, INPUT); // Recepteur 
  digitalWrite(emetteur, LOW); 
  Serial.begin(9600);
  // Attache les servos aux broches correspondantes
  leftLegServo.attach(LeftLeg);
  rightLegServo.attach(RightLeg);
  leftFootServo.attach(LeftFoot);
  rightFootServo.attach(RightFoot);
  delay(500);
  Home();

  Serial.begin(115200);
  pinMode(ledPin, OUTPUT);  // Configurer la LED comme sortie
  digitalWrite(ledPin, LOW);  // LED éteinte par défaut

  // Configuration du mode station et ajout du point d'accès
  WiFi.mode(WIFI_STA);
  WiFiMulti.addAP(ssid, password);

  Serial.println();
  Serial.println("Attente de la connexion WiFi...");

  while (WiFiMulti.run() != WL_CONNECTED) {
    Serial.print(".");
    delay(500);
  }

  Serial.println("");
  Serial.println("WiFi connecté");
  Serial.print("Adresse IP : ");
  Serial.println(WiFi.localIP());

  // Connexion initiale au serveur
  connectToServer();
}


//*****************************************LOOP*******************************************//
void loop() {
  // Vérifier si la connexion au serveur est toujours active
  if (!client.connected()) {
    connectToServer();
    return; // Sortir de la boucle si la connexion a échoué
  }

  // Envoi d'une requête au serveur
  client.println("Hello ROBOT A");

  // Lecture de la réponse du serveur
  if (client.available()) {  // Lire uniquement si des données sont disponibles
    String message = client.readStringUntil('\n');  // Lire jusqu'à la nouvelle ligne
    message.trim();  // Nettoyer le message

    Serial.print("Message reçu : ");
    Serial.println(message);

    // Traiter le message
    if (message == "1") {
      iEtat = 10;
    } else if (message == "2") {
      iEtat = 20;
    } else if (message == "3") {
      iEtat = 30;
    }
    else if (message == "4") {
      iEtat = 40;
    }

    // Exécuter les actions en fonction de l'état
    switch (iEtat) {
      case 10:
        DanceTango(5, 1000); 
        iEtat = 0;
        break;
      case 20:
        DanceOnToes(5, 1000); 
        iEtat = 0;
        break;
      case 30:
        DanceBalance(5, 1000);  // Appelle la deuxième danse
        iEtat = 0;
        break;
      case 40:
        DanceBalanceEqui(5, 1000); 
        iEtat = 0;
        break;
      default:
        Home();  // Retourner à la position par défaut
        break;
    }
  }

  // Attendre un court moment avant la prochaine itération
  delay(100);  // Délai réduit pour une meilleure réactivité
}
//*************************************************************************************************//



//****************************Fonction Connection au serveur*****************************************//
void connectToServer() {
  if (!client.connect(host, port)) {
    Serial.println("Échec de la connexion au serveur, nouvelle tentative dans 2 secondes...");
    delay(2000);  // Attendre un peu avant de réessayer
  } else {
    Serial.println("Connecté au serveur");
  }
}
//*************************************************************************************************//



//**************************************Fonction UltraSons************************************************//
int Ultrasons(){
  digitalWrite(emetteur, HIGH); // Activationde  la sortie de l'émetteur afin d'envoyer une onde
  delayMicroseconds(10); // Impulsion a l'état haut de 10 microsecondes
  digitalWrite(emetteur, LOW); // Désactivation de l'émetteur pour que le récepteur prenne le relais
  duree_impulsion = pulseIn(recepteur,HIGH,T_out); // Mesure du temps écoulé entre l'envoi et la réception de l'onde
  temps_act = duree_impulsion-T_out;
  dist_mesuree = duree_impulsion*0.034/2; // Caclul pour la conversion en cm => valeur lue sur echo divisee par 58
  return dist_mesuree;
  delay(50); // Delai de 1 seconde entre chaque mesure
}
//*************************************************************************************************//

//**************************************Fonction Home************************************************//
void Home() {
  leftLegServo.write(homeLeftLeg);
  rightLegServo.write(homeRightLeg);
  leftFootServo.write(homeLeftFoot);
  rightFootServo.write(homeRightFoot);
  Serial.println("Home");
  delay(500); // Délai plus court pour éviter les blocages
}
//*************************************************************************************************//

//**************************************Fonction Danse d'Équilibre Améliorée************************************************//
void DanceBalanceEqui(int steps, int timePerStep) {
  // Définir les paramètres d'oscillation
  int A[4] = {0, 0, 25, 25};  // Amplitudes pour les pieds
  int O[4] = {homeLeftLeg, homeRightLeg, homeLeftFoot, homeRightFoot};  // Positions neutres
  double phase_diff[4] = {0, 0, 0, 0};  // Pas de déphasage pour un mouvement synchronisé

  int currentStep = 0;  // Variable pour stocker l'étape actuelle
  int currentTime = 0;  // Variable pour stocker la progression dans l'étape actuelle

  while (currentStep < steps) {
    // Lire la distance de l'obstacle
    double distanceMeusure = Ultrasons();
    
    // Si un obstacle est détecté à moins de 10 cm, mettre la danse en pause
    if (distanceMeusure <= 10) {
      Serial.println("Obstacle détecté. Danse en pause...");
      delay(100);  // Pause courte pour éviter une surcharge

      // Boucle d'attente jusqu'à ce que l'obstacle soit éloigné (> 10 cm)
      while (distanceMeusure <= 10) {
        distanceMeusure = Ultrasons();
        delay(100);  // Réévaluation périodique
      }

      Serial.println("Obstacle dégagé. Reprise de la danse...");
    }

    // Reprise de la danse après que l'obstacle a été dégagé
    double progress = (double)currentTime / timePerStep;  // Progression dans le cycle (0 à 1)

    // Calculer la position pour les pieds avec oscillation
    double leftFootPos = O[2] + A[2] * sin(2 * M_PI * progress);  // Mouvement du pied gauche
    double rightFootPos = O[3] + A[3] * sin(2 * M_PI * progress);  // Mouvement du pied droit

    // Appliquer les positions aux servos
    leftFootServo.write(leftFootPos);
    rightFootServo.write(rightFootPos);

    // Mise à jour du temps pour cette étape
    currentTime += 50;
    delay(50);  // Pause courte entre les mises à jour

    // Si le temps pour cette étape est écoulé, passer à l'étape suivante
    if (currentTime >= timePerStep) {
      currentStep++;
      currentTime = 0;  // Réinitialiser le compteur de temps pour la prochaine étape

      // Changer la position des jambes pour simuler un mouvement de danse
      leftFootServo.write(homeLeftFoot + 30);  // Position élevée pour le pied gauche
      rightFootServo.write(homeRightFoot + 30);  // Position élevée pour le pied droit
      delay(1000);  // Attendre 1 seconde
      while (distanceMeusure <= 10) {
        distanceMeusure = Ultrasons();
        delay(100);  // Réévaluation périodique
      }
      leftFootServo.write(homeLeftFoot - 30);  // Position basse pour le pied gauche
      rightFootServo.write(homeRightFoot - 30);  // Position basse pour le pied droit
      delay(1000);  // Attendre 1 seconde
    }
  }

  Serial.println("Danse avec Oscillation Terminée");
  Home();  // Ramener les servos à la position neutre à la fin
}
//*************************************************************************************************//

//*************************************DanceBalance*************************************************//
void DanceBalance(int steps, int timePerStep) {
  int A[4] = {0, 0, 25, 25};  // Amplitudes : les jambes ne bougent pas, seulement les pieds
  int O[4] = {homeLeftLeg, homeRightLeg, homeLeftFoot, homeRightFoot};  // Positions neutres
  double phase_diff[4] = {0, 0, 0, 0};  // Pas de déphasage, mouvement synchronisé

  int currentStep = 0;  // Variable pour stocker l'étape actuelle
  int currentTime = 0;  // Variable pour stocker la progression dans l'étape actuelle

  while (currentStep < steps) {
    // Lire la distance de l'obstacle
    double distanceMeusure = Ultrasons();
    
    // Si un obstacle est détecté à moins de 10 cm, arrêter la danse
    if (distanceMeusure <= 10) {
      Serial.println("Obstacle détecté. Danse en pause...");
      delay(100);  // Pause courte pour ne pas surcharger le CPU
      
      // Boucle d'attente jusqu'à ce que l'obstacle soit à plus de 10 cm
      while (distanceMeusure <= 10) {
        distanceMeusure = Ultrasons();
        delay(100);  // Réévaluation périodique
      }
      
      Serial.println("Obstacle dégagé. Reprise de la danse...");
    }

    // Reprise de la danse après que l'obstacle est dégagé
    double progress = (double)currentTime / timePerStep;

    // Les pieds montent et descendent comme un saut
    double leftFootPos = O[2] + A[2] * sin(2 * M_PI * progress);
    double rightFootPos = O[3] + A[3] * sin(2 * M_PI * progress);

    // Appliquer les positions aux pieds
    leftFootServo.write(leftFootPos);
    rightFootServo.write(rightFootPos);

    // Mise à jour du temps pour cette étape
    currentTime += 50;
    delay(50);  // Pause courte entre les cycles

    // Si le temps pour cette étape est écoulé, passer à l'étape suivante
    if (currentTime >= timePerStep) {
      currentStep++;
      currentTime = 0;  // Réinitialiser le compteur de temps pour la prochaine étape
    }
  }

  Serial.println("Danse Style 4 Terminée");
  Home();  // Ramener le robot à la position neutre
}
//*************************************************************************************************//


//****************************************DanceOnToes*********************************************************//
void DanceOnToes(int steps, int timePerStep) {
  // Définir les paramètres d'oscillation
  int A[4] = {0, 0, 30, -30};  // Amplitudes : les jambes ne bougent pas, seulement les pieds
  int O[4] = {homeLeftLeg, homeRightLeg, homeLeftFoot, homeRightFoot};  // Positions neutres
  double phase_diff[4] = {0, 0, 0, 0};  // Pas de déphasage, mouvement synchronisé

  int currentStep = 0;  // Étape actuelle
  int currentTime = 0;  // Temps dans l'étape actuelle

  while (currentStep < steps) {
    // Lire la distance avec le capteur à ultrasons
    double distanceMeusure = Ultrasons();

    // Si un obstacle est détecté à moins de 10 cm, mettre la danse en pause
    if (distanceMeusure <= 10) {
      Serial.println("Obstacle détecté. Danse en pause...");
      delay(100);

      // Attendre jusqu'à ce que l'obstacle soit éloigné
      while (distanceMeusure <= 10) {
        distanceMeusure = Ultrasons();
        delay(100);
      }

      Serial.println("Obstacle dégagé. Reprise de la danse...");
    }

    // Progression dans la montée sur les pointes
    double progress = (double)currentTime / timePerStep;
    double leftFootPos = O[2] + A[2] * sin(M_PI * progress);  // Monter sur les pointes
    double rightFootPos = O[3] + A[3] * sin(M_PI * progress);

    // Appliquer les positions aux servos
    leftFootServo.write(leftFootPos);
    rightFootServo.write(rightFootPos);

    // Mise à jour du temps pour cette étape
    currentTime += 50;
    delay(50);  // Pause courte entre les cycles

    // Si le temps pour cette étape est écoulé, passer à l'étape suivante
    if (currentTime >= timePerStep) {
      currentStep++;
      currentTime = 0;

      // Redescendre sur les talons
      for (int t = 0; t < timePerStep; t += 50) {
        progress = (double)t / timePerStep;

        // Descendre des pointes vers les talons
        leftFootPos = O[2] - A[2] * sin(M_PI * progress);  // Descente sur les talons
        rightFootPos = O[3] - A[3] * sin(M_PI * progress);

        // Appliquer les positions aux servos
        leftFootServo.write(leftFootPos);
        rightFootServo.write(rightFootPos);

        delay(50);  // Pause courte entre les cycles

        // Vérifier à nouveau la distance pendant la descente
        distanceMeusure = Ultrasons();
        if (distanceMeusure <= 10) {
          Serial.println("Obstacle détecté. Danse en pause...");
          delay(100);

          // Attendre jusqu'à ce que l'obstacle soit éloigné
          while (distanceMeusure <= 10) {
            distanceMeusure = Ultrasons();
            delay(100);
          }

          Serial.println("Obstacle dégagé. Reprise de la danse...");
        }
      }
    }
  }

  Serial.println("Danse sur les pointes et talons terminée");
  Home();  // Ramener le robot à la position neutre
}
//*************************************************************************************************//

void DanceTango(int steps, int timePerStep) {
  int A[4] = {40, 40, 15, 15};  // Amplitudes pour jambes et pieds
  int O[4] = {90, 90, 90, 90};  // Offsets : tout part de 90°
  double phase_diff[4] = {0, M_PI / 2, M_PI / 2, M_PI / 4};  // Décalage en diagonale

  int currentStep = 0;  // Étape actuelle
  int currentTime = 0;  // Temps dans l'étape actuelle

  while (currentStep < steps) {
    double distanceMeusure = Ultrasons();  // Mesurer la distance avec le capteur à ultrasons

    // Si un obstacle est détecté à moins de 10 cm, mettre la danse en pause
    if (distanceMeusure <= 10) {
      Serial.println("Obstacle détecté. Danse en pause...");
      delay(100);

      // Attendre jusqu'à ce que l'obstacle soit dégagé
      while (distanceMeusure <= 10) {
        distanceMeusure = Ultrasons();
        delay(100);
      }

      Serial.println("Obstacle dégagé. Reprise de la danse...");
    }

    // Progression dans la danse tango
    double progress = (double)currentTime / timePerStep;

    // Calculer la position des jambes et des pieds
    double leftLegPos = O[0] + A[0] * sin(2 * M_PI * progress);
    double rightLegPos = O[1] + A[1] * sin(2 * M_PI * progress + phase_diff[1]);
    double leftFootPos = O[2] + A[2] * sin(2 * M_PI * progress + phase_diff[2]);
    double rightFootPos = O[3] + A[3] * sin(2 * M_PI * progress + phase_diff[3]);

    // Appliquer les positions calculées aux servos
    leftLegServo.write(leftLegPos);
    rightLegServo.write(rightLegPos);
    leftFootServo.write(leftFootPos);
    rightFootServo.write(rightFootPos);

    // Mise à jour du temps pour cette étape
    currentTime += 50;
    delay(50);  // Pause courte entre les cycles

    // Si le temps pour cette étape est écoulé, passer à l'étape suivante
    if (currentTime >= timePerStep) {
      currentStep++;
      currentTime = 0;

      delay(300);  // Pause légèrement plus longue entre les pas
    }
  }

  Serial.println("Danse tango terminée");
  Home();  // Ramener le robot à la position neutre
}


```
