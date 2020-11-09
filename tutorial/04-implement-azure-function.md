---
ms.openlocfilehash: 728138b5d298f9469a566defed6ee2e457896c0b
ms.sourcegitcommit: 57a5c2e1a562d8af092a3e78786d711ce1e8f9cb
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 10/30/2020
ms.locfileid: "48822644"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="f74a4-101">In dieser Übung schließen Sie die Implementierung der Azure-Funktion ab `GetMyNewestMessage` und aktualisieren den Testclient, um die Funktion aufzurufen.</span><span class="sxs-lookup"><span data-stu-id="f74a4-101">In this exercise you will finish implementing the Azure Function `GetMyNewestMessage` and update the test client to call the function.</span></span>

<span data-ttu-id="f74a4-102">Die Azure-Funktion verwendet den [im-Auftrag-von-Fluss](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow).</span><span class="sxs-lookup"><span data-stu-id="f74a4-102">The Azure Function uses the [on-behalf-of flow](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow).</span></span> <span data-ttu-id="f74a4-103">Die grundlegende Reihenfolge der Ereignisse in diesem Fluss sind:</span><span class="sxs-lookup"><span data-stu-id="f74a4-103">The basic order of events in this flow are:</span></span>

- <span data-ttu-id="f74a4-104">Die Testanwendung verwendet einen interaktiven Authentifizierungs Fluss, um dem Benutzer die Möglichkeit zu geben, sich anzumelden und die Zustimmung zu erteilen.</span><span class="sxs-lookup"><span data-stu-id="f74a4-104">The test application uses an interactive auth flow to allow the user to sign in and grant consent.</span></span> <span data-ttu-id="f74a4-105">Es ruft ein Token zurück, das auf die Azure-Funktion ausgelegt ist.</span><span class="sxs-lookup"><span data-stu-id="f74a4-105">It gets back a token that is scoped to the Azure Function.</span></span> <span data-ttu-id="f74a4-106">Das **Token enthält keine** Microsoft Graph-Bereiche.</span><span class="sxs-lookup"><span data-stu-id="f74a4-106">The token does **NOT** contain any Microsoft Graph scopes.</span></span>
- <span data-ttu-id="f74a4-107">Die Testanwendung Ruft die Azure-Funktion auf, wobei ihr Zugriffstoken in der Kopfzeile gesendet wird `Authorization` .</span><span class="sxs-lookup"><span data-stu-id="f74a4-107">The test application invokes the Azure Function, sending its access token in the `Authorization` header.</span></span>
- <span data-ttu-id="f74a4-108">Die Azure-Funktion überprüft das Token und tauscht dann dieses Token für ein zweites Zugriffstoken aus, das Microsoft Graph-Bereiche enthält.</span><span class="sxs-lookup"><span data-stu-id="f74a4-108">The Azure Function validates the token, then exchanges that token for a second access token that contains Microsoft Graph scopes.</span></span>
- <span data-ttu-id="f74a4-109">Die Azure-Funktion ruft Microsoft Graph im Namen des Benutzers mithilfe des zweiten Zugriffstokens auf.</span><span class="sxs-lookup"><span data-stu-id="f74a4-109">The Azure Function calls Microsoft Graph on the user's behalf using the second access token.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="f74a4-110">Um zu vermeiden, dass die Anwendungs-ID und der geheime Schlüssel in der Quelle gespeichert werden, verwenden Sie den [.net Secret Manager](https://docs.microsoft.com/aspnet/core/security/app-secrets) zum Speichern dieser Werte.</span><span class="sxs-lookup"><span data-stu-id="f74a4-110">To avoid storing the application ID and secret in source, you will use the [.NET Secret Manager](https://docs.microsoft.com/aspnet/core/security/app-secrets) to store these values.</span></span> <span data-ttu-id="f74a4-111">Der geheime Manager dient nur Entwicklungszwecken, für Produktions-apps sollte ein vertrauenswürdiger geheimer geheim Manager zum Speichern von Geheimnissen verwendet werden.</span><span class="sxs-lookup"><span data-stu-id="f74a4-111">The Secret Manager is for development purposes only, production apps should use a trusted secret manager for storing secrets.</span></span>

