---
ms.openlocfilehash: 0c9c6703a54e50f576a171a4a161305c9b9ae6ff
ms.sourcegitcommit: 57a5c2e1a562d8af092a3e78786d711ce1e8f9cb
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 10/30/2020
ms.locfileid: "48822640"
---
<!-- markdownlint-disable MD002 MD041 -->

In dieser Übung erstellen Sie drei neue Azure AD-Anwendungen mithilfe des Azure Active Directory Admin Center:

- Eine APP-Registrierung für die einseitige Anwendung, sodass Sie Benutzer anmelden und Token erhalten kann, die es der Anwendung ermöglichen, die Azure-Funktion aufzurufen.
- Eine APP-Registrierung für die Azure-Funktion, mit der Sie den [im-Auftrag-von-Fluss](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow) zum Austauschen des vom Spa gesendeten Tokens für ein Token verwenden können, mit dem Microsoft Graph aufgerufen werden kann.
- Eine APP-Registrierung für die Azure-Funktion webhook, die es ermöglicht, den [Client-Anmelde Informationsfluss](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow) zu verwenden, um Microsoft Graph ohne Benutzer aufzurufen.

> [!NOTE]
> In diesem Beispiel sind drei App-Registrierungen erforderlich, da Sie sowohl den Fluss "im Auftrag von" als auch den Client-Anmeldeinformationen implementieren. Wenn Ihre Azure-Funktion nur einen dieser Datenflüsse verwendet, müssen Sie nur die APP-Registrierungen erstellen, die diesem Fluss entsprechen.

