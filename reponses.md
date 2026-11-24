TP1
Partie B — Contract-first : réponses aux questions
Les réponses ci-dessous concernent uniquement la Partie B et les comparaisons demandées avec la Partie A. Les observations qui dépendent d'un test SoapUI doivent être confirmées avec les réponses réellement obtenues.

A. Questions sur le WSDL et les classes générées
1. Dans ce WSDL, retrouvez l'opération, son message de fault et l'action SOAP à envoyer.
- Opération : emprunterLivre
- Message de fault : LivreIndisponibleFault
- Élément XML du fault : LivreIndisponible
- Action SOAP à envoyer : http://supnum.mr/bib/emprunterLivre
Dans le WSDL, l'opération se trouve dans EmpruntPortType. Le fault est déclaré dans wsdl:fault et l'action est indiquée dans soap:operation.
2. Quelle est la signature de emprunterLivre dans EmpruntPortType ?
La méthode générée reçoit deux paramètres de type String et peut déclencher LivreIndisponibleFault :
public EmpruntType emprunterLivre(String idAdherent, String isbn)
        throws LivreIndisponibleFault;
Elle retourne un objet EmpruntType.
3. Quel type Java a été généré pour dateRetour ? Comparez avec la partie A.
Dans la classe générée EmpruntType, le champ dateRetour, défini en xs:date dans le WSDL, est représenté par le type Java XMLGregorianCalendar.

Dans la Partie A, il faut comparer avec le type réellement déclaré dans Emprunt.java. La génération wsimport de la Partie B utilise ici XMLGregorianCalendar pour représenter xs:date.
4. Quel est le nom de la classe d'exception générée, et d'où vient ce nom dans le WSDL ?
La classe générée est LivreIndisponibleFault.

Elle correspond au fault déclaré dans l'opération WSDL. Le WSDL contient un message nommé LivreIndisponibleFault et un élément de données LivreIndisponible. La classe d'exception transporte les données structurées du fault.
B. Questions sur le WSDL publié, le XML de test et la validation
1. Le WSDL publié est-il celui que vous avez fourni ? Contient-il le motif de l'ISBN et le type xs:date ?
Avec wsdlLocation configuré, le service utilise le WSDL fourni comme contrat. Le schéma original contient :
- le type personnalisé IsbnType, avec une restriction pattern ;
- xs:date pour dateRetour et retourPrevu.
Après suppression de wsdlLocation, le WSDL publié est généré automatiquement par Metro. Dans le XSD publié observé, le motif IsbnType n'apparaît plus : les champs isbn sont déclarés en xs:string.

En revanche, dateRetour et retourPrevu restent de type xs:date.
2. Avez-vous dû modifier 01-emprunter.xml pour qu'il fonctionne ici ? Pourquoi ?
Le fichier de requête doit respecter exactement le contrat de la Partie B : l'élément racine est emprunterLivre, dans le namespace http://supnum.mr/bib, et il contient les éléments idAdherent et isbn.

Si le fichier 01-emprunter.xml de la Partie A utilisait une autre structure, un autre namespace ou d'autres noms d'éléments, il fallait l'adapter au WSDL de la Partie B. Le format de la requête SOAP dépend du contrat utilisé.

Exemple du contenu attendu dans le Body :
<emprunterLivre xmlns="http://supnum.mr/bib">
    <idAdherent>A123</idAdherent>
    <isbn>978-2-10-074444-5</isbn>
</emprunterLivre>
3. Sans @SchemaValidation, l'ISBN 978-2-10 est-il refusé ? Et avec ? Quelle Fault obtenez-vous ?
Sans @SchemaValidation, le contrôle du motif XSD n'est pas effectué automatiquement par la validation du schéma. Comme le paramètre Java est une chaîne (String), la valeur peut atteindre la méthode si aucun contrôle métier supplémentaire ne la refuse.

Avec @SchemaValidation et le WSDL/XSD original contenant IsbnType, la valeur 978-2-10 ne respecte pas le motif ISBN et est rejetée pendant la validation du message SOAP. Le test rapporté a donné une erreur de validation XSD, de type S:Client, avec le message cvc-pattern-valid.