## <a name="add-authentication-to-the-single-page-application"></a><span data-ttu-id="f74a4-112">Hinzufügen der Authentifizierung zur Anwendung für einzelne Seiten</span><span class="sxs-lookup"><span data-stu-id="f74a4-112">Add authentication to the single page application</span></span>

<span data-ttu-id="f74a4-113">Beginnen Sie mit dem Hinzufügen der Authentifizierung zum Spa.</span><span class="sxs-lookup"><span data-stu-id="f74a4-113">Start by adding authentication to the SPA.</span></span> <span data-ttu-id="f74a4-114">Dadurch kann die Anwendung ein Zugriffstoken abrufen, das den Zugriff zum Aufrufen der Azure-Funktion gewährt.</span><span class="sxs-lookup"><span data-stu-id="f74a4-114">This will allow the application to get an access token granting access to call the Azure Function.</span></span> <span data-ttu-id="f74a4-115">Da es sich hierbei um ein Spa handelt, wird der [Autorisierungscode Fluss mit PKCE](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-auth-code-flow)verwendet.</span><span class="sxs-lookup"><span data-stu-id="f74a4-115">Because this is a SPA, it will use the [authorization code flow with PKCE](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-auth-code-flow).</span></span>

1. <span data-ttu-id="f74a4-116">Erstellen Sie eine neue Datei im **Testclient** -Verzeichnis mit dem Namen **config.js** , und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="f74a4-116">Create a new file in the **TestClient** directory named **config.js** and add the following code.</span></span>

    :::code language="javascript" source="../demo/TestClient/config.example.js" id="msalConfigSnippet":::

    <span data-ttu-id="f74a4-117">Ersetzen `YOUR_TEST_APP_APP_ID_HERE` Sie durch die Anwendungs-ID, die Sie im Azure-Portal für die **Graph Azure Function Test-App** erstellt haben.</span><span class="sxs-lookup"><span data-stu-id="f74a4-117">Replace `YOUR_TEST_APP_APP_ID_HERE` with the application ID you created in the Azure portal for the **Graph Azure Function Test App**.</span></span> <span data-ttu-id="f74a4-118">Ersetzen `YOUR_TENANT_ID_HERE` Sie durch den **Verzeichnis Wert (Mandanten)** , den Sie aus dem Azure-Portal kopiert haben.</span><span class="sxs-lookup"><span data-stu-id="f74a4-118">Replace `YOUR_TENANT_ID_HERE` with the **Directory (tenant) ID** value you copied from the Azure portal.</span></span> <span data-ttu-id="f74a4-119">Ersetzen Sie `YOUR_AZURE_FUNCTION_APP_ID_HERE` durch die Anwendungs-ID für die **Graph-Azure-Funktion**.</span><span class="sxs-lookup"><span data-stu-id="f74a4-119">Replace `YOUR_AZURE_FUNCTION_APP_ID_HERE` with the application ID for the **Graph Azure Function**.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="f74a4-120">Wenn Sie die Quellcodeverwaltung wie git verwenden, wäre es nun ein guter Zeitpunkt, die **config.js** Datei aus der Quellcodeverwaltung auszuschließen, um zu vermeiden, dass versehentlich Ihre APP-IDs und die Mandanten-ID verloren gehen.</span><span class="sxs-lookup"><span data-stu-id="f74a4-120">If you're using source control such as git, now would be a good time to exclude the **config.js** file from source control to avoid inadvertently leaking your app IDs and tenant ID.</span></span>

