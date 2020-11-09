---
ms.openlocfilehash: 0c9c6703a54e50f576a171a4a161305c9b9ae6ff
ms.sourcegitcommit: 57a5c2e1a562d8af092a3e78786d711ce1e8f9cb
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 10/30/2020
ms.locfileid: "48822640"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="b63f9-101">In dieser Übung erstellen Sie drei neue Azure AD-Anwendungen mithilfe des Azure Active Directory Admin Center:</span><span class="sxs-lookup"><span data-stu-id="b63f9-101">In this exercise you will create three new Azure AD applications using the Azure Active Directory admin center:</span></span>

- <span data-ttu-id="b63f9-102">Eine APP-Registrierung für die einseitige Anwendung, sodass Sie Benutzer anmelden und Token erhalten kann, die es der Anwendung ermöglichen, die Azure-Funktion aufzurufen.</span><span class="sxs-lookup"><span data-stu-id="b63f9-102">An app registration for the single-page application so that it can sign in users and get tokens allowing the application to call the Azure Function.</span></span>
- <span data-ttu-id="b63f9-103">Eine APP-Registrierung für die Azure-Funktion, mit der Sie den [im-Auftrag-von-Fluss](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow) zum Austauschen des vom Spa gesendeten Tokens für ein Token verwenden können, mit dem Microsoft Graph aufgerufen werden kann.</span><span class="sxs-lookup"><span data-stu-id="b63f9-103">An app registration for the Azure Function that allows it to use the [on-behalf-of flow](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow) to exchange the token sent by the SPA for a token that will allow it to call Microsoft Graph.</span></span>
- <span data-ttu-id="b63f9-104">Eine APP-Registrierung für die Azure-Funktion webhook, die es ermöglicht, den [Client-Anmelde Informationsfluss](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow) zu verwenden, um Microsoft Graph ohne Benutzer aufzurufen.</span><span class="sxs-lookup"><span data-stu-id="b63f9-104">An app registration for the Azure Function webhook that allows it to use the [client credential flow](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow) to call Microsoft Graph without a user.</span></span>

> [!NOTE]
> <span data-ttu-id="b63f9-105">In diesem Beispiel sind drei App-Registrierungen erforderlich, da Sie sowohl den Fluss "im Auftrag von" als auch den Client-Anmeldeinformationen implementieren.</span><span class="sxs-lookup"><span data-stu-id="b63f9-105">This example requires three app registrations because it is implementing both the on-behalf-of flow and the client credential flow.</span></span> <span data-ttu-id="b63f9-106">Wenn Ihre Azure-Funktion nur einen dieser Datenflüsse verwendet, müssen Sie nur die APP-Registrierungen erstellen, die diesem Fluss entsprechen.</span><span class="sxs-lookup"><span data-stu-id="b63f9-106">If your Azure Function only uses one of these flows, you would only need to create the app registrations that correspond to that flow.</span></span>

