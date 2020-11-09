---
ms.openlocfilehash: 1e4267a5d8bdfe02dbc0170122113c35df822659
ms.sourcegitcommit: 57a5c2e1a562d8af092a3e78786d711ce1e8f9cb
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 10/30/2020
ms.locfileid: "48822613"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="1a1ba-101">In diesem Lernprogramm erfahren Sie, wie Sie eine Azure-Funktion erstellen, die die Microsoft Graph-API zum Abrufen von Kalenderinformationen für einen Benutzer verwendet.</span><span class="sxs-lookup"><span data-stu-id="1a1ba-101">This tutorial teaches you how to build an Azure Function that uses the Microsoft Graph API to retrieve calendar information for a user.</span></span>

> [!TIP]
> <span data-ttu-id="1a1ba-102">Wenn Sie das fertige Lernprogramm einfach herunterladen möchten, können Sie das GitHub- [Repository](https://github.com/microsoftgraph/msgraph-training-azurefunction-csharp)herunterladen oder Klonen.</span><span class="sxs-lookup"><span data-stu-id="1a1ba-102">If you prefer to just download the completed tutorial, you can download or clone the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-azurefunction-csharp).</span></span> <span data-ttu-id="1a1ba-103">Anweisungen zum Konfigurieren der APP mit einer APP-ID und einem geheimen Schlüssel finden Sie in der Readme-Datei im **Demo** -Ordner.</span><span class="sxs-lookup"><span data-stu-id="1a1ba-103">See the README file in the **demo** folder for instructions on configuring the app with an app ID and secret.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="1a1ba-104">Voraussetzungen</span><span class="sxs-lookup"><span data-stu-id="1a1ba-104">Prerequisites</span></span>

<span data-ttu-id="1a1ba-105">Bevor Sie dieses Lernprogramm starten, sollten Sie die folgenden Tools auf dem Entwicklungscomputer installiert haben.</span><span class="sxs-lookup"><span data-stu-id="1a1ba-105">Before you start this tutorial, you should have the following tools installed on your development machine.</span></span>

- [<span data-ttu-id="1a1ba-106">.Net Core SDK</span><span class="sxs-lookup"><span data-stu-id="1a1ba-106">.NET Core SDK</span></span>](https://dotnet.microsoft.com/download)
- [<span data-ttu-id="1a1ba-107">Kern Tools für Azure-Funktionen</span><span class="sxs-lookup"><span data-stu-id="1a1ba-107">Azure Functions Core Tools</span></span>](https://docs.microsoft.com/azure/azure-functions/functions-run-local)
- [<span data-ttu-id="1a1ba-108">Azure-CLI</span><span class="sxs-lookup"><span data-stu-id="1a1ba-108">Azure CLI</span></span>](https://docs.microsoft.com/cli/azure/install-azure-cli)
- [<span data-ttu-id="1a1ba-109">ngrok</span><span class="sxs-lookup"><span data-stu-id="1a1ba-109">ngrok</span></span>](https://ngrok.com/)

<span data-ttu-id="1a1ba-110">Sie sollten auch über ein Microsoft-Büro-oder Schulkonto verfügen, das über Zugriff auf ein globales Administratorkonto in derselben Organisation verfügt.</span><span class="sxs-lookup"><span data-stu-id="1a1ba-110">You should also have a Microsoft work or school account, with access to a global administrator account in the same organization.</span></span> <span data-ttu-id="1a1ba-111">Wenn Sie kein Microsoft-Konto haben, können Sie [sich für das Office 365 Entwicklerprogramm registrieren](https://developer.microsoft.com/office/dev-program) , um ein kostenloses Office 365-Abonnement zu erhalten.</span><span class="sxs-lookup"><span data-stu-id="1a1ba-111">If you don't have a Microsoft account, you can [sign up for the Office 365 Developer Program](https://developer.microsoft.com/office/dev-program) to get a free Office 365 subscription.</span></span>

> [!NOTE]
> <span data-ttu-id="1a1ba-112">Dieses Lernprogramm wurde mit den folgenden Versionen der oben genannten Tools geschrieben.</span><span class="sxs-lookup"><span data-stu-id="1a1ba-112">This tutorial was written with the following versions of the above tools.</span></span> <span data-ttu-id="1a1ba-113">Die Schritte in diesem Leitfaden funktionieren möglicherweise mit anderen Versionen, jedoch nicht getestet.</span><span class="sxs-lookup"><span data-stu-id="1a1ba-113">The steps in this guide may work with other versions, but that has not been tested.</span></span>
>
> - <span data-ttu-id="1a1ba-114">.Net Core SDK-3.1.301</span><span class="sxs-lookup"><span data-stu-id="1a1ba-114">.NET Core SDK 3.1.301</span></span>
> - <span data-ttu-id="1a1ba-115">Azure-Funktionen Kern Tools 3.0.2630</span><span class="sxs-lookup"><span data-stu-id="1a1ba-115">Azure Functions Core Tools 3.0.2630</span></span>
> - <span data-ttu-id="1a1ba-116">Azure CLI 2.8.0</span><span class="sxs-lookup"><span data-stu-id="1a1ba-116">Azure CLI 2.8.0</span></span>
> - <span data-ttu-id="1a1ba-117">ngrok 2.3.35</span><span class="sxs-lookup"><span data-stu-id="1a1ba-117">ngrok 2.3.35</span></span>

## <a name="feedback"></a><span data-ttu-id="1a1ba-118">Feedback</span><span class="sxs-lookup"><span data-stu-id="1a1ba-118">Feedback</span></span>

<span data-ttu-id="1a1ba-119">Geben Sie Feedback zu diesem Lernprogramm im [GitHub-Repository](https://github.com/microsoftgraph/msgraph-training-azurefunction-csharp)an.</span><span class="sxs-lookup"><span data-stu-id="1a1ba-119">Please provide any feedback on this tutorial in the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-azurefunction-csharp).</span></span>
