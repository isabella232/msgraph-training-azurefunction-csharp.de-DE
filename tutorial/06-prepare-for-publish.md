---
ms.openlocfilehash: fbd43b4708cd1aeaad93b70fcf151869bd9e543e
ms.sourcegitcommit: 57a5c2e1a562d8af092a3e78786d711ce1e8f9cb
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 10/30/2020
ms.locfileid: "48822628"
---
<!-- markdownlint-disable MD002 MD041 -->

In dieser Übung erfahren Sie, welche Änderungen für die Azure-Beispielfunktion erforderlich sind, um die [Veröffentlichung in einer Azure-Funktionen-App](https://docs.microsoft.com/azure/azure-functions/functions-run-local#publish)vorzubereiten.

## <a name="update-code"></a>Aktualisieren des Codes

Die Konfiguration wird aus dem geheimen Benutzerspeicher gelesen, der nur für Ihren Entwicklungscomputer gilt. Bevor Sie in Azure veröffentlichen, müssen Sie ändern, wo Sie Ihre Konfiguration speichern, und den Code in **Startup.cs** entsprechend aktualisieren.

Anwendungs Geheimnisse sollten im sicheren Speicher wie [Azure Key Vault](https://docs.microsoft.com/azure/key-vault/general/overview)gespeichert werden.

## <a name="update-cors-setting-for-azure-function"></a>Aktualisieren der CORS-Einstellung für die Azure-Funktion

In diesem Beispiel haben wir CORS in **local.settings.json** so konfiguriert, dass die Testanwendung die Funktion aufrufen kann. Sie müssen Ihre veröffentlichte Funktion so konfigurieren, dass alle Spa-apps zugelassen werden, die Sie aufrufen.

## <a name="update-app-registrations"></a>Aktualisieren von App-Registrierungen

Die  `knownClientApplications` Eigenschaft im Manifest für die APP-Registrierung der **Graph-Azure-Funktion** muss mit den Anwendungs-IDs aller apps aktualisiert werden, die die Azure-Funktion aufrufen werden.

## <a name="recreate-existing-subscriptions"></a>Vorhandene Abonnements neu erstellen

Alle Abonnements, die mit der webhook-URL auf Ihrem lokalen Computer oder ngrok erstellt wurden, sollten mithilfe der Produktions-URL der `Notify` Azure-Funktion neu erstellt werden.
