# ansible-playbooks

## Visão Geral

Para simplificar, faça uso do recurso devcontainer do VSCode, já configurado neste projeto: https://docs.ansible.com/projects/dev-tools/container/

Este projeto Ansible configura utilitários de storage, incluindo **NFS** e **iSCSI initiator**, em servidores baseados em Debian/Ubuntu.

Ele realiza as seguintes ações:
- Instala os pacotes `nfs-common` e `open-iscsi`.
- Garante que o serviço iSCSI (`iscsid`) esteja habilitado e em execução.
- Utiliza as roles `geerlingguy.nfs` e `ricsanfre.iscsi_initiator` para configurações adicionais.

## Preparação do Ambiente Python e Ansible (Linux)

Obs.: Não é necessário prepaparar o ambiente Python caso esteja utilizando o recurso DevContainer do VSCode, pois ele já isola o ambiente e configura todas as dependências automaticamente, tornando o uso do venv redundante.

Caso não esteja utilizando DevContainer, é altamente recomendável usar um ambiente virtual Python (`venv`) para isolar as dependências do Ansible.

1.  **Instalar `python3-venv` (se necessário)**:
    Se você estiver em um sistema Debian/Ubuntu, pode precisar instalar o pacote `python3-venv`:
```bash
sudo apt update
sudo apt install python3-venv
```

2.  **Criar e Ativar o Ambiente Virtual**:
    A partir da raiz do projeto, crie um ambiente virtual:
```bash
python3 -m venv .venv
source .venv/bin/activate
```
    (Se estiver usando Fish shell: `source .venv/bin/activate.fish`)

3.  **Instalar Ansible**:
    Com o ambiente virtual ativado, instale o Ansible:
```bash
pip install ansible
```

4.  **Desativar o Ambiente Virtual**:
    Quando terminar de trabalhar, você pode desativar o ambiente virtual:
```bash
deactivate
```

    Lembre-se de ativar o ambiente virtual (`source .venv/bin/activate`) sempre que for executar os playbooks.

## Requisitos

1. Certificado para acesso SSH

    Certifique-se de ter seu certificado de acesso ssh no caminho abaixo:
```bash
~/.ssh/ansible
```

2. Instalação de roles

    As roles de dependência são definidas no arquivo `requirements.yml`. Para instalá-las, utilize o seguinte comando:
```bash
ansible-galaxy install -r requirements.yml
```

## Configuração do usuário ansible nas máquinas do inventário:
```bash
# 1. Cria o usuário 'ansible' com diretório home e shell bash
sudo useradd -m -s /bin/bash ansible

# 2. (Opcional) Define uma senha para o usuário
# Para automação, o ideal é usar chaves SSH, mas uma senha inicial pode ser útil
sudo passwd ansible

# 3. Adiciona o usuário ao grupo sudo
sudo usermod -aG sudo ansible

# 4. Configura o Sudo sem senha (Crucial para Ansible)
# Isso permite que o Ansible execute comandos root sem travar pedindo senha
echo "ansible ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/ansible
```

## Como Executar

1.  **Preencha o Inventário**:
    *   Abra o arquivo `inventory.ini`.
    *   Na seção `[storage]`, substitua o IP de exemplo (`192.168.56.10`) pelos IPs ou hostnames das suas VMs de destino.
    Teste:
```bash
ansible k8s-worker-01 -i inventory.ini -m ping
```
2.  **Execute o Playbook**:
    *   Use o seguinte comando no seu terminal, a partir da raiz do projeto:

```bash
ansible-playbook -i inventory.ini playbooks/clear_apt_cache.yaml
ansible-playbook -i inventory.ini playbooks/setup_storage.yaml
```