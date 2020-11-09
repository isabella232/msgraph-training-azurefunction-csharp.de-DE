---
ms.openlocfilehash: 728138b5d298f9469a566defed6ee2e457896c0b
ms.sourcegitcommit: 57a5c2e1a562d8af092a3e78786d711ce1e8f9cb
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 10/30/2020
ms.locfileid: "48822644"
---
<!-- markdownlint-disable MD002 MD041 -->

In dieser Übung schließen Sie die Implementierung der Azure-Funktion ab `GetMyNewestMessage` und aktualisieren den Testclient, um die Funktion aufzurufen.

Die Azure-Funktion verwendet den [im-Auftrag-von-Fluss](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow). Die grundlegende Reihenfolge der Ereignisse in diesem Fluss sind:

- Die Testanwendung verwendet einen interaktiven Authentifizierungs Fluss, um dem Benutzer die Möglichkeit zu geben, sich anzumelden und die Zustimmung zu erteilen. Es ruft ein Token zurück, das auf die Azure-Funktion ausgelegt ist. Das **Token enthält keine** Microsoft Graph-Bereiche.
- Die Testanwendung Ruft die Azure-Funktion auf, wobei ihr Zugriffstoken in der Kopfzeile gesendet wird `Authorization` .
- Die Azure-Funktion überprüft das Token und tauscht dann dieses Token für ein zweites Zugriffstoken aus, das Microsoft Graph-Bereiche enthält.
- Die Azure-Funktion ruft Microsoft Graph im Namen des Benutzers mithilfe des zweiten Zugriffstokens auf.