1. Öffnen Sie einen Browser, und navigieren Sie zum [Azure Active Directory Admin Center](https://aad.portal.azure.com) , und melden Sie sich mit einem Administrator der Microsoft 365-mandantenorganisation an.

1. Wählen Sie in der linken Navigationsleiste **Azure Active Directory** aus, und wählen Sie dann **App-Registrierungen** unter **Verwalten** aus.

    ![Screenshot der APP-Registrierungen ](./images/aad-portal-app-registrations.png)

## <a name="register-an-app-for-the-single-page-application"></a>Registrieren einer APP für die Anwendung mit einer einzelnen Seite

1. Wählen Sie **Neue Registrierung** aus. Legen Sie auf der Seite **Anwendung registrieren** die Werte wie folgt fest.

    - Legen Sie **Name** auf `Graph Azure Function Test App` fest.
    - Legen Sie die **unterstützten Kontotypen** **nur in diesem Organisations Verzeichnis auf Konten** fest.
    - Ändern Sie unter **Umleitungs-URI** das Dropdown in **Single-Page Application (Spa)** , und legen Sie den Wert auf fest `http://localhost:8080` .

    ![Screenshot der Seite "Anwendung registrieren"](./images/register-command-line-app.png)

1. Wählen Sie **Registrieren** aus. Kopieren Sie auf der Seite **Graph Azure Function Test App** die Werte der Anwendungs-ID **(Client) ID** und **Verzeichnis (Mandanten)** , und speichern Sie Sie, dann benötigen Sie Sie in den späteren Schritten.

    ![Screenshot der Anwendungs-ID der neuen App-Registrierung](./images/aad-application-id.png)

## <a name="register-an-app-for-the-azure-function"></a>Registrieren einer APP für die Azure-Funktion

1. Kehren Sie zu **App-Registrierungen** zurück, und wählen Sie **neue Registrierung** aus. Legen Sie auf der Seite **Anwendung registrieren** die Werte wie folgt fest.

    - Legen Sie **Name** auf `Graph Azure Function` fest.
    - Legen Sie die **unterstützten Kontotypen** **nur in diesem Organisations Verzeichnis auf Konten** fest.
    - Lassen Sie die **Umleitungs-URI** leer.

1. Wählen Sie **Registrieren** aus. Kopieren Sie auf der Seite **Diagramm Azure-Funktion** den Wert der **Anwendungs-ID (Client)** , und speichern Sie Sie, um Sie im nächsten Schritt zu benötigen.

1. Wählen Sie unter **Verwalten** die Option **Zertifikate und Geheime Clientschlüssel** aus. Wählen Sie die Schaltfläche **Neuen geheimen Clientschlüssel** aus. Geben Sie einen Wert in **Beschreibung** ein, wählen Sie eine der Optionen für **Gilt bis** aus, und wählen Sie dann **Hinzufügen** aus.

    ![Screenshot des Dialogfelds "Geheimen Clientschlüssel hinzufügen"](./images/aad-new-client-secret.png)

1. Kopieren Sie den Wert des geheimen Clientschlüssels, bevor Sie diese Seite verlassen. Sie benötigen ihn im nächsten Schritt.

    > [!IMPORTANT]
    > Dieser geheime Clientschlüssel wird nicht noch einmal angezeigt, stellen Sie daher sicher, dass Sie ihn jetzt kopieren.

    ![Screenshot des neu hinzugefügten Clientschlüssels](./images/aad-copy-client-secret.png)

1. Wählen Sie unter **Manage** die **API-Berechtigungen** aus. Wählen Sie **Berechtigung hinzufügen** aus.

1. Wählen Sie **Microsoft Graph** und dann **Delegierte Berechtigungen** aus. Fügen Sie **Mail. Read** hinzu, und wählen Sie **Berechtigungen hinzufügen** aus.

    ![Ein Screenshot der konfigurierten Berechtigungen für die Azure-Funktion-App-Registrierung](./images/web-api-configured-permissions.png)

1. Wählen Sie unter **Manage** **eine API verfügbar machen** aus, und wählen Sie dann **Bereich hinzufügen** aus.

1. Akzeptieren Sie den Standard **-URI der Anwendungs-ID** , und wählen Sie **Speichern und Fortfahren**.

1. Füllen Sie das Formular zum **Hinzufügen eines Bereichs** wie folgt aus:

    - **Bereichsname:** Mail. Read
    - **Wer kann zustimmen?:** Administratoren und Benutzer
    - **Anzeigename der Administrator Zustimmung:** Alle Posteingänge aller Benutzer lesen
    - **Beschreibung der Administrator Zustimmung:** Ermöglicht der APP, alle Posteingänge aller Benutzer zu lesen.
    - **Anzeigename der Benutzer Zustimmung:** Lesen Ihres Posteingangs
    - **Beschreibung der Benutzer Zustimmung:** Ermöglicht der APP, Ihren Posteingang zu lesen
    - **Status:** Aktiviert

1. Klicken Sie auf **Bereich hinzufügen**.

1. Kopieren Sie den neuen Bereich, Sie benötigen ihn in späteren Schritten.

    ![Ein Screenshot der definierten Bereiche für die APP-Registrierung in Azure-Funktionen](./images/web-api-defined-scopes.png)

1. Wählen Sie unter **Manage** die Option **Manifest** aus.

1. Suchen `knownClientApplications` Sie im Manifest, und ersetzen Sie den aktuellen Wert von `[]` mit `[TEST_APP_ID]` , wobei `TEST_APP_ID` die Anwendungs-ID der APP-App-Registrierung der **Graph-Azure-Funktions Test** ist. Wählen Sie **Speichern** aus.

> [!NOTE]
> Durch Hinzufügen der APP-ID der Testanwendung zur `knownClientApplications` Eigenschaft im Manifest der Azure-Funktion kann die Testanwendung einen [kombinierten Zustimmungs Fluss](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow#default-and-combined-consent)auslösen. Dies ist erforderlich, damit der "im Auftrag von"-Fluss funktioniert.

## <a name="add-azure-function-scope-to-test-application-registration"></a>Hinzufügen eines Azure-Funktionsbereichs zum Testen der Anwendungsregistrierung

1. Kehren Sie zur APP-Registrierung von **Graph Azure Function Test** zurück, und wählen Sie unter **Manage** die Option **API-Berechtigungen** aus. Wählen Sie **Berechtigung hinzufügen** aus.

1. Wählen Sie **meine APIs** aus, und wählen Sie dann **mehr laden** aus. Wählen Sie **Graph Azure Function** aus.

    ![Screenshot des Dialogfelds "Anforderungs-API-Berechtigungen"](./images/test-app-add-permissions.png)

1. Wählen Sie die Berechtigung **Mail. Read** aus, und wählen Sie **Berechtigungen hinzufügen** aus.

1. Entfernen Sie in den **konfigurierten Berechtigungen** die Berechtigung **User. Read** unter **Microsoft Graph** , indem Sie rechts neben der Berechtigung die **...** auswählen und die Berechtigung **Entfernen** auswählen. Wählen Sie **Ja, entfernen** zur Bestätigung aus.

    ![Ein Screenshot der konfigurierten Berechtigungen für die APP-App-Registrierung](./images/test-app-configured-permissions.png)

## <a name="register-an-app-for-the-azure-function-webhook"></a>Registrieren einer APP für die Azure-Funktion webhook

1. Kehren Sie zu **App-Registrierungen** zurück, und wählen Sie **neue Registrierung** aus. Legen Sie auf der Seite **Anwendung registrieren** die Werte wie folgt fest.

    - Legen Sie **Name** auf `Graph Azure Function Webhook` fest.
    - Legen Sie die **unterstützten Kontotypen** **nur in diesem Organisations Verzeichnis auf Konten** fest.
    - Lassen Sie die **Umleitungs-URI** leer.

1. Wählen Sie **Registrieren** aus. Kopieren Sie auf der Seite **Graph Azure Function webhook** den Wert der **Anwendungs-ID (Client)** , und speichern Sie Sie, um Sie im nächsten Schritt zu benötigen.

1. Wählen Sie unter **Verwalten** die Option **Zertifikate und Geheime Clientschlüssel** aus. Wählen Sie die Schaltfläche **Neuen geheimen Clientschlüssel** aus. Geben Sie einen Wert in **Beschreibung** ein, wählen Sie eine der Optionen für **Gilt bis** aus, und wählen Sie dann **Hinzufügen** aus.

1. Kopieren Sie den Wert des geheimen Clientschlüssels, bevor Sie diese Seite verlassen. Sie benötigen ihn im nächsten Schritt.

1. Wählen Sie unter **Manage** die **API-Berechtigungen** aus. Wählen Sie **Berechtigung hinzufügen** aus.

1. Wählen Sie **Microsoft Graph** und dann **Anwendungsberechtigungen** aus. Fügen Sie **User. Read. all** und **Mail. Read** hinzu, und wählen Sie dann **Berechtigungen hinzufügen** aus.

1. Entfernen Sie in den **konfigurierten Berechtigungen** die Berechtigung Delegierter **Benutzer. Read** unter **Microsoft Graph** , indem Sie rechts neben der Berechtigung die **...** auswählen und die Berechtigung **Entfernen** auswählen. Wählen Sie **Ja, entfernen** zur Bestätigung aus.

1. Wählen Sie die Schaltfläche **Administrator Zustimmung erteilen für...** , und klicken Sie dann auf **Ja** , um der Administrator Zustimmung für die konfigurierten Anwendungsberechtigungen zu erteilen. Die Spalte **Status** in der **konfigurierten Berechtigungs** Tabelle ändert sich in **gewährt für...**.

    ![Ein Screenshot der konfigurierten Berechtigungen für den webhook mit erteilter Zustimmung des Administrators](./images/webhook-configured-permissions.png)
