O LRS (armazenamento com redundância local) é projetado para fornecer pelo menos 99,999999999% (11 9's) de durabilidde de objetos em um determinado ano replicando seus dados dentro de uma unidade de escala de armazenamento, que é hospedada em um centro de dados na região em que você criou a conta de armazenamento. Uma solicitação de gravação retorna com êxito somente depois que tiver sido gravada em todas as réplicas. Essas réplicas residem em domínios de falha e domínios de atualização separados dentro de uma unidade de escala de armazenamento.

Uma unidade de escala de armazenamento é uma coleção de racks de nós de armazenamento. Um domínio de falha (FD) é um grupo de nós que representam uma unidade física de falha e podem ser considerados nós que pertencem ao mesmo rack físico. Um domínio de atualização (UD) é um grupo de nós atualizados em conjunto durante o processo de atualização de um serviço (distribuição). As réplicas estão difundidas entre UDs e FDs em uma única unidade de escala de armazenamento para garantir que os dados estejam disponíveis, mesmo se a falha de hardware afetar um único rack ou quando os nós forem atualizados durante uma distribuição.

O LRS é a opção de custo mais baixo e oferece durabilidade menor em comparação com outras opções. Em caso de desastre no nível de datacenter (incêndio, inundação etc.), as réplicas podem ser perdidas ou ficar irrecuperáveis. Para reduzir esse risco, o GRS (armazenamento com redundância geográfica) é a opção recomendada para a maioria dos aplicativos.

O armazenamento com redundância local ainda pode ser desejável em determinados cenários:

* Fornece largura de banda máxima das opções de replicação do Armazenamento do Azure.
* Se seu aplicativo armazenar dados que possam ser facilmente reconstruídos, você pode optar por LRS.
* Alguns aplicativos são restritos à replicação de dados somente em um país devido a requisitos de governança de dados. Uma região emparelhada poderia estar em outro país. Para obter mais informações sobre pares de regiões, consulte [Regiões do Azure](https://azure.microsoft.com/regions/).