> [!IMPORTANT]
> Um zu vermeiden, dass die Anwendungs-ID und der geheime Schlüssel in der Quelle gespeichert werden, verwenden Sie den [.net Secret Manager](https://docs.microsoft.com/aspnet/core/security/app-secrets) zum Speichern dieser Werte. Der geheime Manager dient nur Entwicklungszwecken, für Produktions-apps sollte ein vertrauenswürdiger geheimer geheim Manager zum Speichern von Geheimnissen verwendet werden.

## <a name="add-authentication-to-the-single-page-application"></a>Hinzufügen der Authentifizierung zur Anwendung für einzelne Seiten

Beginnen Sie mit dem Hinzufügen der Authentifizierung zum Spa. Dadurch kann die Anwendung ein Zugriffstoken abrufen, das den Zugriff zum Aufrufen der Azure-Funktion gewährt. Da es sich hierbei um ein Spa handelt, wird der [Autorisierungscode Fluss mit PKCE](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-auth-code-flow)verwendet.

1. Erstellen Sie eine neue Datei im **Testclient** -Verzeichnis mit dem Namen **config.js** , und fügen Sie den folgenden Code hinzu.

    :::code language="javascript" source="../demo/TestClient/config.example.js" id="msalConfigSnippet":::

    Ersetzen `YOUR_TEST_APP_APP_ID_HERE` Sie durch die Anwendungs-ID, die Sie im Azure-Portal für die **Graph Azure Function Test-App** erstellt haben. Ersetzen `YOUR_TENANT_ID_HERE` Sie durch den **Verzeichnis Wert (Mandanten)** , den Sie aus dem Azure-Portal kopiert haben. Ersetzen Sie `YOUR_AZURE_FUNCTION_APP_ID_HERE` durch die Anwendungs-ID für die **Graph-Azure-Funktion**.

    > [!IMPORTANT]
    > Wenn Sie die Quellcodeverwaltung wie git verwenden, wäre es nun ein guter Zeitpunkt, die **config.js** Datei aus der Quellcodeverwaltung auszuschließen, um zu vermeiden, dass versehentlich Ihre APP-IDs und die Mandanten-ID verloren gehen.

1. Erstellen Sie eine neue Datei im **Testclient** -Verzeichnis mit dem Namen **auth.js** , und fügen Sie den folgenden Code hinzu.

    :::code language="javascript" source="../demo/TestClient/auth.js" id="signInSignOutSnippet":::

    Überprüfen Sie die Funktionsweise dieses Codes.

    - Es initialisiert a `PublicClientApplication` mithilfe der in **config.js** gespeicherten Werte.
    - Er verwendet `loginPopup` , um den Benutzer unter Verwendung des Berechtigungs Bereichs für die Azure-Funktion zu signieren.
    - Der Benutzername wird in der Sitzung gespeichert.

    > [!IMPORTANT]
    > Da die APP verwendet `loginPopup` wird, müssen Sie möglicherweise den Popupblocker Ihres Browsers ändern, um Popups von zu ermöglichen `http://localhost:8080` .

1. Aktualisieren Sie die Seite, und melden Sie sich an. Die Seite sollte mit dem Benutzernamen aktualisiert werden, was bedeutet, dass das Anmelden erfolgreich war.

> [!TIP]
> Sie können das Zugriffstoken analysieren [https://jwt.ms](https://jwt.ms) und bestätigen, dass der `aud` Anspruch die APP-ID für die Azure-Funktion ist und dass der `scp` Anspruch den Berechtigungsbereich der Azure-Funktion enthält, nicht Microsoft Graph.

## <a name="add-authentication-to-the-azure-function"></a>Hinzufügen von Authentifizierung zur Azure-Funktion

In diesem Abschnitt implementieren Sie den im-Auftrag-von-Ablauf in der `GetMyNewestMessage` Azure-Funktion, um ein mit Microsoft Graph kompatibles Zugriffstoken abzurufen.

1. Initialisieren Sie den geheimen .net-entwicklungsspeicher, indem Sie die CLI in dem Verzeichnis öffnen, das **GraphTutorial. csproj** enthält, und führen Sie den folgenden Befehl aus.

    ```Shell
    dotnet user-secrets init
    ```

1. Fügen Sie die Anwendungs-ID, die geheime und die Mandanten-ID dem geheimen Speicher mit den folgenden Befehlen hinzu. Ersetzen Sie `YOUR_API_FUNCTION_APP_ID_HERE` durch die Anwendungs-ID für die **Graph-Azure-Funktion**. Ersetzen `YOUR_API_FUNCTION_APP_SECRET_HERE` Sie durch den Anwendungsschlüssel, den Sie im Azure-Portal für die **Graph-Azure-Funktion** erstellt haben. Ersetzen `YOUR_TENANT_ID_HERE` Sie durch den **Verzeichnis Wert (Mandanten)** , den Sie aus dem Azure-Portal kopiert haben.

    ```Shell
    dotnet user-secrets set apiFunctionId "YOUR_API_FUNCTION_APP_ID_HERE"
    dotnet user-secrets set apiFunctionSecret "YOUR_API_FUNCTION_APP_SECRET_HERE"
    dotnet user-secrets set tenantId "YOUR_TENANT_ID_HERE"
    ```

### <a name="process-the-incoming-bearer-token"></a>Verarbeiten des eingehenden Bearer-Tokens

In diesem Abschnitt implementieren Sie eine Klasse zum Validieren und Verarbeiten des Bearer-Tokens, das vom Spa an die Azure-Funktion gesendet wurde.

1. Erstellen Sie ein neues Verzeichnis im **GraphTutorial** -Verzeichnis mit dem Namen **Authentication**.

1. Erstellen Sie im Ordner **./GraphTutorial/Authentication** eine neue Datei mit dem Namen **TokenValidationResult.cs** , und fügen Sie den folgenden Code hinzu.

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/TokenValidationResult.cs" id="TokenValidationResultSnippet":::

1. Erstellen Sie im Ordner **./GraphTutorial/Authentication** eine neue Datei mit dem Namen **TokenValidation.cs** , und fügen Sie den folgenden Code hinzu.

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/TokenValidation.cs" id="TokenValidationSnippet":::

Überprüfen Sie die Funktionsweise dieses Codes.

- Sie stellen sicher, dass in der Kopfzeile ein Bearer-Token vorhanden ist `Authorization` .
- Die Signatur und der Aussteller werden anhand der veröffentlichten OpenID-Konfiguration von Azure überprüft.
- Er überprüft, ob die Benutzergruppe ( `aud` Forderung) mit der Anwendungs-ID der Azure-Funktion übereinstimmt.
- Es analysiert das Token und generiert eine MSAL-Konto-ID, die benötigt wird, um die Token-Zwischenspeicherung zu nutzen.

### <a name="create-an-on-behalf-of-authentication-provider"></a>Erstellen eines Authentifizierungsanbieters im Auftrag von

1. Erstellen Sie eine neue Datei im **Authentifizierungs** Verzeichnis mit dem Namen **OnBehalfOfAuthProvider.cs** , und fügen Sie der Datei den folgenden Code hinzu.

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/OnBehalfOfAuthProvider.cs" id="AuthProviderSnippet":::

Nehmen Sie sich einen Moment Zeit, um zu prüfen, was der Code in **OnBehalfOfAuthProvider.cs** macht.

- In der `GetAccessToken` -Funktion wird zunächst versucht, ein Benutzertoken aus dem Token-Cache mithilfe von zu erhalten `AcquireTokenSilent` . Wenn dies fehlschlägt, verwendet es das Bearer-Token, das von der Test-App an die Azure-Funktion gesendet wird, um eine Benutzer Assertion zu generieren. Anschließend wird diese Benutzer Assertion verwendet, um ein Graph-kompatibles Token mithilfe von zu erhalten `AcquireTokenOnBehalfOf` .
- Die `Microsoft.Graph.IAuthenticationProvider` Schnittstelle wird implementiert, sodass diese Klasse im Konstruktor von The übergeben werden kann, `GraphServiceClient` um ausgehende Anforderungen zu authentifizieren.

### <a name="implement-a-graph-client-service"></a>Implementieren eines Graph-Clientdiensts

In diesem Abschnitt implementieren Sie einen Dienst, der für die [Abhängigkeitsinjektion](https://docs.microsoft.com/azure/azure-functions/functions-dotnet-dependency-injection)registriert werden kann. Der Dienst wird verwendet, um einen authentifizierten Graph-Client abzurufen.

1. Erstellen Sie ein neues Verzeichnis im **GraphTutorial** -Verzeichnis mit dem Namen **Services**.

1. Erstellen Sie eine neue Datei im Verzeichnis **Dienste** mit dem Namen **IGraphClientService.cs** , und fügen Sie der Datei den folgenden Code hinzu.

    :::code language="csharp" source="../demo/GraphTutorial/Services/IGraphClientService.cs" id="IGraphClientServiceSnippet":::

1. Erstellen Sie eine neue Datei im Verzeichnis **Dienste** mit dem Namen **GraphClientService.cs** , und fügen Sie der Datei den folgenden Code hinzu.

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

1. Fügen Sie der Klasse die folgenden Eigenschaften hinzu `GraphClientService` .

    :::code language="csharp" source="../demo/GraphTutorial/Services/GraphClientService.cs" id="UserGraphClientMembers":::

1. Fügen Sie der-Klasse die folgenden Funktionen hinzu `GraphClientService` .

    :::code language="csharp" source="../demo/GraphTutorial/Services/GraphClientService.cs" id="UserGraphClientFunctions":::

1. Fügen Sie eine Platzhalterimplementierung für die `GetAppGraphClient` Funktion hinzu. Sie werden dies in späteren Abschnitten implementieren.

    ```csharp
    public GraphServiceClient GetAppGraphClient()
    {
        throw new System.NotImplementedException();
    }
    ```

    Die `GetUserGraphClient` -Funktion verwendet die Ergebnisse der Tokenprüfung und erstellt eine authentifizierte `GraphServiceClient` für den Benutzer.

1. Erstellen Sie eine neue Datei im **GraphTutorial** -Verzeichnis mit dem Namen **Startup.cs** , und fügen Sie der Datei den folgenden Code hinzu.

    :::code language="csharp" source="../demo/GraphTutorial/Startup.cs" id="StartupSnippet":::

    Mit diesem Code wird die [Abhängigkeitsinjektion](https://docs.microsoft.com/azure/azure-functions/functions-dotnet-dependency-injection) in ihren Azure-Funktionen aktiviert, wodurch das `IConfiguration` Objekt und der Dienst verfügbar gemacht werden `GraphClientService` .

### <a name="implement-getmynewestmessage-function"></a>Implementieren der GetMyNewestMessage-Funktion

1. Öffnen Sie **/GraphTutorial/GetMyNewestMessage.cs** , und ersetzen Sie den gesamten Inhalt durch Folgendes.

    :::code language="csharp" source="../demo/GraphTutorial/GetMyNewestMessage.cs" id="GetMyNewestMessageSnippet":::

#### <a name="review-the-code-in-getmynewestmessagecs"></a>Überprüfen des Codes in GetMyNewestMessage.cs

Nehmen Sie sich einen Moment Zeit, um zu prüfen, was der Code in **GetMyNewestMessage.cs** macht.

- Im Konstruktor wird das Objekt gespeichert, das `IConfiguration` über Dependency Injection übergeben wurde.
- In der `Run` -Funktion werden folgende Aktionen durchführt:
  - Überprüft, ob die erforderlichen Konfigurationswerte im Objekt vorhanden sind `IConfiguration` .
  - Überprüft das Bearer-Token und gibt einen `401` Statuscode zurück, wenn das Token ungültig ist.
  - Ruft einen Graph-Client aus dem `GraphClientService` für den Benutzer ab, der diese Anforderung gestellt hat.
  - Verwendet das Microsoft Graph-SDK, um die neueste Nachricht aus dem Posteingang des Benutzers abzurufen und Sie als JSON-Text in der Antwort zurückgibt.

## <a name="call-the-azure-function-from-the-test-app"></a>Aufrufen der Azure-Funktion aus der Test-App

1. Öffnen Sie **auth.js** , und fügen Sie die folgende Funktion hinzu, um ein Zugriffstoken abzurufen.

    :::code language="javascript" source="../demo/TestClient/auth.js" id="getTokenSnippet":::

    Überprüfen Sie die Funktionsweise dieses Codes.

    - Zunächst wird versucht, ein Zugriffstoken ohne Benutzereingriff im Hintergrund abzurufen. Da der Benutzer bereits angemeldet sein sollte, sollte MSAL über Token für den Benutzer im Cache verfügen.
    - Wenn ein Fehler auftritt, der angibt, dass der Benutzer interagieren muss, versucht er, ein Token interaktiv abzurufen.

1. Erstellen Sie eine neue Datei im **Testclient** -Verzeichnis mit dem Namen **azurefunctions.js** , und fügen Sie den folgenden Code hinzu.

    :::code language="javascript" source="../demo/TestClient/azurefunctions.js" id="getLatestMessageSnippet":::

1. Ändern Sie das aktuelle Verzeichnis in ihrer CLI in das Verzeichnis **./GraphTutorial** , und führen Sie den folgenden Befehl aus, um die Azure-Funktion lokal zu starten.

    ```Shell
    func start
    ```

1. Wenn Sie den Whirlpool nicht bereits bereitstellen, öffnen Sie ein zweites CLI-Fenster, und ändern Sie das aktuelle Verzeichnis in das Verzeichnis **./Testclient** . Führen Sie den folgenden Befehl aus, um die Testanwendung auszuführen.

    ```Shell
    dotnet serve -h "Cache-Control: no-cache, no-store, must-revalidate"
    ```

1. Öffnen Sie Ihren Browser und navigieren Sie zu `http://localhost:8080`. Melden Sie sich an, und wählen Sie das **neueste Nachrichten** Navigationselement aus. Die APP zeigt Informationen über die neueste Nachricht im Posteingang des Benutzers an.