1. <span data-ttu-id="f74a4-121">Erstellen Sie eine neue Datei im **Testclient** -Verzeichnis mit dem Namen **auth.js** , und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="f74a4-121">Create a new file in the **TestClient** directory named **auth.js** and add the following code.</span></span>

    :::code language="javascript" source="../demo/TestClient/auth.js" id="signInSignOutSnippet":::

    <span data-ttu-id="f74a4-122">Überprüfen Sie die Funktionsweise dieses Codes.</span><span class="sxs-lookup"><span data-stu-id="f74a4-122">Consider what this code does.</span></span>

    - <span data-ttu-id="f74a4-123">Es initialisiert a `PublicClientApplication` mithilfe der in **config.js** gespeicherten Werte.</span><span class="sxs-lookup"><span data-stu-id="f74a4-123">It initializes a `PublicClientApplication` using the values stored in **config.js**.</span></span>
    - <span data-ttu-id="f74a4-124">Er verwendet `loginPopup` , um den Benutzer unter Verwendung des Berechtigungs Bereichs für die Azure-Funktion zu signieren.</span><span class="sxs-lookup"><span data-stu-id="f74a4-124">It uses `loginPopup` to sign the user in, using the permission scope for the Azure Function.</span></span>
    - <span data-ttu-id="f74a4-125">Der Benutzername wird in der Sitzung gespeichert.</span><span class="sxs-lookup"><span data-stu-id="f74a4-125">It stores the user's username in the session.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="f74a4-126">Da die APP verwendet `loginPopup` wird, müssen Sie möglicherweise den Popupblocker Ihres Browsers ändern, um Popups von zu ermöglichen `http://localhost:8080` .</span><span class="sxs-lookup"><span data-stu-id="f74a4-126">Since the app uses `loginPopup`, you may need to change your browser's pop-up blocker to allow pop-ups from `http://localhost:8080`.</span></span>

1. <span data-ttu-id="f74a4-127">Aktualisieren Sie die Seite, und melden Sie sich an.</span><span class="sxs-lookup"><span data-stu-id="f74a4-127">Refresh the page and sign in.</span></span> <span data-ttu-id="f74a4-128">Die Seite sollte mit dem Benutzernamen aktualisiert werden, was bedeutet, dass das Anmelden erfolgreich war.</span><span class="sxs-lookup"><span data-stu-id="f74a4-128">The page should update with the user name, indicating that the sign in was successful.</span></span>

