# TP5 – RMI Conversion (Serveur + Client)

Application Java RMI qui convertit un montant (multiplication par 3) via un objet distant.

## Structure des projets

```
TP5_RMIConversion_Server
├── src/main/java
│   ├── rmiService
│   │   ├── IConversion.java        (interface distante)
│   │   └── ConversionImpl.java     (implémentation)
│   └── rmiServer
│       └── ConversionServer.java   (démarre le RMIRegistry et publie l'objet)
└── pom.xml                         (génère aussi un jar "api" contenant IConversion)

TP5_RMIConversion_Client
├── src/main/java
│   └── rmiClient
│       └── ConversionClient.java   (appelle la méthode distante)
└── pom.xml
```

## Prérequis

- JDK compatible avec la version configurée dans les `pom.xml` (`maven.compiler.source/target = 27`).
  Si vous avez un JDK plus ancien (17 ou 21), changez ces deux valeurs en conséquence.
- Maven (ou IntelliJ IDEA avec Maven intégré)

## Principe

Le client a besoin de l'interface `rmiService.IConversion` pour pouvoir caster le stub.
On ne copie pas le code du serveur : on **génère un jar `api`** côté serveur qui contient
uniquement `IConversion.class`, puis on l'ajoute comme dépendance au client.

## Étapes

### 1. Builder le serveur

Dans le projet **TP5_RMIConversion_Server** :

- Avec Maven en ligne de commande :
  ```bash
  mvn clean package
  ```
- Ou avec IntelliJ : onglet **Maven** → *TP5_RMIConversion_Server* → *Lifecycle* → **clean** puis **package**.
  (ou menu **Build → Build Project**)

### 2. Récupérer le fichier jar

Après le build, le jar de l'interface se trouve dans le dossier `target` du serveur :

```
TP5_RMIConversion_Server/target/TP5_RMIConversion_Server-1.0-SNAPSHOT-api.jar
```

Ce jar contient uniquement `rmiService/IConversion.class`.

### 3. Ajouter le jar au client

**Option A – IntelliJ (module)**

1. **File → Project Structure → Modules**
2. Sélectionner le module **TP5_RMIConversion_Client**
3. Onglet **Dependencies** → bouton **+** → **JARs or Directories…**
4. Choisir `TP5_RMIConversion_Server-1.0-SNAPSHOT-api.jar` → **OK** → **Apply**

**Option B – Maven (recommandé)**

Installer le jar dans le dépôt local :

```bash
mvn install:install-file \
  -Dfile=TP5_RMIConversion_Server/target/TP5_RMIConversion_Server-1.0-SNAPSHOT-api.jar \
  -DgroupId=org.example \
  -DartifactId=TP5_RMIConversion_Server \
  -Dversion=1.0-SNAPSHOT \
  -Dclassifier=api \
  -Dpackaging=jar
```

Puis ajouter dans le `pom.xml` du client :

```xml
<dependencies>
    <dependency>
        <groupId>org.example</groupId>
        <artifactId>TP5_RMIConversion_Server</artifactId>
        <version>1.0-SNAPSHOT</version>
        <classifier>api</classifier>
    </dependency>
</dependencies>
```

Recharger Maven (**Reload All Maven Projects**).

### 4. Lancer le serveur

Exécuter `rmiServer.ConversionServer` (la méthode `main` doit être `public static`).

Sortie attendue :

```
Serveur RMI de conversion démarré...
```

Le serveur crée le RMIRegistry sur le port **1099** et publie l'objet sous le nom
`rmi://localhost:1099/BK`.

### 5. Lancer le client

Le serveur doit rester actif. Exécuter `rmiClient.ConversionClient`.

Sortie attendue :

```
Résultat de la conversion : 600.0
```

(200 × 3 = 600)

## Dépannage

| Problème | Cause probable | Solution |
|---|---|---|
| `ClassNotFoundException: rmiService.IConversion` côté client | Le jar `api` n'est pas ajouté au client | Refaire l'étape 3 |
| `ConnectException: Connection refused` | Le serveur n'est pas lancé | Lancer le serveur avant le client |
| `ExportException: Port already in use` | Port 1099 déjà utilisé (ancien serveur encore actif) | Arrêter l'ancien processus |
| `NotBoundException` | Le nom de lookup diffère du nom du `rebind` | Utiliser `BK` des deux côtés |
| `release version 27 not supported` | JDK trop ancien | Adapter `maven.compiler.source/target` à votre JDK |
| Le jar `api` n'apparaît pas | Le build `package` n'a pas été exécuté | Lancer `mvn clean package` |

## Exécuter sur deux machines

Remplacer `localhost` dans le client par l'adresse IP du serveur, et lancer le serveur avec :

```bash
java -Djava.rmi.server.hostname=<IP_DU_SERVEUR> ...
```

Vérifier que le port 1099 est ouvert dans le pare-feu.