Petit rapport :


1. Docker compose

a. Chargez l’image Docker webapp-1.tar
-> On charge l'image via la commande "docker load -i webapp-1.tar"
Créez un fichier compose.yaml avec un service mongodb/mongodb-community-server pour la démarrer. (images sur le Moodle)
-> Voir le fichier 'compose.yaml' + "docker compose up -d"

b. Essayez d’ajouter un utilisateur.
-> Cela ne fonctionne pas. ("ERR_EMPTY_RESPONSE")

c. Il est possible que Lucas n’aie pas bien testé. Quel est le problème ? Comment le voyez-vous ?
-> Suite à l'erreur ci-dessus, on vérifie les logs du conteneur 'webapp-1' avec la commande : "docker compose logs webapp-1".
On remarque alors : "webapp-1  | 19:59:40 [ERROR] unable to create /var/log/webapp/webapp.log (No such file or directory (os error 2)), exiting".
L'erreur survient car le répertoire '/var/log/webapp' n'existe pas. Il est donc impossible de créer le fichier 'webapp.log'.

d. « Faut qu’ça tourne! » Redémarrez le conteneur webapp et connectez-y vous pour le faire fonctionner tout de même.
-> On exécute les commandes suivantes :
"docker-compose restart webapp-1"
"docker exec -it webapp-1 /bin/bash"
"mkdir -p /var/log/webapp-1"
On peut maintenant ajouter un nouvel utilisateur : "User "M Dupond" added to the database".

e. Quelle ligne Lucas pourrait-il rajouter à son Dockerfile pour le résoudre ?
Il pourrait, à son Dockerfile, rajouter la ligne : "RUN mkdir -p /var/log/webapp" (avant CMD/ENTRYPOINT).

f. Remplacez l’image de la webapp par celle de webapp-2.tar et testez que l’ajout
d’utilisateur soit fonctionnel.
-> On charge l'image de la même manière que la webapp-1, avec la commande suivante : "docker load -i webapp-2.tar".
Puis on modifie le fichier 'compose.yaml' en conséquence.
L'ajout d'utilisateur est directement fonctionnel avec 'webapp-2'.


2. Kubernetes

a. Installez et démarrez Minikube ( minikube start --driver=docker ).

b. Monitorez l’état de votre cluster Minikube avec kubectl get all .

c. Utilisez l’utilitaire kompose pour convertir votre compose.yaml en configurations K8s.

d. Appliquez les configurations. Quelles corrections avez-vous dû leur apporter ? Dans
quel ordre les appliquez-vous, et pourquoi ? Utilisez les attributs readinessProbe sur
les pods MongoDB et webapp.

e. Pourquoi la webapp n’a pas été déployée ? Corrigez cela avec minikube docker-env .

f. Paramétrez le déploiement de la webapp pour avoir deux répliques. Testez la
redondance en tuant un des pods de la webapp.