1. <span data-ttu-id="b63f9-107">Öffnen Sie einen Browser, und navigieren Sie zum [Azure Active Directory Admin Center](https://aad.portal.azure.com) , und melden Sie sich mit einem Administrator der Microsoft 365-mandantenorganisation an.</span><span class="sxs-lookup"><span data-stu-id="b63f9-107">Open a browser and navigate to the [Azure Active Directory admin center](https://aad.portal.azure.com) and login using an Microsoft 365 tenant organization admin.</span></span>

1. <span data-ttu-id="b63f9-108">Wählen Sie in der linken Navigationsleiste **Azure Active Directory** aus, und wählen Sie dann **App-Registrierungen** unter **Verwalten** aus.</span><span class="sxs-lookup"><span data-stu-id="b63f9-108">Select **Azure Active Directory** in the left-hand navigation, then select **App registrations** under **Manage**.</span></span>

    ![<span data-ttu-id="b63f9-109">Screenshot der APP-Registrierungen</span><span class="sxs-lookup"><span data-stu-id="b63f9-109">A screenshot of the App registrations</span></span> ](./images/aad-portal-app-registrations.png)

## <a name="register-an-app-for-the-single-page-application"></a><span data-ttu-id="b63f9-110">Registrieren einer APP für die Anwendung mit einer einzelnen Seite</span><span class="sxs-lookup"><span data-stu-id="b63f9-110">Register an app for the single-page application</span></span>

1. <span data-ttu-id="b63f9-111">Wählen Sie **Neue Registrierung** aus.</span><span class="sxs-lookup"><span data-stu-id="b63f9-111">Select **New registration**.</span></span> <span data-ttu-id="b63f9-112">Legen Sie auf der Seite **Anwendung registrieren** die Werte wie folgt fest.</span><span class="sxs-lookup"><span data-stu-id="b63f9-112">On the **Register an application** page, set the values as follows.</span></span>

    - <span data-ttu-id="b63f9-113">Legen Sie **Name** auf `Graph Azure Function Test App` fest.</span><span class="sxs-lookup"><span data-stu-id="b63f9-113">Set **Name** to `Graph Azure Function Test App`.</span></span>
    - <span data-ttu-id="b63f9-114">Legen Sie die **unterstützten Kontotypen** **nur in diesem Organisations Verzeichnis auf Konten** fest.</span><span class="sxs-lookup"><span data-stu-id="b63f9-114">Set **Supported account types** to **Accounts in this organizational directory only**.</span></span>
    - <span data-ttu-id="b63f9-115">Ändern Sie unter **Umleitungs-URI** das Dropdown in **Single-Page Application (Spa)** , und legen Sie den Wert auf fest `http://localhost:8080` .</span><span class="sxs-lookup"><span data-stu-id="b63f9-115">Under **Redirect URI** , change the dropdown to **Single-page application (SPA)** and set the value to `http://localhost:8080`.</span></span>

    ![Screenshot der Seite "Anwendung registrieren"](./images/register-command-line-app.png)

1. <span data-ttu-id="b63f9-117">Wählen Sie **Registrieren** aus.</span><span class="sxs-lookup"><span data-stu-id="b63f9-117">Select **Register**.</span></span> <span data-ttu-id="b63f9-118">Kopieren Sie auf der Seite **Graph Azure Function Test App** die Werte der Anwendungs-ID **(Client) ID** und **Verzeichnis (Mandanten)** , und speichern Sie Sie, dann benötigen Sie Sie in den späteren Schritten.</span><span class="sxs-lookup"><span data-stu-id="b63f9-118">On the **Graph Azure Function Test App** page, copy the values of the **Application (client) ID** and **Directory (tenant) ID** and save them, you will need them in the later steps.</span></span>

    ![Screenshot der Anwendungs-ID der neuen App-Registrierung](./images/aad-application-id.png)

## <a name="register-an-app-for-the-azure-function"></a><span data-ttu-id="b63f9-120">Registrieren einer APP für die Azure-Funktion</span><span class="sxs-lookup"><span data-stu-id="b63f9-120">Register an app for the Azure Function</span></span>

1. <span data-ttu-id="b63f9-121">Kehren Sie zu **App-Registrierungen** zurück, und wählen Sie **neue Registrierung** aus.</span><span class="sxs-lookup"><span data-stu-id="b63f9-121">Return to **App Registrations** , and select **New registration**.</span></span> <span data-ttu-id="b63f9-122">Legen Sie auf der Seite **Anwendung registrieren** die Werte wie folgt fest.</span><span class="sxs-lookup"><span data-stu-id="b63f9-122">On the **Register an application** page, set the values as follows.</span></span>

    - <span data-ttu-id="b63f9-123">Legen Sie **Name** auf `Graph Azure Function` fest.</span><span class="sxs-lookup"><span data-stu-id="b63f9-123">Set **Name** to `Graph Azure Function`.</span></span>
    - <span data-ttu-id="b63f9-124">Legen Sie die **unterstützten Kontotypen** **nur in diesem Organisations Verzeichnis auf Konten** fest.</span><span class="sxs-lookup"><span data-stu-id="b63f9-124">Set **Supported account types** to **Accounts in this organizational directory only**.</span></span>
    - <span data-ttu-id="b63f9-125">Lassen Sie die **Umleitungs-URI** leer.</span><span class="sxs-lookup"><span data-stu-id="b63f9-125">Leave **Redirect URI** blank.</span></span>

1. <span data-ttu-id="b63f9-126">Wählen Sie **Registrieren** aus.</span><span class="sxs-lookup"><span data-stu-id="b63f9-126">Select **Register**.</span></span> <span data-ttu-id="b63f9-127">Kopieren Sie auf der Seite **Diagramm Azure-Funktion** den Wert der **Anwendungs-ID (Client)** , und speichern Sie Sie, um Sie im nächsten Schritt zu benötigen.</span><span class="sxs-lookup"><span data-stu-id="b63f9-127">On the **Graph Azure Function** page, copy the value of the **Application (client) ID** and save it, you will need it in the next step.</span></span>

1. <span data-ttu-id="b63f9-128">Wählen Sie unter **Verwalten** die Option **Zertifikate und Geheime Clientschlüssel** aus.</span><span class="sxs-lookup"><span data-stu-id="b63f9-128">Select **Certificates & secrets** under **Manage**.</span></span> <span data-ttu-id="b63f9-129">Wählen Sie die Schaltfläche **Neuen geheimen Clientschlüssel** aus.</span><span class="sxs-lookup"><span data-stu-id="b63f9-129">Select the **New client secret** button.</span></span> <span data-ttu-id="b63f9-130">Geben Sie einen Wert in **Beschreibung** ein, wählen Sie eine der Optionen für **Gilt bis** aus, und wählen Sie dann **Hinzufügen** aus.</span><span class="sxs-lookup"><span data-stu-id="b63f9-130">Enter a value in **Description** and select one of the options for **Expires** and select **Add**.</span></span>

    ![Screenshot des Dialogfelds "Geheimen Clientschlüssel hinzufügen"](./images/aad-new-client-secret.png)

1. <span data-ttu-id="b63f9-132">Kopieren Sie den Wert des geheimen Clientschlüssels, bevor Sie diese Seite verlassen.</span><span class="sxs-lookup"><span data-stu-id="b63f9-132">Copy the client secret value before you leave this page.</span></span> <span data-ttu-id="b63f9-133">Sie benötigen ihn im nächsten Schritt.</span><span class="sxs-lookup"><span data-stu-id="b63f9-133">You will need it in the next step.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="b63f9-134">Dieser geheime Clientschlüssel wird nicht noch einmal angezeigt, stellen Sie daher sicher, dass Sie ihn jetzt kopieren.</span><span class="sxs-lookup"><span data-stu-id="b63f9-134">This client secret is never shown again, so make sure you copy it now.</span></span>

    ![Screenshot des neu hinzugefügten Clientschlüssels](./images/aad-copy-client-secret.png)

1. <span data-ttu-id="b63f9-136">Wählen Sie unter **Manage** die **API-Berechtigungen** aus.</span><span class="sxs-lookup"><span data-stu-id="b63f9-136">Select **API Permissions** under **Manage**.</span></span> <span data-ttu-id="b63f9-137">Wählen Sie **Berechtigung hinzufügen** aus.</span><span class="sxs-lookup"><span data-stu-id="b63f9-137">Choose **Add a permission**.</span></span>

1. <span data-ttu-id="b63f9-138">Wählen Sie **Microsoft Graph** und dann **Delegierte Berechtigungen** aus.</span><span class="sxs-lookup"><span data-stu-id="b63f9-138">Select **Microsoft Graph** , then **Delegated Permissions**.</span></span> <span data-ttu-id="b63f9-139">Fügen Sie **Mail. Read** hinzu, und wählen Sie **Berechtigungen hinzufügen** aus.</span><span class="sxs-lookup"><span data-stu-id="b63f9-139">Add **Mail.Read** and select **Add permissions**.</span></span>

    ![Ein Screenshot der konfigurierten Berechtigungen für die Azure-Funktion-App-Registrierung](./images/web-api-configured-permissions.png)

1. <span data-ttu-id="b63f9-141">Wählen Sie unter **Manage** **eine API verfügbar machen** aus, und wählen Sie dann **Bereich hinzufügen** aus.</span><span class="sxs-lookup"><span data-stu-id="b63f9-141">Select **Expose an API** under **Manage** , then choose **Add a scope**.</span></span>

1. <span data-ttu-id="b63f9-142">Akzeptieren Sie den Standard **-URI der Anwendungs-ID** , und wählen Sie **Speichern und Fortfahren**.</span><span class="sxs-lookup"><span data-stu-id="b63f9-142">Accept the default **Application ID URI** and choose **Save and continue**.</span></span>

1. <span data-ttu-id="b63f9-143">Füllen Sie das Formular zum **Hinzufügen eines Bereichs** wie folgt aus:</span><span class="sxs-lookup"><span data-stu-id="b63f9-143">Fill in the **Add a scope** form as follows:</span></span>

    - <span data-ttu-id="b63f9-144">**Bereichsname:** Mail. Read</span><span class="sxs-lookup"><span data-stu-id="b63f9-144">**Scope name:** Mail.Read</span></span>
    - <span data-ttu-id="b63f9-145">**Wer kann zustimmen?:** Administratoren und Benutzer</span><span class="sxs-lookup"><span data-stu-id="b63f9-145">**Who can consent?:** Admins and users</span></span>
    - <span data-ttu-id="b63f9-146">**Anzeigename der Administrator Zustimmung:** Alle Posteingänge aller Benutzer lesen</span><span class="sxs-lookup"><span data-stu-id="b63f9-146">**Admin consent display name:** Read all users' inboxes</span></span>
    - <span data-ttu-id="b63f9-147">**Beschreibung der Administrator Zustimmung:** Ermöglicht der APP, alle Posteingänge aller Benutzer zu lesen.</span><span class="sxs-lookup"><span data-stu-id="b63f9-147">**Admin consent description:** Allows the app to read all users' inboxes</span></span>
    - <span data-ttu-id="b63f9-148">**Anzeigename der Benutzer Zustimmung:** Lesen Ihres Posteingangs</span><span class="sxs-lookup"><span data-stu-id="b63f9-148">**User consent display name:** Read your inbox</span></span>
    - <span data-ttu-id="b63f9-149">**Beschreibung der Benutzer Zustimmung:** Ermöglicht der APP, Ihren Posteingang zu lesen</span><span class="sxs-lookup"><span data-stu-id="b63f9-149">**User consent description:** Allows the app to read your inbox</span></span>
    - <span data-ttu-id="b63f9-150">**Status:** Aktiviert</span><span class="sxs-lookup"><span data-stu-id="b63f9-150">**State:** Enabled</span></span>

1. <span data-ttu-id="b63f9-151">Klicken Sie auf **Bereich hinzufügen**.</span><span class="sxs-lookup"><span data-stu-id="b63f9-151">Select **Add scope**.</span></span>

1. <span data-ttu-id="b63f9-152">Kopieren Sie den neuen Bereich, Sie benötigen ihn in späteren Schritten.</span><span class="sxs-lookup"><span data-stu-id="b63f9-152">Copy the new scope, you'll need it in later steps.</span></span>

    ![Ein Screenshot der definierten Bereiche für die APP-Registrierung in Azure-Funktionen](./images/web-api-defined-scopes.png)

1. <span data-ttu-id="b63f9-154">Wählen Sie unter **Manage** die Option **Manifest** aus.</span><span class="sxs-lookup"><span data-stu-id="b63f9-154">Select **Manifest** under **Manage**.</span></span>

1. <span data-ttu-id="b63f9-155">Suchen `knownClientApplications` Sie im Manifest, und ersetzen Sie den aktuellen Wert von `[]` mit `[TEST_APP_ID]` , wobei `TEST_APP_ID` die Anwendungs-ID der APP-App-Registrierung der **Graph-Azure-Funktions Test** ist.</span><span class="sxs-lookup"><span data-stu-id="b63f9-155">Locate `knownClientApplications` in the manifest, and replace it's current value of `[]` with `[TEST_APP_ID]`, where `TEST_APP_ID` is the application ID of the **Graph Azure Function Test App** app registration.</span></span> <span data-ttu-id="b63f9-156">Wählen Sie **Speichern** aus.</span><span class="sxs-lookup"><span data-stu-id="b63f9-156">Select **Save**.</span></span>

> [!NOTE]
> <span data-ttu-id="b63f9-157">Durch Hinzufügen der APP-ID der Testanwendung zur `knownClientApplications` Eigenschaft im Manifest der Azure-Funktion kann die Testanwendung einen [kombinierten Zustimmungs Fluss](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow#default-and-combined-consent)auslösen.</span><span class="sxs-lookup"><span data-stu-id="b63f9-157">Adding the test application's app ID to the `knownClientApplications` property in the Azure Function's manifest allows the test application to trigger a [combined consent flow](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow#default-and-combined-consent).</span></span> <span data-ttu-id="b63f9-158">Dies ist erforderlich, damit der "im Auftrag von"-Fluss funktioniert.</span><span class="sxs-lookup"><span data-stu-id="b63f9-158">This is necessary for the on-behalf-of flow to work.</span></span>

## <a name="add-azure-function-scope-to-test-application-registration"></a><span data-ttu-id="b63f9-159">Hinzufügen eines Azure-Funktionsbereichs zum Testen der Anwendungsregistrierung</span><span class="sxs-lookup"><span data-stu-id="b63f9-159">Add Azure Function scope to test application registration</span></span>

1. <span data-ttu-id="b63f9-160">Kehren Sie zur APP-Registrierung von **Graph Azure Function Test** zurück, und wählen Sie unter **Manage** die Option **API-Berechtigungen** aus.</span><span class="sxs-lookup"><span data-stu-id="b63f9-160">Return to the **Graph Azure Function Test App** registration, and select **API Permissions** under **Manage**.</span></span> <span data-ttu-id="b63f9-161">Wählen Sie **Berechtigung hinzufügen** aus.</span><span class="sxs-lookup"><span data-stu-id="b63f9-161">Select **Add a permission**.</span></span>

1. <span data-ttu-id="b63f9-162">Wählen Sie **meine APIs** aus, und wählen Sie dann **mehr laden** aus.</span><span class="sxs-lookup"><span data-stu-id="b63f9-162">Select **My APIs** , then select **Load more**.</span></span> <span data-ttu-id="b63f9-163">Wählen Sie **Graph Azure Function** aus.</span><span class="sxs-lookup"><span data-stu-id="b63f9-163">Select **Graph Azure Function**.</span></span>

    ![Screenshot des Dialogfelds "Anforderungs-API-Berechtigungen"](./images/test-app-add-permissions.png)

1. <span data-ttu-id="b63f9-165">Wählen Sie die Berechtigung **Mail. Read** aus, und wählen Sie **Berechtigungen hinzufügen** aus.</span><span class="sxs-lookup"><span data-stu-id="b63f9-165">Select the **Mail.Read** permission, then select **Add permissions**.</span></span>

1. <span data-ttu-id="b63f9-166">Entfernen Sie in den **konfigurierten Berechtigungen** die Berechtigung **User. Read** unter **Microsoft Graph** , indem Sie rechts neben der Berechtigung die **...** auswählen und die Berechtigung **Entfernen** auswählen.</span><span class="sxs-lookup"><span data-stu-id="b63f9-166">In the **Configured permissions** , remove the **User.Read** permission under **Microsoft Graph** by selecting the **...** to the right of the permission and selecting **Remove permission**.</span></span> <span data-ttu-id="b63f9-167">Wählen Sie **Ja, entfernen** zur Bestätigung aus.</span><span class="sxs-lookup"><span data-stu-id="b63f9-167">Select **Yes, remove** to confirm.</span></span>

    ![Ein Screenshot der konfigurierten Berechtigungen für die APP-App-Registrierung](./images/test-app-configured-permissions.png)

## <a name="register-an-app-for-the-azure-function-webhook"></a><span data-ttu-id="b63f9-169">Registrieren einer APP für die Azure-Funktion webhook</span><span class="sxs-lookup"><span data-stu-id="b63f9-169">Register an app for the Azure Function webhook</span></span>

1. <span data-ttu-id="b63f9-170">Kehren Sie zu **App-Registrierungen** zurück, und wählen Sie **neue Registrierung** aus.</span><span class="sxs-lookup"><span data-stu-id="b63f9-170">Return to **App Registrations** , and select **New registration**.</span></span> <span data-ttu-id="b63f9-171">Legen Sie auf der Seite **Anwendung registrieren** die Werte wie folgt fest.</span><span class="sxs-lookup"><span data-stu-id="b63f9-171">On the **Register an application** page, set the values as follows.</span></span>

    - <span data-ttu-id="b63f9-172">Legen Sie **Name** auf `Graph Azure Function Webhook` fest.</span><span class="sxs-lookup"><span data-stu-id="b63f9-172">Set **Name** to `Graph Azure Function Webhook`.</span></span>
    - <span data-ttu-id="b63f9-173">Legen Sie die **unterstützten Kontotypen** **nur in diesem Organisations Verzeichnis auf Konten** fest.</span><span class="sxs-lookup"><span data-stu-id="b63f9-173">Set **Supported account types** to **Accounts in this organizational directory only**.</span></span>
    - <span data-ttu-id="b63f9-174">Lassen Sie die **Umleitungs-URI** leer.</span><span class="sxs-lookup"><span data-stu-id="b63f9-174">Leave **Redirect URI** blank.</span></span>

1. <span data-ttu-id="b63f9-175">Wählen Sie **Registrieren** aus.</span><span class="sxs-lookup"><span data-stu-id="b63f9-175">Select **Register**.</span></span> <span data-ttu-id="b63f9-176">Kopieren Sie auf der Seite **Graph Azure Function webhook** den Wert der **Anwendungs-ID (Client)** , und speichern Sie Sie, um Sie im nächsten Schritt zu benötigen.</span><span class="sxs-lookup"><span data-stu-id="b63f9-176">On the **Graph Azure Function webhook** page, copy the value of the **Application (client) ID** and save it, you will need it in the next step.</span></span>

1. <span data-ttu-id="b63f9-177">Wählen Sie unter **Verwalten** die Option **Zertifikate und Geheime Clientschlüssel** aus.</span><span class="sxs-lookup"><span data-stu-id="b63f9-177">Select **Certificates & secrets** under **Manage**.</span></span> <span data-ttu-id="b63f9-178">Wählen Sie die Schaltfläche **Neuen geheimen Clientschlüssel** aus.</span><span class="sxs-lookup"><span data-stu-id="b63f9-178">Select the **New client secret** button.</span></span> <span data-ttu-id="b63f9-179">Geben Sie einen Wert in **Beschreibung** ein, wählen Sie eine der Optionen für **Gilt bis** aus, und wählen Sie dann **Hinzufügen** aus.</span><span class="sxs-lookup"><span data-stu-id="b63f9-179">Enter a value in **Description** and select one of the options for **Expires** and select **Add**.</span></span>

1. <span data-ttu-id="b63f9-180">Kopieren Sie den Wert des geheimen Clientschlüssels, bevor Sie diese Seite verlassen.</span><span class="sxs-lookup"><span data-stu-id="b63f9-180">Copy the client secret value before you leave this page.</span></span> <span data-ttu-id="b63f9-181">Sie benötigen ihn im nächsten Schritt.</span><span class="sxs-lookup"><span data-stu-id="b63f9-181">You will need it in the next step.</span></span>

1. <span data-ttu-id="b63f9-182">Wählen Sie unter **Manage** die **API-Berechtigungen** aus.</span><span class="sxs-lookup"><span data-stu-id="b63f9-182">Select **API Permissions** under **Manage**.</span></span> <span data-ttu-id="b63f9-183">Wählen Sie **Berechtigung hinzufügen** aus.</span><span class="sxs-lookup"><span data-stu-id="b63f9-183">Choose **Add a permission**.</span></span>

1. <span data-ttu-id="b63f9-184">Wählen Sie **Microsoft Graph** und dann **Anwendungsberechtigungen** aus.</span><span class="sxs-lookup"><span data-stu-id="b63f9-184">Select **Microsoft Graph** , then **Application Permissions**.</span></span> <span data-ttu-id="b63f9-185">Fügen Sie **User. Read. all** und **Mail. Read** hinzu, und wählen Sie dann **Berechtigungen hinzufügen** aus.</span><span class="sxs-lookup"><span data-stu-id="b63f9-185">Add **User.Read.All** and **Mail.Read** , then select **Add permissions**.</span></span>

1. <span data-ttu-id="b63f9-186">Entfernen Sie in den **konfigurierten Berechtigungen** die Berechtigung Delegierter **Benutzer. Read** unter **Microsoft Graph** , indem Sie rechts neben der Berechtigung die **...** auswählen und die Berechtigung **Entfernen** auswählen.</span><span class="sxs-lookup"><span data-stu-id="b63f9-186">In the **Configured permissions** , remove the delegated **User.Read** permission under **Microsoft Graph** by selecting the **...** to the right of the permission and selecting **Remove permission**.</span></span> <span data-ttu-id="b63f9-187">Wählen Sie **Ja, entfernen** zur Bestätigung aus.</span><span class="sxs-lookup"><span data-stu-id="b63f9-187">Select **Yes, remove** to confirm.</span></span>

1. <span data-ttu-id="b63f9-188">Wählen Sie die Schaltfläche **Administrator Zustimmung erteilen für...** , und klicken Sie dann auf **Ja** , um der Administrator Zustimmung für die konfigurierten Anwendungsberechtigungen zu erteilen.</span><span class="sxs-lookup"><span data-stu-id="b63f9-188">Select the **Grant admin consent for...** button, then select **Yes** to grant admin consent for the configured application permissions.</span></span> <span data-ttu-id="b63f9-189">Die Spalte **Status** in der **konfigurierten Berechtigungs** Tabelle ändert sich in **gewährt für...**.</span><span class="sxs-lookup"><span data-stu-id="b63f9-189">The **Status** column in the **Configured permissions** table changes to **Granted for ...**.</span></span>

    ![Ein Screenshot der konfigurierten Berechtigungen für den webhook mit erteilter Zustimmung des Administrators](./images/webhook-configured-permissions.png)
