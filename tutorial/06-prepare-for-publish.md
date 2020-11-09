---
ms.openlocfilehash: fbd43b4708cd1aeaad93b70fcf151869bd9e543e
ms.sourcegitcommit: 57a5c2e1a562d8af092a3e78786d711ce1e8f9cb
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 10/30/2020
ms.locfileid: "48822628"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="3ff06-101">In dieser Übung erfahren Sie, welche Änderungen für die Azure-Beispielfunktion erforderlich sind, um die [Veröffentlichung in einer Azure-Funktionen-App](https://docs.microsoft.com/azure/azure-functions/functions-run-local#publish)vorzubereiten.</span><span class="sxs-lookup"><span data-stu-id="3ff06-101">In this exercise you'll learn about what changes are needed to the sample Azure Function to prepare for [publishing to an Azure Functions app](https://docs.microsoft.com/azure/azure-functions/functions-run-local#publish).</span></span>

## <a name="update-code"></a><span data-ttu-id="3ff06-102">Aktualisieren des Codes</span><span class="sxs-lookup"><span data-stu-id="3ff06-102">Update code</span></span>

<span data-ttu-id="3ff06-103">Die Konfiguration wird aus dem geheimen Benutzerspeicher gelesen, der nur für Ihren Entwicklungscomputer gilt.</span><span class="sxs-lookup"><span data-stu-id="3ff06-103">Configuration is read from the user secret store, which only applies to your development machine.</span></span> <span data-ttu-id="3ff06-104">Bevor Sie in Azure veröffentlichen, müssen Sie ändern, wo Sie Ihre Konfiguration speichern, und den Code in **Startup.cs** entsprechend aktualisieren.</span><span class="sxs-lookup"><span data-stu-id="3ff06-104">Before you publish to Azure, you'll need to change where you store your configuration, and update the code in **Startup.cs** accordingly.</span></span>

<span data-ttu-id="3ff06-105">Anwendungs Geheimnisse sollten im sicheren Speicher wie [Azure Key Vault](https://docs.microsoft.com/azure/key-vault/general/overview)gespeichert werden.</span><span class="sxs-lookup"><span data-stu-id="3ff06-105">Application secrets should be stored in secure storage, such as [Azure Key Vault](https://docs.microsoft.com/azure/key-vault/general/overview).</span></span>

## <a name="update-cors-setting-for-azure-function"></a><span data-ttu-id="3ff06-106">Aktualisieren der CORS-Einstellung für die Azure-Funktion</span><span class="sxs-lookup"><span data-stu-id="3ff06-106">Update CORS setting for Azure Function</span></span>

<span data-ttu-id="3ff06-107">In diesem Beispiel haben wir CORS in **local.settings.json** so konfiguriert, dass die Testanwendung die Funktion aufrufen kann.</span><span class="sxs-lookup"><span data-stu-id="3ff06-107">In this sample we configured CORS in **local.settings.json** to allow the test application to call the function.</span></span> <span data-ttu-id="3ff06-108">Sie müssen Ihre veröffentlichte Funktion so konfigurieren, dass alle Spa-apps zugelassen werden, die Sie aufrufen.</span><span class="sxs-lookup"><span data-stu-id="3ff06-108">You'll need to configure your published function to allow any SPA apps that will call it.</span></span>

## <a name="update-app-registrations"></a><span data-ttu-id="3ff06-109">Aktualisieren von App-Registrierungen</span><span class="sxs-lookup"><span data-stu-id="3ff06-109">Update app registrations</span></span>

<span data-ttu-id="3ff06-110">Die  `knownClientApplications` Eigenschaft im Manifest für die APP-Registrierung der **Graph-Azure-Funktion** muss mit den Anwendungs-IDs aller apps aktualisiert werden, die die Azure-Funktion aufrufen werden.</span><span class="sxs-lookup"><span data-stu-id="3ff06-110">The  `knownClientApplications` property in the manifest for the **Graph Azure Function** app registration will need to be updated with the application IDs of any apps that will be calling the Azure Function.</span></span>

## <a name="recreate-existing-subscriptions"></a><span data-ttu-id="3ff06-111">Vorhandene Abonnements neu erstellen</span><span class="sxs-lookup"><span data-stu-id="3ff06-111">Recreate existing subscriptions</span></span>

<span data-ttu-id="3ff06-112">Alle Abonnements, die mit der webhook-URL auf Ihrem lokalen Computer oder ngrok erstellt wurden, sollten mithilfe der Produktions-URL der `Notify` Azure-Funktion neu erstellt werden.</span><span class="sxs-lookup"><span data-stu-id="3ff06-112">Any subscriptions created using the webhook URL on your local machine or ngrok should be recreated using the production URL of the `Notify` Azure Function.</span></span>