Attention : après suppression de wsdlLocation, le XSD publié observé ne contient plus le motif ISBN. Il ne faut donc pas affirmer que le même rejet se produit dans cette configuration sans refaire le test.
4. Retirez wsdlLocation de l'annotation et redémarrez : qu'est-ce qui change dans le WSDL publié ?
Après la suppression de wsdlLocation, Metro génère automatiquement le WSDL à partir de l'interface Java et de ses annotations.

Les éléments principaux restent présents : targetNamespace, service, port, portType, operation et soapAction.

Les différences observées sont notamment :
- le binding devient EmpruntPortBinding au lieu de EmpruntSoapBinding ;
- le message de requête s'appelle emprunterLivre au lieu de emprunterLivreRequest ;
- le fault publié porte le nom LivreIndisponibleFault ;
- le WSDL importe le XSD depuis ?xsd=1, au lieu d'inclure directement le schéma ;
- le type personnalisé IsbnType et son motif ne figurent plus dans le XSD publié ;
- les champs de date restent en xs:date.
C. Rendu
1. Citez deux différences que vous avez observées entre le WSDL de la Partie A et celui de la Partie B.
Deux différences principales sont :
1. La Partie A utilise l'approche Code-first : le service et son contrat sont décrits à partir des classes Java. La Partie B utilise l'approche Contract-first : le WSDL est défini avant l'implémentation Java.
2. En Partie B, les classes Java sont générées à partir du WSDL avec wsimport, alors qu'en Partie A le contrat WSDL est produit à partir du code Java.
On peut aussi observer des différences dans les noms des messages, des bindings et dans la manière de déclarer les Faults, selon les WSDL publiés.
2. Si on renomme le champ dateRetour dans le code, quel projet voit son contrat changer ? Quel risque pour les clients ?
Dans le projet Code-first (Partie A), le contrat WSDL est généré à partir du code Java. Renommer le champ peut donc modifier le WSDL publié et changer le contrat visible par les clients.

Dans le projet Contract-first (Partie B), le contrat de référence reste le WSDL fourni. Il ne faut pas modifier manuellement les classes générées : elles doivent rester conformes au contrat. Un changement du contrat doit être fait explicitement dans le WSDL, puis les classes doivent être régénérées.

Le risque est de casser la compatibilité avec les clients existants : leurs requêtes ou leurs réponses peuvent ne plus correspondre au format attendu. Ils devront alors adapter leur code.
3. Quelle approche choisiriez-vous pour le service Paiement, appelé par Scolarité (.NET), Bibliothèque (PHP) et le portail (Node.js) ? Et pour un prototype interne de deux jours ?
Pour le service Paiement, je choisirais Contract-first. Plusieurs applications développées avec des technologies différentes doivent communiquer avec le même service. Un WSDL défini à l'avance donne un contrat commun et stable, indépendant du langage utilisé. Chaque équipe peut générer ou développer son client à partir de ce contrat.

Pour un prototype interne de deux jours, je choisirais plutôt Code-first, car cette approche permet de démarrer rapidement à partir du code Java et demande moins de préparation du contrat.
Conclusion
La Partie B montre que l'approche Contract-first place le WSDL au centre du développement. wsimport génère les classes Java à partir de ce contrat, puis l'implémentation réalise les opérations définies.

La comparaison après suppression de wsdlLocation montre que Metro peut reconstruire un WSDL à partir du code Java, mais que certaines informations du contrat original, comme la restriction pattern du type IsbnType, peuvent disparaître. Pour un service utilisé par plusieurs applications et plusieurs langages, il est donc important de maîtriser et de préserver le contrat partagé.
À vérifier avant le rendu :
- Comparer le type de dateRetour avec la déclaration exacte dans Emprunt.java de la Partie A.
- Confirmer si 01-emprunter.xml a réellement été modifié pendant vos essais.
- Ajouter, si demandé par l'enseignant, les captures SoapUI des tests avec et sans @SchemaValidation.