---
title: 'Azure Databricks: ajuda e perguntas comuns | Microsoft Docs'
description: "Obtenha respostas a perguntas comuns e informações de solução de problemas sobre o Azure Databricks."
services: azure-databricks
documentationcenter: 
author: nitinme
manager: cgronlun
editor: cgronlun
ms.service: azure-databricks
ms.workload: big-data
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/17/2017
ms.author: nitinme
ms.openlocfilehash: fb77ec001f9f52e0a974f8765f458f831fb63908
ms.sourcegitcommit: b07d06ea51a20e32fdc61980667e801cb5db7333
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 12/08/2017
---
# <a name="frequently-asked-questions-about-azure-databricks"></a>Perguntas frequentes a respeito do Azure Databricks

Este artigo lista as principais dúvidas que você pode ter relacionadas ao Azure Databricks. Também lista alguns problemas comuns que você pode enfrentar durante o uso do Azure Databricks. Para obter mais informações, consulte [O que é o Azure Databricks](what-is-azure-databricks.md). 

## <a name="can-i-use-my-own-keys-for-local-encryption"></a>Posso usar minhas próprias chaves para a criptografia local? 
Na versão atual, não há suporte para o uso de chaves próprias do Azure Key Vault. 

## <a name="can-i-use-azure-virtual-networks-with-databricks"></a>Posso usar redes virtuais do Azure com o Databricks?
Uma nova rede virtual é criada como parte do provisionamento do Databricks. Nessa versão, não é possível usar sua própria rede virtual do Azure.

## <a name="how-do-i-access-azure-data-lake-store-from-a-notebook"></a>Como fazer para acessar o Azure Data Lake Store em um bloco de anotações? 

Siga estas etapas:
1. No Azure Active Directory (Azure AD), provisione uma Entidade de Serviço e registre sua chave.
2. Atribua as permissões necessárias à Entidade de Serviço no Azure Data Lake Store.
3. Para acessar um arquivo no Data Lake Store, use as credenciais da Entidade de Serviço no bloco de notas.

