
# CBDC Sandbox Monoledger

> Sandbox experimental: esse repositório é um protótipo de desenvolvimento local para
> workflows relacionados à CBDC. Não está pronto para produção e é fornecido "COMO ESTÁ",
> sem quaisquer garantias de segurança, correção ou adequação regulatória. O uso é por sua conta e risco e você é responsável por
> validar qualquer implantação, uso ou redistribuição, incluindo conformidade
> com licenças de software de terceiros para componentes externos, imagens, gráficos,
> e dependências. A menos que um identificador SPDX a nível de arquivo ou aviso de atribuição
> declare o contrário, o código-fonte, documentação, exemplos e artefatos gerados nesse repositório são licenciados sob Apache-2.0. 

Esse é um monorepo para a sandbox local da CBDC. O README raiz é intencionalmente curto: use-o para fazer a sandbox funcionar, depois vá para a documentação específica de cada componente.

## Estrutura do Monorepo

- `infra/`: cluster local, gateway, Besu e infraestrutura compartilhada de implantação
- `services/`: serviços no cluster, como Blockscout, API NB Bond e UI NB
- contracts/: contratos Solidity e workflows Foundry
- `scripts/`: CLIs de referência para workflows off-chain
-  `docs/`: notas de arquitetura, diagramas, runbooks e relatórios

## Documentação

Comece aqui após clonar:

- [visão geral da arquitetura](docs/ARCHITECTURE.md): o que o sandbox contém e como ele é estruturado
- [problemas conhecidos](docs/KNOWN_ISSUES.md): limitações atuais e trabalhos futuros planejados
- [post-mortems](docs/post-mortems/README.md): relatórios de incidentes e histórico de resolução de problemas
- [índice de documentação](docs/DOCUMENTATION_INDEX.md): todas as principais documentações em um só lugar
- [README da infraestrutura](infra/README.md): ponto de entrada da infraestrutura e workflow do registry
- [README dos serviços](services/README.md): visão geral dos serviços e para onde ir a seguir
- [scripts README](scripts/README.md): Visão geral da CLI e dicas de uso
- [contracts README](contracts/README.md): fluxo de trabalho e verificação específicos do contrato
- [guia de contribuição](CONTRIBUTING.md)
- [política de segurança](SECURITY.md)

 ## AI Ready

Este repositório inclui orientações específicas para agentes de codificação de IA. Esses arquivos
também ajudam os colaboradores humanos a entender o workflow e as diretrizes específicas do repositório: 

- `AGENTS.md`: orientação do repositório no nível raiz
- `contracts/AGENTS.md`: orientação de contrato e Foundry
- `infra/AGENTS.md`: orientação de infraestrutura e implantação
- `scripts/AGENTS.md`: orientação de scripts e CLI
- `services/AGENTS.md`: orientação específica de serviço
- `services/nb-ui/AGENTS.md`: orientação do frontend do operador NB UI

 ## Chain Local Atual

A baseline local validada é Besu 26.11.0 com Osaka ativo desde o gênese, um validador QBFT e um nó de arquivo/RPC separado não validador. Solidity 0.8.36 e OpenZeppelin 5.6.1 estão fixados nessa baseline de execução. Todo o tráfego de aplicações locais, Blockscout, Foundry e gateway usa o nó de arquivo/RPC; o validador não é um endpoint de aplicação. 
