# Boutique Diayma


## 2. Quels sont les projets de la solution ?

La solution Boutique Diayma se compose de deux éléments principaux. Le premier est le fichier de solution `Diayma.sln`, situé à la racine du projet, qui constitue le point d'entrée pour l'environnement de développement Visual Studio et référence l'ensemble des composants de l'application. Le second est le projet `Diayma.csproj`, localisé dans le répertoire `P2FixAnAppDotNetCode/`, qui contient l'intégralité du code source de l'application web ASP.NET Core.

---

## 3. Quelle est la version SDK .NET utilisée par ces projets ?

L'application cible le framework .NET Core dans sa version 2.0, comme nous pouvons le constater dans le fichier de configuration du projet `P2FixAnAppDotNetCode/Diayma.csproj`. Cette version est spécifiée par la balise `TargetFramework` dont la valeur est définie comme suit :

```xml
<TargetFramework>netcoreapp2.0</TargetFramework>
```

Nous notons que cette version du framework, bien que fonctionnelle, n'est plus maintenue par Microsoft.

---

## 4. Installez le SDK

Pour exécuter l'application, il est nécessaire de disposer du SDK .NET Core correspondant. Nous avons utilisé le gestionnaire de paquets Winget pour procéder à l'installation. La commande suivante permet d'installer le runtime ASP.NET Core requis :

```bash
winget install Microsoft.DotNet.AspNetCore.2_2 --accept-source-agreements --accept-package-agreements
```

Une fois l'installation terminée, nous pouvons vérifier que le SDK a été correctement installé en listant les versions disponibles sur le système à l'aide de la commande suivante :

```bash
dotnet --list-sdks
```

Pour démarrer l'application, nous devons d'abord restaurer les dépendances du projet puis lancer l'exécution. Les commandes ci-dessous permettent d'effectuer ces opérations :

```bash
cd P2FixAnAppDotNetCode
dotnet restore
dotnet run
```

L'application sera alors accessible à l'adresse `http://localhost:62929`.

---

## 6. Explorez l'application. Signalez 2 bugs trouvés

Au cours de notre exploration de l'application, nous avons identifié deux anomalies majeures affectant son bon fonctionnement. Nous présentons ci-après une description détaillée de chacune d'entre elles.

### 6.1. Première anomalie : Calcul incorrect des valeurs du panier

Cette anomalie se situe dans le fichier `P2FixAnAppDotNetCode/Models/Cart.cs`, plus précisément aux lignes 66 et 77.

Nous avons constaté que les méthodes `GetTotalValue()` et `GetAverageValue()` de la classe `Cart` ne prennent pas en compte la quantité des produits lors du calcul. Ces deux méthodes effectuent leurs calculs uniquement sur le prix unitaire des produits, ignorant ainsi le nombre d'unités présentes dans le panier.

Pour reproduire cette anomalie, nous avons suivi la procédure suivante. Nous avons d'abord lancé l'application et accédé à la page d'accueil à l'adresse `http://localhost:62929`. Nous avons ensuite ajouté un même produit plusieurs fois au panier, puis nous avons consulté le contenu du panier. Nous avons alors observé que les valeurs affichées pour le total et la moyenne étaient incorrectes.

Le code à l'origine de cette anomalie est le suivant :

```csharp
// Ligne 66 - GetTotalValue()
return GetCartLineList().Sum(x => x.Product.Price);

// Ligne 77 - GetAverageValue()
return GetCartLineList().Average(x => x.Product.Price);
```

---

### 6.2. Seconde anomalie : Dysfonctionnement de la traduction espagnole

Nous avons observé que la traduction anglaise de l'application fonctionne correctement. Cependant, lorsque l'utilisateur sélectionne la langue espagnole, l'interface bascule vers le français au lieu de l'espagnol attendu. Ce comportement correspond au mécanisme de repli (fallback) du système de localisation.

Pour reproduire cette anomalie, nous avons lancé l'application et sélectionné l'option "Spanish" dans le sélecteur de langue. Après avoir validé ce choix, nous avons constaté que l'interface s'affichait en français plutôt qu'en espagnol.

Notre investigation a révélé deux causes principales à cette anomalie. Premièrement, la culture espagnole n'est pas déclarée dans la liste des cultures supportées par l'application. Dans le fichier `Startup.cs`, aux lignes 45 à 52, nous pouvons observer que seules les cultures anglaise et française sont configurées :

```csharp
// Startup.cs - Cultures supportées (lignes 45-52)
var supportedCultures = new List<CultureInfo>
{
    new CultureInfo("en-GB"),
    new CultureInfo("en-US"),
    new CultureInfo("en"),
    new CultureInfo("fr-FR"),
    new CultureInfo("fr"),
    // MANQUANT : new CultureInfo("es"), new CultureInfo("es-ES")
};
```

