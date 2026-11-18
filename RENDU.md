# Rendu — Atelier Kubernetes

> Complétez ce fichier au fil de l'atelier, puis faites un commit et un push (voir la section « Rendu » du README).

**Nom / Prénom : ROYCE REMINGTON Romen
**Groupe : RS2

---

## Exercice 1 — Cluster

1. Combien de nœuds affiche `kubectl get nodes` ? Quel est le rôle de chacun ?
   > La commande "kubectl get nodes" nous affiche trois noeuds. Nous avons "k3d-lab-server-0" qui est le noeud de contrôle. Et "k3d-lab-agent-0","k3d-lab-agent-1" qui sont les deux workers.

2. Que montre `docker ps` ? Qu'est-ce qu'un « nœud » dans k3d, concrètement ?
   >  "docker ps" nous montre quatre conteneurs.
   Trois correspondent aux nœuds du cluster.  Et le quatrième "k3d-lab-serverlb" sert à rediriger le trafic et ne compte pas comme un nœud.

## Exercice 2 — Premier déploiement

3. Quels objets Kubernetes ont été créés par `kubectl create deployment` ? (indice : `kubectl get all`)
   > La commande a créé le Deployment web. Kubernetes a ensuite créé automatiquement un ReplicaSet et un pod nginx.
4. Vous avez supprimé le pod `web-...` : que s'est-il passé, et pourquoi ?
   > Quand j’ai supprimé le pod qui finissait par ds9x2, Kubernetes l’a remplacé automatiquement par un nouveau pod qui finit par hzn4l. 
   Le nouveau est passé à Running car le déploiement est configuré pour garder un pod en fonctionnement.

## Exercice 3 — Modifier un conteneur en direct

5. Après suppression du pod, votre message « Bienvenue... » a-t-il survécu ? Pourquoi ?
   > Non, mon message a disparu. J’avais modifié la page dans l’ancien pod seulement. Le nouveau pod affiche donc la page par défaut de nginx.
6. Pourquoi le `port-forward` a-t-il cessé de fonctionner après la suppression du pod ?
   > Le port-forward était connecté au pod que j’ai supprimé. Quand ce pod a disparu, le tunnel s’est arrêté. 
   J’ai dû le relancer pour accéder au nouveau pod.

## Exercice 4 — Déclaratif

7. Quelle différence entre `kubectl create` et `kubectl apply` ?
   > "kubectl create" crée une nouvelle ressource et renvoie une erreur si elle existe déjà. 
   "kubectl apply -f" lit un fichier YAML et crée la ressource si elle n’existe pas ou la met à jour si elle existe déjà.
8. À quoi sert le `selector` d'un Service ? Que se passe-t-il s'il ne correspond à aucun pod ?
   > Le selector indique au Service quels pods doivent recevoir le trafic. Ici il sélectionne les pods qui ont l’étiquette "app: site".
   Si aucun pod ne correspond, le Service n’a nulle part où envoyer les requêtes.

## Exercice 5 — Scaling & mises à jour

9. Sur quels nœuds se trouvent vos 3 réplicas ? Collez la sortie de `kubectl get pods -l app=site -o wide`.
   ``
   NAME                   READY   STATUS    RESTARTS   AGE     IP          NODE               NOMINATED NODE   READINESS GATES
site-75775484b-77ztl   1/1     Running   0          7s      10.42.0.5   k3d-lab-agent-0    <none>           <none>
site-75775484b-7vw9c   1/1     Running   0          9m24s   10.42.2.8   k3d-lab-server-0   <none>           <none>
site-75775484b-xsl4q   1/1     Running   0          7s      10.42.1.7   k3d-lab-agent-1    <none>           <none>
   ``

10. Avec l'image `nginx:9.99`, quel statut avait le pod ? Quelle commande vous a donné la cause ?
    > Le nouveau pod était en ImagePullBackOff. La commande kubectl describe pod m’a donné la cause dans la partie Events. L’image nginx:9.99 était introuvable.
11. Pendant la panne, le site restait-il accessible ? Pourquoi ?
    > Dans le cluster, les trois anciens pods sont restés en Running. Ils pouvaient donc continuer à servir le site même si le nouveau pod ne démarrait pas. 
    Dans mon navigateur, j’ai eu une erreur 401 alors que le tunnel port-forward s’était arrêté. Je n’ai donc pas pu vérifier l’accès depuis le navigateur pendant la panne.

## Exercice 6 — Portainer & Mario

12. URL publique de votre Mario :
    > https://musical-bassoon-jjp4x7rjpw62jp5v-8585.app.github.dev/
13. Retrouvez le Deployment `mario` en ligne de commande. Portainer et `kubectl` parlent-ils au même composant ? Lequel ?
    > J’ai créé le Deployment "mario" dans Portainer et je le retrouve avec "kubectl". 
    Les deux parlent au même composant -> le serveur d’API Kubernetes (kube-apiserver).

## Bonus (facultatif)

14. Bonus réalisés et remarques :
    > Bonus A : Juste après la suppression d’un pod, j’ai eu une réponse 502. Et ensuite le site a répondu de nouveau 200 sur le port 8081 sans relancer de tunnel.
    > Bonus B : Avec la sonde sur "/nope", le nouveau pod démarre mais il n’est pas prêt à recevoir des requêtes. En remettant "/" le déploiement fonctionne de nouveau.
    > Bonus C : Après le "rain", le pod qui tournait sur "k3d-lab-agent-0" a été remplacé par un nouveau pod sur "k3d-lab-server-0". 
    Après "uncordon", les pods ne sont pas revenus sur le worker. Donc il peut de nouveau accueillir des pods mais ceux qui tournent déjà ne sont pas déplacés.
    > J’ai déployé les mêmes fichiers dans prod. Les deux déploiements site ont le même nom, mais chacun a ses trois pods dans son propre namespace. J’ai retrouvé default et prod dans Portainer.
---

**Score obtenu avec `./scripts/check.sh` :**  Score : 18/ 18 
