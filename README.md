# Ambiente GitLab com Nginx, Runner e Ngrok via Docker Compose
Este projeto utiliza Docker Compose para orquestrar um ambiente de desenvolvimento completo com GitLab, um GitLab Runner para CI/CD, Nginx como proxy reverso e Ngrok para expor a instância na internet sem a necessidade de configurar portas.

## Visão Geral da Arquitetura

- GitLab CE: A instância principal do GitLab Community Edition.

- GitLab Runner: O executor de trabalhos de CI/CD, configurado para usar o Docker do host.

- Nginx: Atua como um proxy reverso, direcionando o tráfego da internet para o contêiner do GitLab.

- Ngrok: Cria um túnel seguro para o serviço Nginx, usando um domínio estático para acesso público.

- Docker Compose: Orquestra a inicialização, comunicação e o ciclo de vida de todos os contêineres.

## Pré-requisitos

- Docker e Docker Compose: Certifique-se de que ambos estejam instalados. A versão V2 do Compose (docker compose) é recomendada.

- Conta Ngrok e Domínio Estático: É necessária uma conta no ngrok.com com um domínio estático configurado. Você precisará do seu authtoken e do nome do seu domínio estático.

## Estrutura de Arquivos

Certifique-se de que seus arquivos estão organizados da seguinte forma:

    .gitlab/
    ├── docker-compose.yml
    ├── .env
    └── nginx/
        └── gitlab.conf

## Passo a Passo para Executar

#### Passo 1: Configurar o Arquivo .env

O arquivo .env é usado para armazenar suas credenciais e o domínio estático da sua instância.

- Obtenha suas credenciais do Ngrok:

    - Faça login no seu dashboard do Ngrok.

    - Em "Cloud Edge" -> "Domains", certifique-se de que você tem um domínio estático criado (ex: meu-gitlab.ngrok-free.app).

    - Em "Your Authtoken", copie seu token.
    - Edite o arquivo .env:

    - Abra o arquivo .env e preencha as variáveis com seu token e seu domínio estático.

####  Exemplo de .env:

- Cole seu Authtoken do Ngrok aqui
    NGROK_AUTHTOKEN=SEU_TOKEN_AQUI

- Coloque seu domínio estático do Ngrok aqui
    GITLAB_DOMAIN=seu-dominio-estatico.ngrok-free.app

### Passo 2: Atualizar o Comando do Ngrok (se necessário)

Para que o Ngrok utilize seu domínio estático, o serviço ngrok no arquivo docker-compose.yml deve ser configurado para usá-lo. Verifique se a linha command está correta:

#### Em docker-compose.yml, no serviço ngrok:
```
  ngrok:
    image: ngrok/ngrok:latest
    # ... outras configurações ...
    command: 'http nginx:80 --domain=seu-dominio-estatico.ngrok-free.app --log=stdout'
```

Substitua seu-dominio-estatico.ngrok-free.app pelo seu domínio real.
#### Passo 3: Iniciar a Stack Completa

Como o domínio é fixo, você pode iniciar todos os serviços de uma só vez.

```
docker compose up -d
```
O GitLab pode levar de 5 a 15 minutos para iniciar completamente na primeira vez. Você pode acompanhar o progresso com:

```
docker compose logs -f gitlab
```

Aguarde até ver a mensagem gitlab Reconfigured!.

#### Passo 4: Acessar o GitLab e Configurar a Senha

- Acesse a URL: Abra seu navegador e acesse o seu domínio estático do Ngrok (ex: http://seu-dominio-estatico.ngrok-free.app).

- Login: A página de login será exibida. O nome de usuário é root.

- Senha Inicial: Para obter a senha inicial gerada automaticamente, execute o comando:

- docker compose exec gitlab cat /etc/gitlab/initial_root_password

- Copie a senha exibida e use-a para fazer o primeiro login. É altamente recomendado trocar a senha imediatamente após o primeiro acesso.

#### Passo 5: Registrar o GitLab Runner

Para que o Runner possa executar os trabalhos de CI/CD, ele precisa ser registrado na sua instância GitLab.

- Obtenha o Token de Registro:
    - Faça login no GitLab como root.
    - Vá para o Admin Area (ícone de chave inglesa no menu superior).
    - No menu lateral, vá para CI/CD -> Runners.
    - Na seção "Register a new runner", você verá a URL da instância e o token de registro. Copie o token (começa com glrt-).

    - Execute o Comando de Registro:
    Volte ao seu terminal e execute o comando abaixo, substituindo a URL e o token pelos valores da sua instância.

    #### Exemplo de comando completo (você também pode rodar de forma interativa)
    ```
    docker compose exec gitlab-runner gitlab-runner register \
      --non-interactive \
      --url "[http://seu-dominio-estatico.ngrok-free.app](http://seu-dominio-estatico.ngrok-free.app)" \
      --token "SEU_TOKEN_DE_REGISTRO_AQUI" \
      --executor "docker" \
      --docker-image alpine:latest \
      --description "docker-runner" \
      --run-untagged="true" \
      --locked="false" \
      --access-level="not_protected"
    ```
Se preferir o modo interativo, apenas execute docker compose exec gitlab-runner gitlab-runner register e siga as instruções, certificando-se de escolher docker como executor.

Após o registro, o Runner aparecerá na página de Runners do GitLab com um status verde, pronto para uso.
Comandos Úteis

- Iniciar todos os serviços: 
```
docker compose up -d
```
- Parar todos os serviços: 
```
docker compose down
```
- Parar e remover os volumes de dados (recomeçar do zero): 
```
docker compose down -v
```
- Ver logs de um serviço específico (ex: gitlab): 
```
docker compose logs -f gitlab
```
- Acessar o terminal de um contêiner (ex: gitlab): 
```
docker compose exec gitlab bash
```