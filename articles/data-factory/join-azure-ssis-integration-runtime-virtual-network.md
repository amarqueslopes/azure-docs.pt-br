---
title: "Unir o tempo de execução de integração do Azure-SSIS a uma VNET | Microsoft Docs"
description: "Saiba como unir o tempo de execução de integração do Azure-SSIS a uma rede virtual do Azure."
services: data-factory
documentationcenter: 
author: spelluru
manager: jhubbard
editor: monicar
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/27/2017
ms.author: spelluru
ms.openlocfilehash: aa570379890023c83383d291aa5d57fb79b2d5aa
ms.sourcegitcommit: d247d29b70bdb3044bff6a78443f275c4a943b11
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 12/13/2017
---
# <a name="join-an-azure-ssis-integration-runtime-to-a-virtual-network"></a>Unir o tempo de execução de integração do Azure-SSIS a uma rede virtual
Você deve unir o tempo de execução de integração (IR) do Azure-SSIS a uma rede virtual (VNet) do Azure se uma das seguintes condições for verdadeira: 

- Você hospeda o banco de dados de Catálogo do SSIS em uma instância gerenciada do SQL Server (versão prévia privada) que faz parte de uma VNet.
- Você deseja se conectar a armazenamentos de dados locais de pacotes SSIS em execução em um tempo de execução de integração do Azure-SSIS.

 O Azure Data Factory versão 2 (versão prévia) permite que você una o tempo de execução de integração do Azure-SSIS a uma VNet clássica. Atualmente, a VNet do Azure Resource Manager não é compatível. No entanto, você pode contornar essa situação conforme mostrado na seção a seguir. 

 > [!NOTE]
> Este artigo aplica-se à versão 2 do Data Factory, que está atualmente em versão prévia. Se você estiver usando a versão 1 do serviço Data Factory, que está com GA (disponibilidade geral), consulte a [Documentação do Data Factory versão 1](v1/data-factory-introduction.md).

## <a name="access-on-premises-data-stores"></a>Acessar armazenamentos de dados locais
Se os pacotes do SSIS acessam apenas armazenamentos de dados de nuvem pública, você não precisa unir o IR do Azure-SSIS a uma rede virtual. Se os pacotes do SSIS acessam armazenamentos de dados locais, você deve unir o IR do Azure-SSIS a uma VNet que esteja conectada à rede local. Se o catálogo do SSIS estiver hospedado em um banco de dados SQL do Azure que não esteja na VNet, você precisará abrir as portas apropriadas. Se o catálogo do SSIS estiver hospedado na instância gerenciada do SQL edo Azure em uma VNet clássica, você poderá unir o IR do Azure-SSIS à mesma VNet clássica (ou) a uma VNet clássica diferente que tenha conexão entre VNets clássicas com a que tem a instância gerenciada do SQL do Azure. As seções a seguir apresentam mais detalhes.

Alguns pontos importantes a serem considerados: 

- Se não houver nenhuma VNet existente conectada à sua rede local, primeiro crie uma [VNet clássica](../virtual-network/virtual-networks-create-vnet-classic-pportal.md) para unir o tempo de execução de integração do Azure SSIS a ela. Em seguida, configure uma [conexão de gateway VPN site a site](../vpn-gateway/vpn-gateway-howto-site-to-site-classic-portal.md)/[conexão de ExpressRoute](../expressroute/expressroute-howto-linkvnet-classic.md) dessa VNet à sua rede local.
- Se já houver uma VNet clássica conectada à sua rede local no mesmo local que o tempo de execução de integração do Azure-SSIS, você poderá unir o tempo de execução de integração do Azure-SSIS a ela.
- Se já houver uma VNet clássica conectada à sua rede local em um local diferente do tempo de execução de integração do Azure-SSIS, você primeiro tem que criar uma [VNet clássica](../virtual-network/virtual-networks-create-vnet-classic-pportal.md) para unir o tempo de execução de integração do Azure-SSIS a ela. Em seguida, configure uma conexão entre [VNets clássicas](../vpn-gateway/vpn-gateway-howto-vnet-vnet-portal-classic.md).
- Se já houver uma VNet do Azure Resource Manager conectada à sua rede local, primeiro crie uma [VNet clássica](../virtual-network/virtual-networks-create-vnet-classic-pportal.md) para unir o tempo de execução de integração do Azure SSIS a ela. Em seguida, configure uma conexão entre [VNet clássica e Azure Resource Manager](../vpn-gateway/vpn-gateway-connect-different-deployment-models-portal.md).

