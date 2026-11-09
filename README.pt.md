
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

Blocos com transações utilizam um período de um segundo. Quando o pool de transações está vazio, o QBFT produz um bloco vazio apenas a cada cinco minutos para evitar que este sandbox de baixo volume acumule principalmente um histórico vazio.

Essa topologia de validador único é determinística, mas não é tolerante a falhas bizantinas. Não possui cliente beacon ou perfil de bootnode. Veja [ADR 0003](docs/decisions/0003-adopt-besu-qbft-osaka-with-archive-rpc.md) e o [plano de implementação arquivado](docs/plans/archive/besu-qbft-osaka-upgrade-plan.md) para a decisão, restrições de migração e evidências de aceitação.

## Configuração Rápida

Os comandos devem ser executados no Linux ou macOS. No Windows, use o WSL.

1. Instale o Docker.
2. Instale o Foundry e use a versão estável mais recente:

```console
curl -L https://foundry.paradigm.xyz | bash
foundryup
```
3. Instale as ferramentas locais:

```console
# Exemplo Ubuntu/Debian
go install sigs.k8s.io/kind@v0.33.0
sudo snap install kubectl --classic
sudo snap install helm --classic
sudo snap install yq
sudo apt install jq
```

```console
# macOS (Homebrew)
brew install kind kubectl helm yq jq
```

4. Gere os arquivos de fixture locais necessários:

```console
node scripts/generate-local-sandbox-fixtures.mjs
```

Isso escreve arquivos locais ignorados para `contracts/.env`, `services/nb-bond-api/helm/values.local.yaml`, `scripts/bid-submitter/examples/bids.keys.json` e os inputs da criptografia de ofertas executáveis em`.tmp/bid-encryption/examples/`.

Os scripts de start também geram esses arquivos automaticamente se estiverem faltando. Eles são para uso local em sandbox apenas e nunca devem ser reutilizados fora do desenvolvimento local.

5. Instale as dependências do contrato:

```console
cd contracts
forge soldeer install
cd ..
```
6. Inicie o registry local por onde o sandbox obtém todas as imagens:

```console
./infra/infra.sh registry-start
```

O contêiner do registry persiste ao longo dos ciclos de vida do cluster, então essa é uma configuração única (e uma operação sem efeito). `./sandbox.sh start` faz o pull e o push de quaisquer imagens ausentes para esse registry sob demanda via `loadImageToKind`.

As tags de lançamento atuais do backend/frontend do Blockscout não são publicadas como imagens disponíveis para pull upstream. Em uma máquina nova ou após `registry-reset`, construa as fontes fixadas no registry local antes de `registry-sync` ou `start`:

```console
./sandbox.sh build-images
```
Opcionalmente, pré-aqueça o registry com cada imagem de terceiros fixada para que a primeira implantação não pague o custo de pull durante a execução":

```console
./infra/infra.sh registry-sync
```

Os pins das imagens de deploy/build do Sandbox estão centralizados em `common/images.yaml`. O pin da toolchain compartilhada do Node.js está centralizado em `common/node-version.env`.

7. Opcional: gere o arquivo de configuração do sandbox e edite as flags de deploy em
   `.env.sandbox` se você não quiser a stack completa. Isso escreve o arquivo na
   raiz do repositório, e você pode editá-lo lá antes de executar
   `./sandbox.sh start`:

```console
./sandbox.sh generate-config
```

As flags típicas em `.env.sandbox` incluem `DEPLOY_INFRA`, `DEPLOY_BLOCKSCOUT`, `DEPLOY_CONTRACTS`, `DEPLOY_VERIFY_CONTRACTS`, `DEPLOY_SKIP_SIMULATION`, `DEPLOY_NB_BOND_API` e `DEPLOY_NB_UI`. Se você pular essa etapa, o workflow padrão do nível root será utilizado.


8. Inicie o sandbox:

```console
./sandbox.sh start
```

Se um arquivo local necessário estiver ausente, `./sandbox.sh start` interrompe a execução e exibe o comando de cópia para o arquivo de exemplo correspondente.

./sandbox.sh start` tentará adicionar as entradas de host necessárias `*.cbdc-sandbox.local` no Linux/macOS. Se você preferir não editar arquivos de hosts, ou se estiver no Windows/WSL, adicione as entradas de host manualmente ou use `kubectl port-forward` no serviço específico que você precisa.


Se o script não atualizar seu arquivo de hosts, adicione essas entradas manualmente:

```text
127.0.0.1 besu.cbdc-sandbox.local
127.0.0.1 blockscout.cbdc-sandbox.local
127.0.0.1 bond-api.cbdc-sandbox.local
127.0.0.1 web.cbdc-sandbox.local
```
Use `/etc/hosts` no Linux/macOS ou
`C:\Windows\System32\drivers\etc\hosts` no Windows.

## Comandos do Sandbox


| Comando | Propósito |
| --- | --- |
| `./sandbox.sh start` | Crie ou atualize o sandbox local; continue um sandbox parado com `stop` |
| `./sandbox.sh stop` | Pare todo o sandbox de forma graciosa e preserve todo o estado: a chain, o índice e as verificações de contrato do Blockscout e o banco de dados da API NB Bond. O comando start restaura o ambiente exatamente como estava. Execute-o antes de fechar o Docker ou reiniciar a máquina; consulte [unclean shutdowns](docs/KNOWN_ISSUES.md#sandbox-must-be-stopped-before-docker-quits-or-the-host-reboots). O nó do Kind recebe SANDBOX_STOP_TIMEOUT_SECONDS (padrão: 330) para desligar; o encerramento leva cerca de 90 segundos |
| `./sandbox.sh delete` | Destrua o cluster do Kind e todo o estado do sandbox; execute delete e depois start para gerar um sandbox totalmente limpo. O contêiner do registry local e suas imagens em cache são mantidos (eles residem no contêiner Docker kind-registry, que é independente do ciclo de vida do cluster Kind). Para recuperar esse espaço, execute ./sandbox.sh registry-reset (ou docker rm -f kind-registry). |
| `./sandbox.sh generate-config` | Create `.env.sandbox` with deploy toggles |
| `./infra/infra.sh registry-start` | Start the local registry container (one-time setup; no-op if already running, unless the registry pin in `common/images.yaml` changed, which recreates it and re-syncs images) |
| `./infra/infra.sh registry-sync` | Optional pre-warm: push every pinned third-party image into the local registry up front so `sandbox.sh start` doesn't pull them on demand |
| `./sandbox.sh image-report` | Read-only report: running pod images, local registry tags per service, and which tag is the current build / deployed |
| `./sandbox.sh cleanup-images` | Prune old content-hash images for `nb-ui` / `nb-bond-api` / `bens-microservice` (keeps current + 2 newest and the deployed tag; never removes shared base images). `--keep N` changes retention; `--prune-build-cache` also clears the global Docker build cache |
| `./sandbox.sh registry-reset` | Recreate the `kind-registry` container and re-sync base images, clearing accumulated registry tags. Repo-owned images rebuild on the next start |
| `./sandbox.sh build-images` | Build the pinned Blockscout backend/frontend source tags and push them into the local registry; required once on a fresh machine or after `registry-reset` |





