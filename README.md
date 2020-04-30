- Práticas usadas no TJMT para atender a abordagem de Continuous Integration / Continuous Deployment, usando docker.

# Continuous Deployment usando Docker

<details>
  <summary>Contextualização</summary>

Muitas instituições usam ferramentas (Jenkins, TFS, etc) para automatizar as fases de publicação de um software. Nelas, normalmente ficam informações como "comando (tasks) para baixar dependências, compilar, testar, publicar, etc" assim como configurações pertinentes a tecnologia do projeto ("JAVA, .NET, Node, etc").

Muitas vezes este método funciona bem, porém exige a necessidade de que uma equipe (muitas vezes diferente) faça todo o papel de se configurar a infraestrutura necessária para que cada etapa funcione tais como "máquinas virtuais onde será publicado o software (servidor de aplicação)", "configuração na ferramenta de automação (criação dos comandos) para o software", entre outras particularidades da aplicação para o seu ambiente.

Em um cenário onde as aplicações estão ficando cada vez mais difundidas e pequenas (microserviços), cria uma alta demanda para criação de todo esse processo para cada peça de software. Aliado ao fato de que as demandas por resultado de TI (especialmente criação e desenvolvimento de soluções) são cada vez mais velozes, faz com que busquemos meios para facilitar e/ou aprimorar toda essa etapa (criação da automação).

</details>

<details>
  <summary>Objetivo (Vantagens)</summary>

Utilizar docker no desenvolvimento pode proporcionar múltiplas vantagens, porém, nem sempre, estas são utilizadas.

_Considere que Docker de forma geral é uma tecnologia de criação/execução de imagens (algo como uma template de máquina virtual) e criação/execução de ambientes._

> Normalmente, a utilização do docker é vista somente para a publicação do software. É o típico cenário em que o desenvolvedor copia "somente o binário" (já compilado em sua máquina ou na ferramenta de automação) para dentro da imagem e publica esta. Porém, faz com que "a máquina do desenvolvedor ou a ferramenta de automação necessitem das ferramentas de desenvolvimento instaladas" e consequentemente de alguém (ou equipe) para gerenciar essa infraestrutura (no caso da ferramenta de automação), além de que cria um acoplamento nesta (a partir do momento em que se cria nela a configuração/execução das etapas necessárias).

Seguem alguns pontos onde o Docker facilita em todo este processo:
- Criação do processo de compilação do software (via Dockerfile multi-stage)
  > Permite que o mesmo Dockerfile que é utilizado para se compilar a aplicação, seja utilizado na ferramenta de automação
- Criação do processo de execução do teste automatizado (via Dockerfile multi-stage)
  > Permite que o mesmo Dockerfile que é utilizado para se compilar a aplicação, também opcionalmente faça a execução dos testes automatizados (Unitários ou de Integração) 
- Criação do processo de publicação da aplicação (docker-compose)
  > Permite que seja descrito (de forma declarativa) como deve ser criado o ambiente
- Explicitação da aplicação quanto a suas fronteiras (docker-compose)
  > Permite que a configuração de integrações/fronteiras seja feito no arquivo de configuração do ambiente (docker-compose), explicitando suas dependências/integrações

</details>


<details>
  <summary>Imagem</summary>

Para uma imagem de aplicação em docker, existem 3 formas de se utilizar/configurar:
- **Imagem por ambiente** (não recomendado): Onde cada imagem já vem com as configurações específicas para um ambiente em questão. Ou seja, as configurações estão dentro do container (Ex: web.config, application.properties, etc) e sua mudança necessidta da criação de uma nova imagem.
  > Em um ambiente de Integração Contínua, isto impede que uma mesma imagem passe pelas fases de homologação/qualidade do produto. Fazendo que para cada fase, deva-se criar uma nova imagem.
- **Imagem com todas as configurações** de todos os ambientes (não recomendado): Onde a imagem possui as configurações de todos os ambientes (utilizados no processo de desenvolvimento).
  > Este modelo impede que uma imagem possa ser reutilizada em uma infraestrutura diferente, pois nela já contém as configurações de todos os possíveis ambientes.
- **Imagem configurável** (recomendado): As configurações ficam a nível de **variáveis de ambiente**, possibilitando assim que possa ser criado um docker-compose informando as mesmas.
  > Permite que a imagem trafegue pelos ambientes de Integração Contínua e que seja modificado quando em uma infraestrutura diferente da qual foi concebida.

</details>

## Como Funciona?

O repositório GIT deve conter os seguintes arquivos em sua raiz:
- [`Dockerfile`](./docs/dockerfile.md)
- [`docker-compose.yml`](./docs/docker-compose.yml.md)
- [`docker-compose.override.yml`](./docs/docker-compose.override.yml.md)
- Continuous Integration (CI)
  - [`docker-compose.ci-tests.yml`](./docs/docker-compose.ci-tests.yml.md)
  - [`docker-compose.ci-debug.yml`](./docs/docker-compose.ci-debug.yml.md)
  - [`docker-compose.ci-build.yml`](./docs/docker-compose.ci-build.yml.md)
  - [`docker-compose.ci-runtime.yml`](./docs/docker-compose.ci-runtime.yml.md)
  - [`docker-compose.ci-deploy.yml`](./docs/docker-compose.ci-deploy.yml.md) (Opcional)
  - [`ci.sh`](./docs/ci.sh.md) (Executor da pipeline)
  - [`ci-pre-<stage>.sh`](./docs/ci-pre-stage.sh.md) (Hooks de pré-execucação dos estágios - Opcional)
  - [`ci-post-<stage>.sh`](./docs/ci-post-stage.sh.md) (Hooks de pós-execucação dos estágios - Opcional)
- Continuos Delivery (CD)
  - [`docker-compose.cd-release.yml`](./docs/docker-compose.cd-release.yml.md)
  - [`cd.sh`](./docs/cd.sh.md) (Executor da pipeline)
  - [`cd-pre-<stage>.sh`](./docs/cd-pre-stage.sh.md) (Hooks de pré-execucação dos estágios - Opcional)
  - [`cd-post-<stage>.sh`](./docs/cd-post-stage.sh.md) (Hooks de pós-execucação dos estágios - Opcional)

- Ambientes
  - [`docker-compose.env-{environment}.yml`](./docs/docker-compose.env-environment.yml.md)