Para obter mais informações, consulte [Usar o Data Lake Store com o Azure Databricks](https://docs.azuredatabricks.net/spark/latest/data-sources/azure/azure-storage.html#azure-data-lake-store).

## <a name="fix-common-problems"></a>Solução de problemas comuns

Aqui estão alguns problemas que podem ocorrer com o Databricks.

### <a name="this-subscription-is-not-registered-to-use-the-namespace-microsoftdatabricks"></a>Essa assinatura não está registrada para usar o namespace ‘Microsoft.Databricks’

#### <a name="error-message"></a>Mensagem de erro

"Essa assinatura não está registrada para usar o namespace ‘Microsoft.Databricks’. Consulte https://aka.ms/rps-not-found para saber como registrar assinaturas. (Código: MissingSubscriptionRegistration)"

#### <a name="solution"></a>Solução

1. Vá para o [Portal do Azure](https://portal.azure.com).
2. Clique em **Assinaturas**, na assinatura que você está usando e, em seguida, clique em **Provedores de recursos**. 
3. Na lista de provedores de recursos, em relação a **Microsoft.Databricks**, clique em **Registrar**. Você deve ter a função de Colaborador ou Proprietário na assinatura para registrar o provedor de recursos.


### <a name="your-account-email-does-not-have-the-owner-or-contributor-role-on-the-databricks-workspace-resource-in-the-azure-portal"></a>Sua conta {email} não tem a função Proprietário ou Colaborador no recurso do espaço de trabalho do Databricks no portal do Azure

#### <a name="error-message"></a>Mensagem de erro

Sua conta {email} não tem a função Proprietário ou Colaborador no recurso do espaço de trabalho do Databricks no portal do Azure. Esse erro também poderá ocorrer se você for um usuário convidado no locatário. Solicite ao administrador para conceder acesso a você ou adicioná-lo como um usuário diretamente no espaço de trabalho do Databricks." 

#### <a name="solution"></a>Solução

A seguir, estão algumas soluções para esse problema:

* Para inicializar o locatário, você deve estar conectado como um usuário normal do locatário, não um usuário convidado. Você também deve ter a função Colaborador no recurso do espaço de trabalho do Databricks. Você pode garantir o acesso a um usuário por meio da guia **Controle de acesso (IAM)** no espaço de trabalho do Databricks do portal do Azure.

* Esse erro também poderá ocorrer se o seu nome de domínio de e-mail estiver atribuído a vários diretórios no AD do Azure. Para contornar esse problema, crie um novo usuário no diretório que contém a assinatura com o seu espaço de trabalho do Databricks.

    a. No portal do Azure, vá para o AD do Azure. Selecione **Usuários e Grupos** > **Adicione um usuário**.

    b. Adicione um usuário com um `@<tenant_name>.onmicrosoft.com` email em vez do `@<your_domain>` email. Você pode encontrar isso em **Domínios personalizados**, no AD do Azure no portal do Azure.
    
    c. Conceda essa nova função de usuário de **Colaborador** no recurso do espaço de trabalho do Databricks.
    
    d. Entre no portal do Azure com o novo usuário e localize o espaço de trabalho do Databricks.
    
    e. Inicie o espaço de trabalho do Databricks como esse usuário.


### <a name="your-account-email-has-not-been-registered-in-databricks"></a>Sua conta {email} não foi registrada no Databricks 

#### <a name="solution"></a>Solução

Se você não criou o espaço de trabalho e foi adicionado como um usuário do espaço de trabalho, contate a pessoa que criou o espaço de trabalho. Peça à pessoa que o adicione usando o Console de Administração do Azure Databricks. Para obter instruções, consulte [Adicionando e gerenciando usuários](https://docs.azuredatabricks.net/administration-guide/admin-settings/users.html). Se você criou o espaço de trabalho e ainda recebe esse erro, tente clicar em **Inicializar Espaço de Trabalho** novamente no portal do Azure.

### <a name="cloud-provider-launch-failure-while-setting-up-the-cluster"></a>Falha de inicialização do provedor de nuvem durante a configuração do cluster

#### <a name="error-message"></a>Mensagem de erro

"Falha ao iniciar o provedor de nuvem: Foi encontrado um erro do provedor de nuvem durante a configuração do cluster. Consulte o guia do Databricks para obter mais informações. Código de erro do Azure: PublicIPCountLimitReached. Mensagem de erro do Azure: Não é possível criar mais de 60 endereços IP públicos para esta assinatura nesta região."

#### <a name="solution"></a>Solução

Os clusters do Azure Databricks usam um endereço IP público por nó. Caso sua assinatura já tenha usado todos os seus IPs públicos, [solicite um aumento da cota](https://docs.microsoft.com/azure/azure-supportability/resource-manager-core-quotas-request). Escolha **Cota** como o **Tipo de problema**, e **Rede: ARM** como o **Tipo de cota**. Em **Detalhes**, solicite um aumento de cota de endereço de IP público. Por exemplo, se o limite atual for de 60 e você deseja criar um cluster de 100 nós, solicite um aumento de limite para 160.

### <a name="a-second-type-of-cloud-provider-launch-failure-while-setting-up-the-cluster"></a>Um segundo tipo de falha de inicialização do provedor de nuvem durante a configuração do cluster

#### <a name="error-message"></a>Mensagem de erro

"Falha ao iniciar o provedor de nuvem: Foi encontrado um erro do provedor de nuvem durante a configuração do cluster. Consulte o guia do Databricks para obter mais informações.
Código de erro do Azure: mensagem de erro do Azure MissingSubscriptionRegistration: a assinatura não está registrada para usar o namespace 'Microsoft.Compute'. Consulte https://aka.ms/rps-not-found para saber como registrar assinaturas."

#### <a name="solution"></a>Solução

1. Vá para o [Portal do Azure](https://portal.azure.com).
2. Clique em **Assinaturas**, na assinatura que você está usando e, em seguida, clique em **Provedores de recursos**. 
3. Na lista de provedores de recursos, em relação a **Microsoft.Compute**, clique em **Registrar**. Você deve ter a função de Colaborador ou Proprietário na assinatura para registrar o provedor de recursos.

Consulte [Provedores de recursos e tipos](../azure-resource-manager/resource-manager-supported-services.md) para obter instruções mais detalhadas.

## <a name="next-steps"></a>Próximas etapas

- [Guia de início rápido: Introdução ao Azure Databricks](quickstart-create-databricks-workspace-portal.md)
- [O que é o Azure Databricks?](what-is-azure-databricks.md)

