# Code de la carte Maître des 3 robots
## Codé sur Wemos R1 D2 embarquant un module ESP8266

'''c
#include <ESP8266WiFi.h>

const char *id = "ESPap";  // SSID du point d'accès
const char *mdp = "thereisnospoon";    // Mot de passe du point d'accès
WiFiServer server(80);  // Port 80 pour le serveur socket
const int maxEsclaves = 3;  // Nombre maximum de robots esclaves
WiFiClient esclaves[maxEsclaves];  // Tableau pour stocker les clients esclaves

const int buttonPin1 = D3;  // Broche du bouton 1
const int buttonPin2 = D5;
const int buttonPin3 = D6;
const int buttonPin4 = D7;

const int debounceDelay = 50;  // Délai pour debounce du bouton
int der_etat_1 = HIGH;
int der_etat_2 = HIGH;
int der_etat_3 = HIGH;
int der_etat_4 = HIGH;
bool etat_act_1 = LOW;
bool etat_act_2 = LOW;
bool etat_act_3 = LOW;
bool etat_act_4 = LOW;

const int ledPin4 = D8;  // Broche de la LED
const int ledPin3 = D2;  // Broche de la LED
const int ledPin2 = D1;  // Broche de la LED
const int ledPin1 = D0;  // Broche de la LED

void setup() {
  Serial.begin(115200);

  pinMode(buttonPin1, INPUT_PULLUP);  // Configurer les boutons comme entrées avec pullup
  pinMode(buttonPin2, INPUT_PULLUP);
  pinMode(buttonPin3, INPUT_PULLUP);
  pinMode(buttonPin4, INPUT_PULLUP);

  pinMode(LED_BUILTIN, OUTPUT);  // LED intégrée pour visualisation
  digitalWrite(LED_BUILTIN, HIGH);  // Éteindre la LED par défaut

  pinMode(ledPin1, OUTPUT);  // Configurer la LED comme sortie
  digitalWrite(ledPin1, LOW);  // LED éteinte par défaut
  pinMode(ledPin2, OUTPUT);  // Configurer la LED comme sortie
  digitalWrite(ledPin2, LOW);  // LED éteinte par défaut
  pinMode(ledPin3, OUTPUT);  // Configurer la LED comme sortie
  digitalWrite(ledPin3, LOW);  // LED éteinte par défaut
  pinMode(ledPin4, OUTPUT);  // Configurer la LED comme sortie
  digitalWrite(ledPin4, LOW);  // LED éteinte par défaut

  // Configurer la carte en mode Access Point
  WiFi.softAP(id, mdp);

  // Démarrer le serveur socket
  server.begin();
  Serial.println("Serveur démarré");
  Serial.print("Adresse IP : ");
  Serial.println(WiFi.softAPIP());  // Afficher l'adresse IP de notre point d'accès
}

void loop() {
  // Vérifier si un nouveau client (esclave) veut se connecter
  recep_esclaves();

  // Vérifier chaque bouton (BP1 à BP4) et envoyer des commandes à tous les esclaves
  Verif_BP_envoi_infos();
  
  delay(5);  // Délai très court pour éviter de surcharger le CPU
}

// Fonction pour que les robots puissent atteindre le serveur créé
void recep_esclaves() {
  WiFiClient nouveau_client = server.available();  // Vérifier si un nouveau client veut se connecter
  if (nouveau_client) {
    Serial.println("Nouveau client en attente de connexion...");
    for (int i = 0; i < maxEsclaves; i++) {
      // Trouver un slot vide pour le nouveau client
      if (!esclaves[i] || !esclaves[i].connected()) {
        esclaves[i] = nouveau_client;
        Serial.print("Esclave connecté à l'index ");
        Serial.println(i);
        return;
      }
    }
    // Si tous les slots sont pleins, rejeter le nouveau client
    nouveau_client.stop();
    Serial.println("Client rejeté, tous les slots d'esclaves sont pleins.");
  }
}

// Fonction pour vérifier les boutons et envoyer des commandes aux esclaves
void Verif_BP_envoi_infos() {
  // Bouton 1
  bool etat_BP1 = digitalRead(buttonPin1);
  if (etat_BP1 != der_etat_1) {
    delay(debounceDelay);  // Debounce simple
    etat_BP1 = digitalRead(buttonPin1);  // Re-lire après le délai

    if (etat_BP1 == LOW) {  // Bouton appuyé
      Serial.println("BP 1 APPUYÉ");
      envoi_cmd_esclaves("1");  // Envoyer "1" à tous les esclaves
      digitalWrite(ledPin1, HIGH);  // Allumer la LED
      etat_act_1 = !etat_act_1;  // Inverser l'état
    }
    else {
      digitalWrite(ledPin1, LOW);
    }
  }
  der_etat_1 = etat_BP1;

  // Bouton 2
  bool etat_BP2 = digitalRead(buttonPin2); 
  if (etat_BP2 != der_etat_2) {
    delay(debounceDelay);  // Debounce simple
    etat_BP2 = digitalRead(buttonPin2);  // Re-lire après le délai

    if (etat_BP2 == LOW) {  // Bouton appuyé
      Serial.println("BP 2 APPUYÉ");
      envoi_cmd_esclaves("2");  // Envoyer "1" à tous les esclaves
      digitalWrite(ledPin2, HIGH);  // Allumer la LED
      etat_act_2 = !etat_act_2;  // Inverser l'état
    }
    else {
      digitalWrite(ledPin2, LOW);
    }
  }
  der_etat_2 = etat_BP2;

  // Bouton 3
  bool etat_BP3 = digitalRead(buttonPin3);
  if (etat_BP3 != der_etat_3) {
    delay(debounceDelay);  // Debounce simple
    etat_BP3 = digitalRead(buttonPin3);  // Re-lire après le délai

    if (etat_BP3 == LOW) {  // Bouton appuyé
      Serial.println("BP 3 APPUYÉ");
      envoi_cmd_esclaves("3");  // Envoyer "1" à tous les esclaves
      digitalWrite(ledPin3, HIGH);  // Allumer la LED
      etat_act_3 = !etat_act_3;  // Inverser l'état
    }
    else {
      digitalWrite(ledPin3, LOW);
    }
  }
  der_etat_3 = etat_BP3;

  // Bouton 4
  bool etat_BP4 = digitalRead(buttonPin2);
  if (etat_BP4 != der_etat_4) {
    delay(debounceDelay);  // Debounce simple
    etat_BP4 = digitalRead(buttonPin4);  // Re-lire après le délai

    if (etat_BP4 == LOW) {  // Bouton appuyé
      Serial.println("BP 4 APPUYÉ");
      envoi_cmd_esclaves("4");  // Envoyer "1" à tous les esclaves
      digitalWrite(ledPin4, HIGH);  // Allumer la LED
      etat_act_4 = !etat_act_4;  // Inverser l'état
    }
    else {
      digitalWrite(ledPin4, LOW);
    }
  }
  der_etat_4 = etat_BP4;

}

// Fonction pour envoyer une commande à tous les esclaves connectés
void envoi_cmd_esclaves(const char* commande) {
  for (int i = 0; i < maxEsclaves; i++) {
    if (esclaves[i] && esclaves[i].connected()) {
      esclaves[i].println(commande);  // Envoyer la commande à l'esclave
      Serial.print("Commande envoyée à l'esclave ");
      Serial.print(i);
      Serial.print(" : ");
      Serial.println(commande);
    }
  }
}
'''
