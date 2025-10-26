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
-> Il pourrait, à son Dockerfile, rajouter la ligne : "RUN mkdir -p /var/log/webapp" (avant CMD/ENTRYPOINT).

f. Remplacez l’image de la webapp par celle de webapp-2.tar et testez que l’ajout
d’utilisateur soit fonctionnel.
-> On charge l'image de la même manière que la webapp-1, avec la commande suivante : "docker load -i webapp-2.tar".
Puis on modifie le fichier 'compose.yaml' en conséquence.
L'ajout d'utilisateur est directement fonctionnel avec 'webapp-2'.


2. Kubernetes

a. Installez et démarrez Minikube ( minikube start --driver=docker ).
-> On effectue la commande donnée : "minikube start --driver=docker".

b. Monitorez l’état de votre cluster Minikube avec kubectl get all .
-> On effectue la commande donnée : "kubectl get all".

c. Utilisez l’utilitaire kompose pour convertir votre compose.yaml en configurations K8s.
-> On utilise l'utilitaire kompose via la commande suivante "kompose convert -f compose.yaml" afin d'obtenir nos configurations K8s.

d. Appliquez les configurations. Quelles corrections avez-vous dû leur apporter ?
-> Dans le fichier 'webapp-2-deployment.yaml', j'ai ajouté : "imagePullPolicy: IfNotPresent".
L'image webapp-2 est locale, n'existant pas sur Docker Hub (Internet). Grâce à cette correction, Kubernetes ne tente plus de télécharger l'image depuis Internet en premier, il va d'abord chercher si elle existe en local.
-> Dans le fichier 'webapp-2-deployment.yaml', ajout de la partie "readinessProbe".
-> Dans le fichier 'mongodb-deployment.yaml', ajout de la partie "readinessProbe".

Dans quel ordre les appliquez-vous, et pourquoi ? Utilisez les attributs readinessProbe sur les pods MongoDB et webapp.
-> Je les applique dans l'ordre suivant :
1) "kubectl apply -f mongodb-data-persistentvolumeclaim.yaml" : Une demande de stockage.
Le déploiement MongoDB va essayer de monter ce volume : il doit donc exister avant le déploiement MongoDB.
2) "kubectl apply -f mongodb-deployment.yaml" : Déploiement de MongoDB.
3) "kubectl apply -f  webapp-2-deployment.yaml" : Déploiement de Webapp.
Webapp-2 dépendant de MongoDB, doit être réaliser après le déploiement de ce dernier.

e. Pourquoi la webapp n’a pas été déployée ? Corrigez cela avec minikube docker-env .
-> Elle n'a pas été déployée parce qu'elle n'existait tout simmplement pas dans le Docker dans l'environnement de Minikube. (Elle n'a, pour l'instant, été chargée que dans notre Docker local. Hors, Minikube utilise son propre démon Docker, il nous faut donc la charger dans son environnement pour y avoir accès.)

On va donc effectuer les commandes suivantes afin de la charger :
"minikube docker-env"
"eval $(minikube -p minikube docker-env)"
"docker load -i ../webapp-2.tar"

f. Paramétrez le déploiement de la webapp pour avoir deux répliques. Testez la
redondance en tuant un des pods de la webapp.
-> Dans le fichier 'webapp-2-deployment.yaml', on passe la valeur de "replicas" dans "spec" à  2.
On voit que le second pod prend bien le relais du premier à sa mort, le temps qu'un nouveau pod vienne remplacer le premier. Le but étant d'assurer une haute disponibilité, même si un pod "crash".