> [!TIP]
> <span data-ttu-id="f74a4-129">Sie können das Zugriffstoken analysieren [https://jwt.ms](https://jwt.ms) und bestätigen, dass der `aud` Anspruch die APP-ID für die Azure-Funktion ist und dass der `scp` Anspruch den Berechtigungsbereich der Azure-Funktion enthält, nicht Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="f74a4-129">You can parse the access token at [https://jwt.ms](https://jwt.ms) and confirm that the `aud` claim is the app ID for the Azure Function, and that the `scp` claim contains the Azure Function's permission scope, not Microsoft Graph.</span></span>

## <a name="add-authentication-to-the-azure-function"></a><span data-ttu-id="f74a4-130">Hinzufügen von Authentifizierung zur Azure-Funktion</span><span class="sxs-lookup"><span data-stu-id="f74a4-130">Add authentication to the Azure Function</span></span>

<span data-ttu-id="f74a4-131">In diesem Abschnitt implementieren Sie den im-Auftrag-von-Ablauf in der `GetMyNewestMessage` Azure-Funktion, um ein mit Microsoft Graph kompatibles Zugriffstoken abzurufen.</span><span class="sxs-lookup"><span data-stu-id="f74a4-131">In this section you'll implement the on-behalf-of flow in the `GetMyNewestMessage` Azure Function to get an access token compatible with Microsoft Graph.</span></span>

1. <span data-ttu-id="f74a4-132">Initialisieren Sie den geheimen .net-entwicklungsspeicher, indem Sie die CLI in dem Verzeichnis öffnen, das **GraphTutorial. csproj** enthält, und führen Sie den folgenden Befehl aus.</span><span class="sxs-lookup"><span data-stu-id="f74a4-132">Initialize the .NET development secret store by opening your CLI in the directory that contains **GraphTutorial.csproj** and running the following command.</span></span>

    ```Shell
    dotnet user-secrets init
    ```

1. <span data-ttu-id="f74a4-133">Fügen Sie die Anwendungs-ID, die geheime und die Mandanten-ID dem geheimen Speicher mit den folgenden Befehlen hinzu.</span><span class="sxs-lookup"><span data-stu-id="f74a4-133">Add your application ID, secret, and tenant ID to the secret store using the following commands.</span></span> <span data-ttu-id="f74a4-134">Ersetzen Sie `YOUR_API_FUNCTION_APP_ID_HERE` durch die Anwendungs-ID für die **Graph-Azure-Funktion**.</span><span class="sxs-lookup"><span data-stu-id="f74a4-134">Replace `YOUR_API_FUNCTION_APP_ID_HERE` with the application ID for the **Graph Azure Function**.</span></span> <span data-ttu-id="f74a4-135">Ersetzen `YOUR_API_FUNCTION_APP_SECRET_HERE` Sie durch den Anwendungsschlüssel, den Sie im Azure-Portal für die **Graph-Azure-Funktion** erstellt haben.</span><span class="sxs-lookup"><span data-stu-id="f74a4-135">Replace `YOUR_API_FUNCTION_APP_SECRET_HERE` with the application secret you created in the Azure portal for the **Graph Azure Function**.</span></span> <span data-ttu-id="f74a4-136">Ersetzen `YOUR_TENANT_ID_HERE` Sie durch den **Verzeichnis Wert (Mandanten)** , den Sie aus dem Azure-Portal kopiert haben.</span><span class="sxs-lookup"><span data-stu-id="f74a4-136">Replace `YOUR_TENANT_ID_HERE` with the **Directory (tenant) ID** value you copied from the Azure portal.</span></span>

    ```Shell
    dotnet user-secrets set apiFunctionId "YOUR_API_FUNCTION_APP_ID_HERE"
    dotnet user-secrets set apiFunctionSecret "YOUR_API_FUNCTION_APP_SECRET_HERE"
    dotnet user-secrets set tenantId "YOUR_TENANT_ID_HERE"
    ```

### <a name="process-the-incoming-bearer-token"></a><span data-ttu-id="f74a4-137">Verarbeiten des eingehenden Bearer-Tokens</span><span class="sxs-lookup"><span data-stu-id="f74a4-137">Process the incoming bearer token</span></span>

<span data-ttu-id="f74a4-138">In diesem Abschnitt implementieren Sie eine Klasse zum Validieren und Verarbeiten des Bearer-Tokens, das vom Spa an die Azure-Funktion gesendet wurde.</span><span class="sxs-lookup"><span data-stu-id="f74a4-138">In this section you'll implement a class to validate and process the bearer token sent from the SPA to the Azure Function.</span></span>

1. <span data-ttu-id="f74a4-139">Erstellen Sie ein neues Verzeichnis im **GraphTutorial** -Verzeichnis mit dem Namen **Authentication**.</span><span class="sxs-lookup"><span data-stu-id="f74a4-139">Create a new directory in the **GraphTutorial** directory named **Authentication**.</span></span>

1. <span data-ttu-id="f74a4-140">Erstellen Sie im Ordner **./GraphTutorial/Authentication** eine neue Datei mit dem Namen **TokenValidationResult.cs** , und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="f74a4-140">Create a new file named **TokenValidationResult.cs** in the **./GraphTutorial/Authentication** folder, and add the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/TokenValidationResult.cs" id="TokenValidationResultSnippet":::

1. <span data-ttu-id="f74a4-141">Erstellen Sie im Ordner **./GraphTutorial/Authentication** eine neue Datei mit dem Namen **TokenValidation.cs** , und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="f74a4-141">Create a new file named **TokenValidation.cs** in the **./GraphTutorial/Authentication** folder, and add the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/TokenValidation.cs" id="TokenValidationSnippet":::

<span data-ttu-id="f74a4-142">Überprüfen Sie die Funktionsweise dieses Codes.</span><span class="sxs-lookup"><span data-stu-id="f74a4-142">Consider what this code does.</span></span>

- <span data-ttu-id="f74a4-143">Sie stellen sicher, dass in der Kopfzeile ein Bearer-Token vorhanden ist `Authorization` .</span><span class="sxs-lookup"><span data-stu-id="f74a4-143">It ensure there is a bearer token in the `Authorization` header.</span></span>
- <span data-ttu-id="f74a4-144">Die Signatur und der Aussteller werden anhand der veröffentlichten OpenID-Konfiguration von Azure überprüft.</span><span class="sxs-lookup"><span data-stu-id="f74a4-144">It verifies the signature and issuer from Azure's published OpenID configuration.</span></span>
- <span data-ttu-id="f74a4-145">Er überprüft, ob die Benutzergruppe ( `aud` Forderung) mit der Anwendungs-ID der Azure-Funktion übereinstimmt.</span><span class="sxs-lookup"><span data-stu-id="f74a4-145">It verifies that the audience (`aud` claim) matches the Azure Function's application ID.</span></span>
- <span data-ttu-id="f74a4-146">Es analysiert das Token und generiert eine MSAL-Konto-ID, die benötigt wird, um die Token-Zwischenspeicherung zu nutzen.</span><span class="sxs-lookup"><span data-stu-id="f74a4-146">It parses the token and generates an MSAL account ID, which will be needed to take advantage of token caching.</span></span>

### <a name="create-an-on-behalf-of-authentication-provider"></a><span data-ttu-id="f74a4-147">Erstellen eines Authentifizierungsanbieters im Auftrag von</span><span class="sxs-lookup"><span data-stu-id="f74a4-147">Create an on-behalf-of authentication provider</span></span>

1. <span data-ttu-id="f74a4-148">Erstellen Sie eine neue Datei im **Authentifizierungs** Verzeichnis mit dem Namen **OnBehalfOfAuthProvider.cs** , und fügen Sie der Datei den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="f74a4-148">Create a new file in the **Authentication** directory named **OnBehalfOfAuthProvider.cs** and add the following code to that file.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/OnBehalfOfAuthProvider.cs" id="AuthProviderSnippet":::

<span data-ttu-id="f74a4-149">Nehmen Sie sich einen Moment Zeit, um zu prüfen, was der Code in **OnBehalfOfAuthProvider.cs** macht.</span><span class="sxs-lookup"><span data-stu-id="f74a4-149">Take a moment to consider what the code in **OnBehalfOfAuthProvider.cs** does.</span></span>

- <span data-ttu-id="f74a4-150">In der `GetAccessToken` -Funktion wird zunächst versucht, ein Benutzertoken aus dem Token-Cache mithilfe von zu erhalten `AcquireTokenSilent` .</span><span class="sxs-lookup"><span data-stu-id="f74a4-150">In the `GetAccessToken` function, it first attempts to get a user token from the token cache using `AcquireTokenSilent`.</span></span> <span data-ttu-id="f74a4-151">Wenn dies fehlschlägt, verwendet es das Bearer-Token, das von der Test-App an die Azure-Funktion gesendet wird, um eine Benutzer Assertion zu generieren.</span><span class="sxs-lookup"><span data-stu-id="f74a4-151">If this fails, it uses the bearer token sent by the test app to the Azure Function to generate a user assertion.</span></span> <span data-ttu-id="f74a4-152">Anschließend wird diese Benutzer Assertion verwendet, um ein Graph-kompatibles Token mithilfe von zu erhalten `AcquireTokenOnBehalfOf` .</span><span class="sxs-lookup"><span data-stu-id="f74a4-152">It then uses that user assertion to get a Graph-compatible token using `AcquireTokenOnBehalfOf`.</span></span>
- <span data-ttu-id="f74a4-153">Die `Microsoft.Graph.IAuthenticationProvider` Schnittstelle wird implementiert, sodass diese Klasse im Konstruktor von The übergeben werden kann, `GraphServiceClient` um ausgehende Anforderungen zu authentifizieren.</span><span class="sxs-lookup"><span data-stu-id="f74a4-153">It implements the `Microsoft.Graph.IAuthenticationProvider` interface, allowing this class to be passed in the constructor of the `GraphServiceClient` to authenticate outgoing requests.</span></span>

### <a name="implement-a-graph-client-service"></a><span data-ttu-id="f74a4-154">Implementieren eines Graph-Clientdiensts</span><span class="sxs-lookup"><span data-stu-id="f74a4-154">Implement a Graph client service</span></span>

<span data-ttu-id="f74a4-155">In diesem Abschnitt implementieren Sie einen Dienst, der für die [Abhängigkeitsinjektion](https://docs.microsoft.com/azure/azure-functions/functions-dotnet-dependency-injection)registriert werden kann.</span><span class="sxs-lookup"><span data-stu-id="f74a4-155">In this section you'll implement a service that can be registered for [dependency injection](https://docs.microsoft.com/azure/azure-functions/functions-dotnet-dependency-injection).</span></span> <span data-ttu-id="f74a4-156">Der Dienst wird verwendet, um einen authentifizierten Graph-Client abzurufen.</span><span class="sxs-lookup"><span data-stu-id="f74a4-156">The service will be used to get an authenticated Graph client.</span></span>

1. <span data-ttu-id="f74a4-157">Erstellen Sie ein neues Verzeichnis im **GraphTutorial** -Verzeichnis mit dem Namen **Services**.</span><span class="sxs-lookup"><span data-stu-id="f74a4-157">Create a new directory in the **GraphTutorial** directory named **Services**.</span></span>

1. <span data-ttu-id="f74a4-158">Erstellen Sie eine neue Datei im Verzeichnis **Dienste** mit dem Namen **IGraphClientService.cs** , und fügen Sie der Datei den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="f74a4-158">Create a new file in the **Services** directory named **IGraphClientService.cs** and add the following code to that file.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Services/IGraphClientService.cs" id="IGraphClientServiceSnippet":::

1. <span data-ttu-id="f74a4-159">Erstellen Sie eine neue Datei im Verzeichnis **Dienste** mit dem Namen **GraphClientService.cs** , und fügen Sie der Datei den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="f74a4-159">Create a new file in the **Services** directory named **GraphClientService.cs** and add the following code to that file.</span></span>

    ```csharp
    using GraphTutorial.Authentication;
    using Microsoft.Extensions.Configuration;
    using Microsoft.Extensions.Logging;
    using Microsoft.Identity.Client;
    using Microsoft.Graph;

    namespace GraphTutorial.Services
    {
        // Service added via dependency injection
        // Used to get an authenticated Graph client
        public class GraphClientService : IGraphClientService
        {
        }
    }
    ```

1. <span data-ttu-id="f74a4-160">Fügen Sie der Klasse die folgenden Eigenschaften hinzu `GraphClientService` .</span><span class="sxs-lookup"><span data-stu-id="f74a4-160">Add the following properties to the `GraphClientService` class.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Services/GraphClientService.cs" id="UserGraphClientMembers":::

1. <span data-ttu-id="f74a4-161">Fügen Sie der-Klasse die folgenden Funktionen hinzu `GraphClientService` .</span><span class="sxs-lookup"><span data-stu-id="f74a4-161">Add the following functions to the `GraphClientService` class.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Services/GraphClientService.cs" id="UserGraphClientFunctions":::

1. <span data-ttu-id="f74a4-162">Fügen Sie eine Platzhalterimplementierung für die `GetAppGraphClient` Funktion hinzu.</span><span class="sxs-lookup"><span data-stu-id="f74a4-162">Add a placeholder implementation for the `GetAppGraphClient` function.</span></span> <span data-ttu-id="f74a4-163">Sie werden dies in späteren Abschnitten implementieren.</span><span class="sxs-lookup"><span data-stu-id="f74a4-163">You will implement that in later sections.</span></span>

    ```csharp
    public GraphServiceClient GetAppGraphClient()
    {
        throw new System.NotImplementedException();
    }
    ```

    <span data-ttu-id="f74a4-164">Die `GetUserGraphClient` -Funktion verwendet die Ergebnisse der Tokenprüfung und erstellt eine authentifizierte `GraphServiceClient` für den Benutzer.</span><span class="sxs-lookup"><span data-stu-id="f74a4-164">The `GetUserGraphClient` function takes the results of token validation and builds an authenticated `GraphServiceClient` for the user.</span></span>

1. <span data-ttu-id="f74a4-165">Erstellen Sie eine neue Datei im **GraphTutorial** -Verzeichnis mit dem Namen **Startup.cs** , und fügen Sie der Datei den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="f74a4-165">Create a new file in the **GraphTutorial** directory named **Startup.cs** and add the following code to that file.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Startup.cs" id="StartupSnippet":::

    <span data-ttu-id="f74a4-166">Mit diesem Code wird die [Abhängigkeitsinjektion](https://docs.microsoft.com/azure/azure-functions/functions-dotnet-dependency-injection) in ihren Azure-Funktionen aktiviert, wodurch das `IConfiguration` Objekt und der Dienst verfügbar gemacht werden `GraphClientService` .</span><span class="sxs-lookup"><span data-stu-id="f74a4-166">This code will enable [dependency injection](https://docs.microsoft.com/azure/azure-functions/functions-dotnet-dependency-injection) in your Azure Functions, exposing the `IConfiguration` object and the `GraphClientService` service.</span></span>

### <a name="implement-getmynewestmessage-function"></a><span data-ttu-id="f74a4-167">Implementieren der GetMyNewestMessage-Funktion</span><span class="sxs-lookup"><span data-stu-id="f74a4-167">Implement GetMyNewestMessage function</span></span>

1. <span data-ttu-id="f74a4-168">Öffnen Sie **/GraphTutorial/GetMyNewestMessage.cs** , und ersetzen Sie den gesamten Inhalt durch Folgendes.</span><span class="sxs-lookup"><span data-stu-id="f74a4-168">Open **./GraphTutorial/GetMyNewestMessage.cs** and replace its entire contents with the following.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GetMyNewestMessage.cs" id="GetMyNewestMessageSnippet":::

#### <a name="review-the-code-in-getmynewestmessagecs"></a><span data-ttu-id="f74a4-169">Überprüfen des Codes in GetMyNewestMessage.cs</span><span class="sxs-lookup"><span data-stu-id="f74a4-169">Review the code in GetMyNewestMessage.cs</span></span>

<span data-ttu-id="f74a4-170">Nehmen Sie sich einen Moment Zeit, um zu prüfen, was der Code in **GetMyNewestMessage.cs** macht.</span><span class="sxs-lookup"><span data-stu-id="f74a4-170">Take a moment to consider what the code in **GetMyNewestMessage.cs** does.</span></span>

- <span data-ttu-id="f74a4-171">Im Konstruktor wird das Objekt gespeichert, das `IConfiguration` über Dependency Injection übergeben wurde.</span><span class="sxs-lookup"><span data-stu-id="f74a4-171">In the constructor, it saves the `IConfiguration` object passed in via dependency injection.</span></span>
- <span data-ttu-id="f74a4-172">In der `Run` -Funktion werden folgende Aktionen durchführt:</span><span class="sxs-lookup"><span data-stu-id="f74a4-172">In the `Run` function, it does the following:</span></span>
  - <span data-ttu-id="f74a4-173">Überprüft, ob die erforderlichen Konfigurationswerte im Objekt vorhanden sind `IConfiguration` .</span><span class="sxs-lookup"><span data-stu-id="f74a4-173">Validates the required configuration values are present in the `IConfiguration` object.</span></span>
  - <span data-ttu-id="f74a4-174">Überprüft das Bearer-Token und gibt einen `401` Statuscode zurück, wenn das Token ungültig ist.</span><span class="sxs-lookup"><span data-stu-id="f74a4-174">Validates the bearer token and returns a `401` status code if the token is invalid.</span></span>
  - <span data-ttu-id="f74a4-175">Ruft einen Graph-Client aus dem `GraphClientService` für den Benutzer ab, der diese Anforderung gestellt hat.</span><span class="sxs-lookup"><span data-stu-id="f74a4-175">Gets a Graph client from the `GraphClientService` for the user that made this request.</span></span>
  - <span data-ttu-id="f74a4-176">Verwendet das Microsoft Graph-SDK, um die neueste Nachricht aus dem Posteingang des Benutzers abzurufen und Sie als JSON-Text in der Antwort zurückgibt.</span><span class="sxs-lookup"><span data-stu-id="f74a4-176">Uses the Microsoft Graph SDK to get the newest message from the user's inbox and returns it as a JSON body in the response.</span></span>

## <a name="call-the-azure-function-from-the-test-app"></a><span data-ttu-id="f74a4-177">Aufrufen der Azure-Funktion aus der Test-App</span><span class="sxs-lookup"><span data-stu-id="f74a4-177">Call the Azure Function from the test app</span></span>

1. <span data-ttu-id="f74a4-178">Öffnen Sie **auth.js** , und fügen Sie die folgende Funktion hinzu, um ein Zugriffstoken abzurufen.</span><span class="sxs-lookup"><span data-stu-id="f74a4-178">Open **auth.js** and add the following function to get an access token.</span></span>

    :::code language="javascript" source="../demo/TestClient/auth.js" id="getTokenSnippet":::

    <span data-ttu-id="f74a4-179">Überprüfen Sie die Funktionsweise dieses Codes.</span><span class="sxs-lookup"><span data-stu-id="f74a4-179">Consider what this code does.</span></span>

    - <span data-ttu-id="f74a4-180">Zunächst wird versucht, ein Zugriffstoken ohne Benutzereingriff im Hintergrund abzurufen.</span><span class="sxs-lookup"><span data-stu-id="f74a4-180">It first attempts to get an access token silently, without user interaction.</span></span> <span data-ttu-id="f74a4-181">Da der Benutzer bereits angemeldet sein sollte, sollte MSAL über Token für den Benutzer im Cache verfügen.</span><span class="sxs-lookup"><span data-stu-id="f74a4-181">Since the user should already be signed in, MSAL should have tokens for the user in its cache.</span></span>
    - <span data-ttu-id="f74a4-182">Wenn ein Fehler auftritt, der angibt, dass der Benutzer interagieren muss, versucht er, ein Token interaktiv abzurufen.</span><span class="sxs-lookup"><span data-stu-id="f74a4-182">If that fails with an error that indicates the user needs to interact, it attempts to get a token interactively.</span></span>

1. <span data-ttu-id="f74a4-183">Erstellen Sie eine neue Datei im **Testclient** -Verzeichnis mit dem Namen **azurefunctions.js** , und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="f74a4-183">Create a new file in the **TestClient** directory named **azurefunctions.js** and add the following code.</span></span>

    :::code language="javascript" source="../demo/TestClient/azurefunctions.js" id="getLatestMessageSnippet":::

1. <span data-ttu-id="f74a4-184">Ändern Sie das aktuelle Verzeichnis in ihrer CLI in das Verzeichnis **./GraphTutorial** , und führen Sie den folgenden Befehl aus, um die Azure-Funktion lokal zu starten.</span><span class="sxs-lookup"><span data-stu-id="f74a4-184">Change the current directory in your CLI to the **./GraphTutorial** directory and run the following command to start the Azure Function locally.</span></span>

    ```Shell
    func start
    ```

1. <span data-ttu-id="f74a4-185">Wenn Sie den Whirlpool nicht bereits bereitstellen, öffnen Sie ein zweites CLI-Fenster, und ändern Sie das aktuelle Verzeichnis in das Verzeichnis **./Testclient** .</span><span class="sxs-lookup"><span data-stu-id="f74a4-185">If not already serving the SPA, open a second CLI window and change the current directory to the **./TestClient** directory.</span></span> <span data-ttu-id="f74a4-186">Führen Sie den folgenden Befehl aus, um die Testanwendung auszuführen.</span><span class="sxs-lookup"><span data-stu-id="f74a4-186">Run the following command to run the test application.</span></span>

    ```Shell
    dotnet serve -h "Cache-Control: no-cache, no-store, must-revalidate"
    ```

1. <span data-ttu-id="f74a4-187">Öffnen Sie Ihren Browser und navigieren Sie zu `http://localhost:8080`.</span><span class="sxs-lookup"><span data-stu-id="f74a4-187">Open your browser and navigate to `http://localhost:8080`.</span></span> <span data-ttu-id="f74a4-188">Melden Sie sich an, und wählen Sie das **neueste Nachrichten** Navigationselement aus.</span><span class="sxs-lookup"><span data-stu-id="f74a4-188">Sign in and select the **Latest Message** navigation item.</span></span> <span data-ttu-id="f74a4-189">Die APP zeigt Informationen über die neueste Nachricht im Posteingang des Benutzers an.</span><span class="sxs-lookup"><span data-stu-id="f74a4-189">The app displays information about the newest message in the user's inbox.</span></span>