## <a name="domain-name-services-server"></a>Servidor de Serviços de Nome de Domínio 
Se você precisar usar seu próprio servidor de Serviços de Nome de Domínio (DNS) em uma VNet ingressada por seu tempo de execução de integração do Azure-SSIS, siga a diretriz para [certificar-se de que os nós de seu tempo de execução de integração do Azure-SSIS na VNet possam resolver pontos de extremidade do Azure](../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md#name-resolution-using-your-own-dns-server).

## <a name="network-security-group"></a>Grupo de Segurança de Rede
Se você precisar implementar o Grupo de Segurança de Rede (NSG) em uma VNet unida por seu tempo de execução de integração do Azure-SSIS, permita o tráfego de entrada/saída pelas seguintes portas:

| Portas | Direção | Protocolo de Transporte | Finalidade | Origem da entrada/destino da saída |
| ---- | --------- | ------------------ | ------- | ----------------------------------- |
| 10100<br/>20100<br/>30100  | Entrada | TCP | Os serviços do Azure usam essas portas para se comunicar com os nós de seu tempo de execução de integração do Azure-SSIS na VNet. | Internet | 
| 443 | Saída | TCP | Os nós de seu tempo de execução de integração do Azure-SSIS na VNet usam essa porta para acessar os serviços do Azure como, por exemplo, o Armazenamento do Microsoft Azure, Hub de eventos, etc. | INTERNET | 
| 1433<br/>11000-11999<br/>14000-14999  | Saída | TCP | Os nós de seu tempo de execução de integração do Azure-SSIS na VNet usam essas portas para acessar o SSISDB hospedado pelo servidor de banco de dados SQL do Azure (não aplicável ao SSISDB hospedado pela instância gerenciada do SQL do Azure). | Internet | 

## <a name="configure-vnet"></a>Configurar VNet
Primeiro, você precisa configurar a VNet usando uma das seguintes maneiras (script vs. Portal do Azure) antes de adicionar um IR do Azure-SSIS à VNet. 

### <a name="script-to-configure-vnet"></a>Script para configurar a VNet 
Adicione o script a seguir para configurar automaticamente as permissões/configurações de VNet para que seu Integration Runtime do Azure-SSIS se una à VNet.

```powershell
# Register to Azure Batch resource provider
if(![string]::IsNullOrEmpty($VnetId) -and ![string]::IsNullOrEmpty($SubnetName))
{
    $BatchObjectId = (Get-AzureRmADServicePrincipal -ServicePrincipalName "MicrosoftAzureBatch").Id
    Register-AzureRmResourceProvider -ProviderNamespace Microsoft.Batch
    while(!(Get-AzureRmResourceProvider -ProviderNamespace "Microsoft.Batch").RegistrationState.Contains("Registered"))
    {
    Start-Sleep -s 10
    }
    # Assign VM contributor role to Microsoft.Batch
    New-AzureRmRoleAssignment -ObjectId $BatchObjectId -RoleDefinitionName "Classic Virtual Machine Contributor" -Scope $VnetId
}
```

### <a name="use-portal-to-configure-vnet"></a>Usar o portal para configurar a VNet
A execução do script é a maneira mais fácil de configurar a VNet. Se você não tiver acesso para configurar essa VNet/a configuração automática falhar, o proprietário dessa VNet/você pode tentar configurá-la manualmente nas etapas a seguir:

### <a name="find-the-resource-id-for-your-azure-vnet"></a>Localize a ID de recurso da VNet do Azure.
 
1. Faça logon no [portal do Azure](https://portal.azure.com).
2. Clique em **Mais serviços**. Filtre e selecione **Redes virtuais (clássicas)**.
3. Filtre e selecione a sua **rede virtual** na lista. 
4. Na página Rede virtual (clássica), selecione **Propriedades**. 

    ![ID de recurso da rede virtual clássica](media/join-azure-ssis-integration-runtime-virtual-network/classic-vnet-resource-id.png)
5. Clique no botão de copiar para que a **ID DE RECURSO** copie a ID de recurso da rede clássica para a área de transferência. Salve a ID da área de transferência no OneNote ou em um arquivo.
6. Clique em **Sub-redes** no menu à esquerda e certifique-se de que o número de **endereços disponíveis** seja maior do que os nós em seu tempo de execução de integração do Azure-SSIS.

    ![Número de endereços disponíveis na VNet](media/join-azure-ssis-integration-runtime-virtual-network/number-of-available-addresses.png)
7. Una o **MicrosoftAzureBatch** à função **Colaborador de Máquina Virtual Clássica** da VNet. 
    1. Clique em Controle de acesso (IAM) no menu à esquerda e, em seguida, clique em **Adicionar** na barra de ferramentas.
    
        ![Controle de acesso -> Adicionar](media/join-azure-ssis-integration-runtime-virtual-network/access-control-add.png) 
    2. Na página **Adicionar permissões**, selecione **Colaborador de Máquina Virtual Clássica** como **Função**. Copiar/colar **ddbf3205-c6bd-46ae-8127-60eb93363864** na caixa de texto **Selecione** e, em seguida, selecione **lote do Microsoft Azure** da lista de resultados da pesquisa. 
    
        ![Adicionar permissões - pesquisa](media/join-azure-ssis-integration-runtime-virtual-network/azure-batch-to-vm-contributor.png)
    3. Clique em Salvar para salvar as configurações e para fechar a página.
    
        ![Salvar as configurações de acesso](media/join-azure-ssis-integration-runtime-virtual-network/save-access-settings.png)
    4. Confirme que você está vendo **MicrosoftAzureBatch** na lista de colaboradores.
    
        ![Confirmar o acesso do AzureBatch](media/join-azure-ssis-integration-runtime-virtual-network/azure-batch-in-list.png)
5. Verifique se o provedor do lote do Azure está registrado na assinatura do Azure que tem a VNet ou registre o provedor de lote do Azure. Se você já tiver uma conta de lote do Azure em sua assinatura, sua assinatura está registrada para o lote do Azure.
    1. No portal do Azure, clique em **Assinaturas** no menu à esquerda. 
    2. Selecione sua **assinatura**. 
    3. Clique em **Provedores de recursos** à esquerda e confirme se `Microsoft.Batch` é um provedor registrado. 
    
        ![confirmation-batch-registered](media/join-azure-ssis-integration-runtime-virtual-network/batch-registered-confirmation.png)

    Se você não vir `Microsoft.Batch` na lista, [crie uma conta vazia de lote do Azure](../batch/batch-account-create-portal.md) em sua assinatura para registrá-lo. Você pode excluí-lo depois. 

## <a name="create-an-azure-ssis-ir-and-join-it-to-a-vnet"></a>Crie um IR do Azure-SSIS e adicione-o a uma VNet
Você pode criar um IR do Azure-SSIS e adicioná-lo à VNet ao mesmo tempo. Para o script completo e instruções para criar um IR do Azure-SSIS e adicioná-lo a uma VNet ao mesmo tempo, consulte [Criar IR do Azure-SSIS](create-azure-ssis-integration-runtime.md).

## <a name="join-an-existing-azure-ssis-ir-to-a-vnet"></a>Adicionar um IR do Azure-SSIS a uma VNet
O script no artigo [Criar Integration Runtime do Azure-SSIS](create-azure-ssis-integration-runtime.md) mostra como criar um IR do Azure-SSIS e adicioná-lo a uma VNet no mesmo script. Se você tiver um Azure-SSIS existente, execute as seguintes etapas para adicioná-lo para a VNet. 

1. Interrompa o IR do Azure-SSIS.
2. Configure o IR do Azure-SSIS para adicionar à VNet. 
3. Inicie o IR do Azure-SSIS. 

## <a name="define-the-variables"></a>Defina as variáveis

```powershell
$ResourceGroupName = "<Azure resource group name>"
$DataFactoryName = "<Data factory name>" 
$AzureSSISName = "<Specify Azure-SSIS IR name>"
# Get the following information from the properties page for your Classic Virtual Network in the Azure portal
# It should be in the format: 
# $VnetId = "/subscriptions/<Azure Subscription ID>/resourceGroups/<Azure Resource Group>/providers/Microsoft.ClassicNetwork/virtualNetworks/<Class Virtual Network Name>"
$VnetId = "<Name of your Azure classic virtual netowrk>"
$SubnetName = "<Name of the subnet in VNet>"
```

### <a name="stop-the-azure-ssis-ir"></a>Interrompa o IR do Azure-SSIS
Interrompa o Integration Runtime do Azure SSIS antes de associá-lo a uma VNet. Este comando libera todos os seus nós e interrompe a cobrança.

```powershell
Stop-AzureRmDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
                                             -DataFactoryName $DataFactoryName `
                                             -Name $AzureSSISName `
                                             -Force 
```
### <a name="configure-vnet-settings-for-the-azure-ssis-ir-to-join"></a>Defina configurações de VNet para que o IR do Azure-SSIS IR seja associado
Registre o provedor de recursos do lote do Azure:

```powershell
if(![string]::IsNullOrEmpty($VnetId) -and ![string]::IsNullOrEmpty($SubnetName))
{
    $BatchObjectId = (Get-AzureRmADServicePrincipal -ServicePrincipalName "MicrosoftAzureBatch").Id
    Register-AzureRmResourceProvider -ProviderNamespace Microsoft.Batch
    while(!(Get-AzureRmResourceProvider -ProviderNamespace "Microsoft.Batch").RegistrationState.Contains("Registered"))
    {
        Start-Sleep -s 10
    }
    # Assign VM contributor role to Microsoft.Batch
    New-AzureRmRoleAssignment -ObjectId $BatchObjectId -RoleDefinitionName "Classic Virtual Machine Contributor" -Scope $VnetId
}
```

### <a name="configure-the-azure-ssis-ir"></a>Configure o IR do Azure-SSIS
Execute o comando Set-AzureRmDataFactoryV2IntegrationRuntime para configurar o Integration Runtime do Azure-SSIS para associar à VNet: 

```powershell
Set-AzureRmDataFactoryV2IntegrationRuntime  -ResourceGroupName $ResourceGroupName `
                                            -DataFactoryName $DataFactoryName `
                                            -Name $AzureSSISName `
                                            -Type Managed `
                                            -VnetId $VnetId `
                                            -Subnet $SubnetName
```

## <a name="start-the-azure-ssis-ir"></a>Inicie o IR do Azure-SSIS
Execute o comando a seguir para iniciar o Integration Runtime do Azure-SSIS: 

```powershell
Start-AzureRmDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
                                             -DataFactoryName $DataFactoryName `
                                             -Name $AzureSSISName `
                                             -Force

```
Esse comando o levará de **20 a 30 minutos** para concluir.

## <a name="next-steps"></a>Próximas etapas
Confira estes tópicos para obter mais informações sobre o tempo de execução do Azure-SSIS: 

- [Tempo de execução de integração do Azure-SSIS](concepts-integration-runtime.md#azure-ssis-integration-runtime). Este artigo fornece informações conceituais sobre tempos de execução de integração em geral, incluindo o IR do Azure-SSIS. 
- [Tutorial: implantar pacotes do SSIS para o Azure](tutorial-deploy-ssis-packages-azure.md). Este artigo fornece instruções passo a passo para criar um IR do Azure-SSIS e usa um banco de dados SQL do Azure para hospedar o catálogo do SSIS. 
- [Como: Criar um tempo de execução de integração do Azure-SSIS](create-azure-ssis-integration-runtime.md). Este artigo expande o tutorial e fornece instruções sobre como usar a Instância Gerenciada do SQL do Azure (visualização particular) e unir o IR para uma VNet. 
- [Monitore um IR do Azure-SSIS](monitor-integration-runtime.md#azure-ssis-integration-runtime). Este artigo mostra como recuperar informações sobre um IR do Azure-SSIS e descrições de status nas informações retornadas. 
- [Gerencie um IR do Azure-SSIS](manage-azure-ssis-integration-runtime.md). Este artigo mostra como parar, iniciar ou remover um IR do Azure-SSIS. Ele também mostra como expandir o IR do Azure-SSIS adicionando mais nós ao IR. 