Deuxièmement, les fichiers de ressources pour la langue espagnole (fichiers portant l'extension `.es.resx`) sont absents du répertoire `Resources/`. Nous notons toutefois que le service `LanguageService.cs` retourne correctement le code "es" pour la sélection espagnole à la ligne 35, ce qui indique que la logique de sélection de langue est fonctionnelle mais ne peut aboutir faute de ressources disponibles.

---

## 7. Placez un point d'arrêt sur les lignes suivantes

Dans le cadre de notre analyse du flux d'exécution, nous avons placé des points d'arrêt aux emplacements indiqués afin de comprendre les mécanismes fondamentaux d'ASP.NET Core.

### a) CartSummaryViewComponent (ligne 12)

Le premier point d'arrêt se situe dans le composant `CartSummaryViewComponent`, à la ligne 12, sur l'instruction `_cart = cart as Cart;`. Ce point d'arrêt permet d'observer le mécanisme d'injection de dépendances (Dependency Injection) en action. Nous pouvons constater que le constructeur reçoit automatiquement une instance de `ICart` (une interface) injectée par le conteneur de services d'ASP.NET Core. Cette observation illustre le principe de l'inversion de contrôle et le découplage entre les composants.

### b) ProductController (ligne 15)

Le deuxième point d'arrêt se trouve dans le contrôleur `ProductController`, à la ligne 15, sur l'instruction `_productService = productService;`. Ce point d'arrêt permet de comprendre le processus d'instanciation des contrôleurs. Nous pouvons observer qu'un nouveau contrôleur est créé pour chaque requête HTTP entrante, et que les services requis sont automatiquement injectés lors de cette instanciation.

### c) OrderController (ligne 17)

Le troisième point d'arrêt est positionné dans le contrôleur `OrderController`, à la ligne 17, sur l'instruction `_cart = pCart;`. Ce point d'arrêt permet de vérifier le partage d'état du panier entre les différents composants de l'application. Nous pouvons observer que la même instance de `Cart`, configurée en tant que Singleton, est injectée dans ce contrôleur, garantissant ainsi la persistance des données du panier tout au long de la session.

### d) CartController (ligne 15)

Le quatrième point d'arrêt se situe dans le contrôleur `CartController`, à la ligne 15, sur l'instruction `_cart = pCart;`. Ce point d'arrêt permet d'approfondir notre compréhension du cycle de vie des services. Nous pouvons observer que le `Cart` est enregistré en tant que Singleton (une seule instance partagée), tandis que le `ProductService` est enregistré en tant que Transient (une nouvelle instance à chaque injection).

### e) Startup (ligne 20)

Le cinquième point d'arrêt est placé dans la classe `Startup`, à la ligne 20, sur l'instruction `Configuration = configuration;`. Ce point d'arrêt permet d'observer le démarrage de l'application. Contrairement aux points d'arrêt précédents qui sont atteints à chaque requête, celui-ci n'est exécuté qu'une seule fois lors du lancement de l'application, ce qui permet de comprendre la phase d'initialisation du framework.

---

## 5. Quels sont les namespaces, classes et méthodes visités avant l'affichage des produits ?

Voici l'ordre des namespaces, classes et méthodes visités lors de l'affichage de la page des produits :

1. `P2FixAnAppDotNetCode.Program.Main()` (F10)
2. `P2FixAnAppDotNetCode.Startup.Startup()` (F10)
3. `P2FixAnAppDotNetCode.Startup.ConfigureServices()` (F11)
4. `P2FixAnAppDotNetCode.Startup.Configure()` (F10)
5. `P2FixAnAppDotNetCode.Controllers.ProductController` — Constructeur (F11)
6. `P2FixAnAppDotNetCode.Models.Services.ProductService` — Constructeur (F11)
7. `P2FixAnAppDotNetCode.Models.Repositories.ProductRepository` — Constructeur (F11)
8. `P2FixAnAppDotNetCode.Models.Repositories.ProductRepository.GenerateProductData()` (F10)
9. `P2FixAnAppDotNetCode.Controllers.ProductController.Index()` (F11)
10. `P2FixAnAppDotNetCode.Models.Services.ProductService.GetAllProducts()` (F11)
11. `P2FixAnAppDotNetCode.Models.Repositories.ProductRepository.GetAllProducts()` (F11)
12. `Views/Product/Index.cshtml` (F10)
13. `Views/Shared/_Layout.cshtml` (F10)
14. `P2FixAnAppDotNetCode.Components.CartSummaryViewComponent.Invoke()` (F10)

| Touche | Mode | Action |
|--------|------|--------|
| **F11** | Pas à pas détaillé | Entre dans la méthode |
| **F10** | Pas à pas principal | Exécute sans entrer |
| **Shift+F11** | Pas à pas sortant | Sort de la méthode |


---

## 6. Déployez votre solution sous forme d'exécutable Windows

Pour déployer l'application sous forme d'exécutable autonome pour Windows, nous utilisons la commande de publication du SDK .NET. Cette commande génère un package contenant l'application et toutes ses dépendances, y compris le runtime .NET, ce qui permet une exécution sans installation préalable du framework sur la machine cible.

La commande de publication est la suivante :

```bash
dotnet publish -c Release -r win-x64 --self-contained true
```

Cette commande utilise trois paramètres essentiels. Le paramètre `-c Release` indique que la compilation doit être effectuée en mode Release, ce qui active les optimisations du compilateur et produit un binaire plus performant. Le paramètre `-r win-x64` spécifie le runtime cible, en l'occurrence Windows 64 bits. Enfin, le paramètre `--self-contained true` indique que le runtime .NET doit être inclus dans le package de déploiement, rendant ainsi l'application totalement autonome.

Une fois la publication terminée, l'exécutable `Diayma.exe` se trouve dans le répertoire suivant :

```
P2FixAnAppDotNetCode\bin\Release\netcoreapp2.0\win-x64\publish\Diayma.exe
```

L'exécutable déployé est disponible sur Google Drive : [Télécharger l'exécutable](https://drive.google.com/drive/folders/1fA2Eu0xoHJIKhAA7BRZaKX6_tz3y3reX?usp=sharing)
