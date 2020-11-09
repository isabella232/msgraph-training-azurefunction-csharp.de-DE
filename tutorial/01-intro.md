---
ms.openlocfilehash: 1e4267a5d8bdfe02dbc0170122113c35df822659
ms.sourcegitcommit: 57a5c2e1a562d8af092a3e78786d711ce1e8f9cb
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 10/30/2020
ms.locfileid: "48822613"
---
<!-- markdownlint-disable MD002 MD041 -->

In diesem Lernprogramm erfahren Sie, wie Sie eine Azure-Funktion erstellen, die die Microsoft Graph-API zum Abrufen von Kalenderinformationen für einen Benutzer verwendet.

> [!TIP]
> Wenn Sie das fertige Lernprogramm einfach herunterladen möchten, können Sie das GitHub- [Repository](https://github.com/microsoftgraph/msgraph-training-azurefunction-csharp)herunterladen oder Klonen. Anweisungen zum Konfigurieren der APP mit einer APP-ID und einem geheimen Schlüssel finden Sie in der Readme-Datei im **Demo** -Ordner.

## <a name="prerequisites"></a>Voraussetzungen

Bevor Sie dieses Lernprogramm starten, sollten Sie die folgenden Tools auf dem Entwicklungscomputer installiert haben.

- [.Net Core SDK](https://dotnet.microsoft.com/download)
- [Kern Tools für Azure-Funktionen](https://docs.microsoft.com/azure/azure-functions/functions-run-local)
- [Azure-CLI](https://docs.microsoft.com/cli/azure/install-azure-cli)
- [ngrok](https://ngrok.com/)

Sie sollten auch über ein Microsoft-Büro-oder Schulkonto verfügen, das über Zugriff auf ein globales Administratorkonto in derselben Organisation verfügt. Wenn Sie kein Microsoft-Konto haben, können Sie [sich für das Office 365 Entwicklerprogramm registrieren](https://developer.microsoft.com/office/dev-program) , um ein kostenloses Office 365-Abonnement zu erhalten.

> [!NOTE]
> Dieses Lernprogramm wurde mit den folgenden Versionen der oben genannten Tools geschrieben. Die Schritte in diesem Leitfaden funktionieren möglicherweise mit anderen Versionen, jedoch nicht getestet.
>
> - .Net Core SDK-3.1.301
> - Azure-Funktionen Kern Tools 3.0.2630
> - Azure CLI 2.8.0
> - ngrok 2.3.35

## <a name="feedback"></a>Feedback

Geben Sie Feedback zu diesem Lernprogramm im [GitHub-Repository](https://github.com/microsoftgraph/msgraph-training-azurefunction-csharp)an.
