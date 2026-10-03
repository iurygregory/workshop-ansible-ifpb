# Workshop - Ansible na pratica: O simples que resolve

Este projeto contem playbooks Ansible que instalam e configuram o **nginx** em uma maquina virtual CentOS Stream 10, acompanhado de um site de documentacao construido com Antora (tema Red Hat Showroom).

| VM | Funcao | Descricao |
|----|--------|-----------|
| VM 1 | **Control node** | Onde voce executa os comandos Ansible |
| VM 2 | **Managed node** | O servidor que vai rodar o nginx |

## Pre-requisitos

Na **VM 1** (control node):

```bash
sudo dnf install -y ansible-core
```

Verifique que a VM 1 consegue acessar a VM 2 via rede (ping, SSH).

## Quick Start

### 1. Clone o repositorio

```bash
git clone <REPO_URL>
cd workshop-ansible-ifpb
```

### 2. Configure o inventario

Edite `inventory/hosts.ini` e descomente **uma** das opcoes de conexao, substituindo os valores com o IP e credenciais da sua VM 2.

**Opcao A - Autenticacao por chave SSH (recomendado):**

```ini
[webservers]
vm2 ansible_host=192.168.56.11 ansible_user=seu_usuario ansible_ssh_private_key_file=~/.ssh/id_rsa
```

**Opcao B - Autenticacao por senha:**

```ini
[webservers]
vm2 ansible_host=192.168.56.11 ansible_user=seu_usuario ansible_ssh_pass=sua_senha
```

Se usar autenticacao por senha, instale o `sshpass` na VM 1: `sudo dnf install -y sshpass`

### 3. Execute o playbook

```bash
ansible-playbook playbooks/site.yml
```

### 4. Acesse a pagina

No navegador do computador host, abra:

```
http://<IP_DA_VM2>:80
```

Substitua `<IP_DA_VM2>` pelo IP da sua VM 2. Se alterou a porta, use a porta configurada.

## Personalizacao

Todas as configuracoes ficam em `inventory/group_vars/webservers.yml` (ou via `-e` na linha de comando).

### Alterar a porta do nginx

```bash
ansible-playbook playbooks/site.yml -e nginx_port=8080
```

### Instalar a versao mais recente do nginx

```bash
ansible-playbook playbooks/site.yml -e nginx_enable_official_repo=true
```

### Combinar ambos

```bash
ansible-playbook playbooks/site.yml -e nginx_enable_official_repo=true -e nginx_port=8080
```

## Comportamento da Pagina Web

| Condicao | O que a pagina mostra |
|----------|----------------------|
| Porta padrao (80) | Imagem (`default.png`) |
| Porta customizada + nginx NAO e a versao mais recente | Mensagem pedindo atualizacao |
| Porta customizada + nginx E a versao mais recente | Imagem (`latest.png`) |

### Substituindo as imagens

As imagens ficam em `roles/nginx/files/`. Substitua os arquivos placeholder:

- `roles/nginx/files/default.png` - exibida na pagina com porta padrao
- `roles/nginx/files/latest.png` - exibida na pagina com porta customizada + versao mais recente

Apos substituir, execute o playbook novamente para deployar.

## Site de Documentacao (Antora)

O conteudo do workshop esta disponivel como um site estatico construido com Antora e o tema Red Hat Showroom.

### Preview local com Podman

```bash
podman run --rm --name antora -v $PWD:/antora:Z -p 8080:8080 -i -t ghcr.io/juliaaano/antora-viewer
```

Acesse http://localhost:8080. O site e reconstruido automaticamente ao alterar arquivos `.adoc` ou `.yml`.

### Preview local com npm

```bash
npm install
npx antora site.yml
# Abra www/workshop-ansible-ifpb/index.html
```

### Deploy automatico

O repositorio inclui um workflow GitHub Actions (`.github/workflows/gh-pages.yml`) que publica o site automaticamente no GitHub Pages ao fazer push na branch `main`.

> Apos criar o repositorio no GitHub, ative o GitHub Pages em Settings > Pages > Source: GitHub Actions.

## Estrutura do Projeto

```
workshop-ansible-ifpb/
  .github/workflows/
    gh-pages.yml                     # CI/CD para GitHub Pages
  ansible.cfg                        # Configuracao do Ansible
  site.yml                           # Playbook Antora (gerador do site)
  package.json                       # Dependencias Antora
  content/
    antora.yml                       # Descritor do componente Antora
    supplemental-ui/css/             # CSS customizado (opcional)
    modules/ROOT/
      nav.adoc                       # Navegacao do site
      pages/
        index.adoc                   # Pagina inicial do workshop
        01-setup.adoc                # Modulo 1: Preparando o ambiente
        02-inventory.adoc            # Modulo 2: Inventario Ansible
        03-playbook.adoc             # Modulo 3: Executando o playbook
        04-customization.adoc        # Modulo 4: Personalizando o nginx
      images/                        # Imagens para o site
  inventory/
    hosts.ini                        # Inventario (edite este!)
    group_vars/
      webservers.yml                 # Variaveis do grupo webservers
  playbooks/
    site.yml                         # Playbook principal do Ansible
  roles/
    nginx/
      defaults/main.yml              # Valores padrao das variaveis
      tasks/main.yml                 # Tarefas (instalar, configurar, firewall)
      handlers/main.yml              # Handler de restart do nginx
      templates/
        nginx.conf.j2                # Template de configuracao do nginx
        index.html.j2                # Template da pagina web (Jinja2)
      files/
        default.png                  # Imagem para pagina com porta padrao
        latest.png                   # Imagem para pagina com versao mais recente
  diagrams/src/                      # Diagramas Mermaid
  labs/                              # Scripts auxiliares
```

## Troubleshooting

- **Nao consegue acessar a pagina**: Verifique se o firewalld esta rodando e a porta esta aberta. O playbook faz isso automaticamente, mas voce pode verificar com `sudo firewall-cmd --list-ports` na VM 2.
- **SELinux bloqueando a porta customizada**: O playbook usa `semanage` para permitir a porta. Verifique com `sudo semanage port -l | grep http_port_t`.
- **Autenticacao por senha falha**: Instale `sshpass` na VM 1 (`sudo dnf install -y sshpass`).

## Licenca

Este projeto e disponibilizado para fins educacionais como parte do workshop Ansible do IFPB.
