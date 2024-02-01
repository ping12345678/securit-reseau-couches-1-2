# Protocoles de sécurité couche 1 & 2

SOURCES

CCNA et OPENCLASSROOMS

[DHCP Snooping](https://community.fs.com/fr/article/what-is-dhcp-snooping-and-how-it-works.html)

[Configurer le serveur DHCP sur les appareils Cisco](https://www.manageengine.com/fr/network-configuration-manager/configlets/configure-dhcp-server-cisco.html)

[Configuration de la surveillance DHCP de Cisco](https://www.networkstraining.com/cisco-dhcp-snooping-configuration/)

[Configuration des fonctionnalités de sécurité CISCO](https://www.cisco.com/c/fr_ca/support/docs/switches/catalyst-3750-series-switches/72846-layer2-secftrs-catl3fixed.html)

[Switchport Port-Security](https://cisco.goffinet.org/ccna/ethernet/switchport-port-security-cisco-ios/#:~:text=Cisco%20en%20IOS-,1.,port%2C%20une%20action%20est%20prise.)

[Commande port-security sur un switch Cisco 3750 ou 2960](https://blog.clemanet.com/reseaux/port-security-switch-cisco.html)

## 1. Présentation de la situation

### A. Situation initiale (Switch non paramétré)

Expliquer la configuration par défaut du switch.

Dans la situation initiale, il y a un serveur DHCP TRUST distribuant des adresses IP au Poste client par l'intermédiare d'un Switch CISCO Catalyst 2960.
Un serveur DHCP ROGUE vient se connecter au sein du réseau et tenter de distribuer une adresse IP au Poste Client en usurpant le DHCP TRUST.

Démontrer qu'il est possible d'intercepter des données dans cette situation.

### B. Situation Finale (Switch paramétré)

Décrire les paramètres appliqués sur le switch.

Aucun équipement n'est ajouté pour la résolution du problème, uniquement de la sécurisation sur l'équipement Switch CISCO Catalyst 2960.
Pour éviter à nouveau qu'un individu non-authorisé sur le réseau ne tente une attaque par usurpation DHCP, il y a quelques manipulations basiques à effectuer sur le Switch.

*Sécurisation des ports* : fermer les interfaces non utilisées.
Switch(config-if)#shutdown interface x/x


*Table d'adresses MAC* : Switch(config-if)# switchport port-security mac-address sticky
La commande sert à ce que le Switch enregistre dynamiquement les adresses MAC sur les interfaces connectées.

*Affilier une adresse MAC authorisé sur un port* : 
Switch(config)# mac address-table static 0011.2233.4455 vlan 100 interface FastEthernet0/1
- Associe le port de l'interface à une adresse MAC

*Port-security* : 

Switch(config)# interface FastEthernet0/1

Switch(config-if)# switchport port-security

Switch(config-if)# switchport port-security maximum 1

Switch(config-if)# switchport port-security violation restrict  # Mode de violation restrictif (autres options : shutdown, protect)

-Cela permet d'éteindre l'interface Fa0/1 si jamais une autre adresse MAC autre que celle du serveur DHCP TRUST serait connecté à l'interface.


*DHCP snooping* : Switch(config)# ip dhcp snooping (globale)
- Surveille la cohérence entre les adresses IP et MAC sur toutes les interfaces
  
  Switch(config)# interface Fa 0/1-2
  Switch(config-if-range)# ip dhcp snooping trust 
- Interfaces de confiances spécifiées

  *VLAN* : La segmentation du réseau permet de réduire la portée de l'attaque.
  

## 2. Attaque d'usurpation DHCP (Rogue)

### A. Fonctionnement

Expliquer comment l'attaque d'usurpation DHCP fonctionne.

*Détection du serveur DHCP légitime* : L'attaquant identifie un réseau où les clients dépendent du protocole DHCP pour obtenir une configuration réseau automatique.

*Émission de faux paquets DHCP* : L'attaquant envoie des paquets DHCP falsifiés sur le réseau, se faisant passer pour un serveur DHCP légitime. Ces paquets contiennent des informations de configuration réseau, telles que l'adresse IP, la passerelle par défaut et les serveurs DNS.

*Réponse plus rapide* : En règle générale, les serveurs DHCP légitimes répondent plus lentement que l'attaquant. Les clients, désireux de recevoir rapidement une configuration réseau, peuvent accepter la première réponse qu'ils reçoivent, qui pourrait être celle de l'attaquant.

*Clients configurés avec de fausses informations* : Les clients acceptent la configuration réseau fournie par l'attaquant. Cela peut entraîner une redirection du trafic à travers des serveurs contrôlés par l'attaquant, permettant le vol de données sensibles ou la réalisation d'attaques de type "man-in-the-middle".

### B. Enjeux et risques

Identifier les enjeux et risques associés à cette attaque.

*Interruption du service* : Une attaque DHCP peut entraîner une interruption du service réseau pour les utilisateurs légitimes, car leurs configurations réseau peuvent être modifiées de manière incorrecte ou délibérée.

*Interception du trafic* : L'attaquant peut rediriger le trafic réseau à travers ses propres serveurs, facilitant la possibilité d'intercepter et d'analyser le trafic, ce qui peut entraîner la capture de données sensibles.

*Attaques "Man-in-the-Middle" (MitM)* : En contrôlant le flux de trafic, l'attaquant peut se positionner en tant que "man-in-the-middle", facilitant la capture ou la modification de données transitant entre les clients et les serveurs légitimes.

*Vol d'identifiants* : En redirigeant le trafic vers des serveurs contrôlés par l'attaquant, ce dernier peut capturer des informations d'identification sensibles, comme les noms d'utilisateur et les mots de passe.

Pour l'attaquant cela est une énorme opportunité de récupérer des données sensibles, qui ont une valeur marchande. L'attaquant aura le choix de les utlisées ou directement les revendre.

Pour l'entité attaquée cela est un vol de ses données, qui peuvent par la suite être utilisés à l'insu des personnes concernés.
Cela, pouvant entrainer:

*Perte de productivité* : L'interruption des services réseau peut entraîner une baisse de la productivité des employés, en particulier si les services critiques dépendent d'une connexion réseau stable.

*Perte de données* : Si l'attaquant parvient à intercepter des données sensibles pendant l'attaque, cela peut entraîner une perte de données confidentielles, y compris des informations client, des données financières, etc.

*Coûts de remédiation* : Rétablir la configuration réseau correcte, identifier et résoudre les problèmes de sécurité, et mettre en œuvre des mesures préventives peuvent engendrer des coûts importants en termes de main-d'œuvre et de ressources technologiques.

*Dommages à la réputation* : Les clients et les partenaires peuvent perdre confiance envers l'entreprise si celle-ci est victime d'une attaque. La réputation de l'entreprise peut être ternie, ce qui peut avoir des conséquences à long terme.

*Pertes financières directes* : Les attaques peuvent entraîner des pertes financières directes, que ce soit en raison d'une baisse des revenus due à l'interruption des services ou de coûts supplémentaires liés à la remédiation et à la mise en place de mesures de sécurité renforcées.

*Frais juridiques et réglementaires* : Si l'attaque entraîne une violation de la vie privée ou des données, l'entreprise pourrait faire face à des frais juridiques importants et à des amendes réglementaires en raison du non-respect des lois sur la protection des données.

*Perte de contrôle* : Une attaque réussie peut compromettre le contrôle et la confidentialité des informations internes, affectant la capacité de l'entreprise à gérer ses activités de manière efficace.

## 3. Protection

### A. Équipements

Présenter les équipements ou les mesures mis en place pour se protéger contre cette attaque.

*Utilisation de DHCP sécurisé* : Les serveurs DHCP sécurisés sont configurés pour n'accepter que les requêtes provenant de serveurs DHCP autorisés. Cela réduit le risque d'usurpation en restreignant les réponses DHCP aux sources légitimes.

*Configuration correcte des commutateurs* : Les commutateurs réseau peuvent être configurés pour bloquer les paquets DHCP provenant de ports non autorisés. Cela empêche les attaquants d'envoyer des paquets DHCP malveillants depuis des ports non autorisés.

### B. Réseau

*Surveillance du trafic DHCP* : La surveillance proactive du trafic DHCP permet de détecter les anomalies et les activités suspectes. Des outils de surveillance réseau peuvent alerter les administrateurs en cas d'activité DHCP inhabituelle.

*Utilisation de l'authentification 802.1X* : L'authentification 802.1X peut renforcer la sécurité en exigeant une authentification avant qu'un appareil ne soit autorisé à accéder au réseau. Cela peut aider à prévenir les attaques en limitant l'accès aux seuls appareils autorisés.

*Mise à jour régulière des équipements réseau* : Les fabricants publient fréquemment des mises à jour de firmware pour leurs équipements réseau. S'assurer que tous les équipements sont à jour avec les derniers correctifs de sécurité est essentiel pour atténuer les vulnérabilités.

*Utilisation de VLANs* : La segmentation du réseau à l'aide de VLANs (Virtual Local Area Networks) peut limiter la portée des attaques. En isolant les appareils sur des VLANs distincts, on peut réduire l'impact potentiel d'une attaque DHCP à une partie spécifique du réseau.


*Mise en œuvre de pare-feu* : L'utilisation de pare-feu peut aider à filtrer et à bloquer le trafic malveillant, y compris les tentatives d'usurpation DHCP.

En combinant ces mesures, une entreprise peut renforcer sa résilience contre les attaques d'usurpation DHCP et réduire les risques associés à ce type de menace. Il est important de mettre en place une approche multicouche de la sécurité pour garantir une protection complète du réseau.
