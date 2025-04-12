# LAURENT - Sistema de Administração Linux

![LAURENT Banner](https://img.shields.io/badge/LAURENT-Sistema%20de%20Administração-blue?style=for-the-badge)

LAURENT é uma ferramenta de administração de sistema para Linux com interface de linha de comando, projetada para facilitar o gerenciamento e monitoramento de servidores e estações de trabalho. O script fornece um menu interativo com várias funcionalidades para diagnóstico, manutenção e otimização de sistemas Linux.

## 🌟 Funcionalidades

O script LAURENT oferece uma variedade de funções administrativas, incluindo:

- **📊 Informações do Sistema**: Exibe detalhes sobre o sistema operacional, kernel, CPU, memória e tempo de atividade
- **💾 Monitoramento de Disco**: Análise de uso de espaço em disco com alertas para partições críticas
- **⚙️ Gerenciamento de Processos**: Visualização dos processos que consomem mais CPU e memória
- **🔄 Atualização do Sistema**: Detecta automaticamente o gerenciador de pacotes e atualiza o sistema
- **🌐 Suporte para Kong API Gateway**: Acesso rápido ao Kong Manager e verificação de status
- **🔌 Diagnóstico de Rede**: Exibe interfaces, conexões e testa conectividade
- **📋 Visualização de Logs**: Acesso rápido aos logs de sistema mais recentes
- **🐳 Gerenciamento do Docker**: Monitora o status do Docker, contêineres em execução e uso de recursos
- **🔍 Verificação de Serviços**: Verifica o status de serviços essenciais no sistema
- **🧹 Limpeza do Sistema**: Remove arquivos temporários, pacotes obsoletos e libera espaço

## 📋 Requisitos

- Sistema operacional Linux (compatível com distribuições baseadas em Debian, Red Hat, Arch Linux)
- Permissões de administrador (sudo) para algumas funcionalidades
- Bash versão 4.0 ou superior

## 🚀 Instalação

1. Clone o repositório:
```bash
git clone https://github.com/seu-usuario/laurent.git
```

2. Navegue até o diretório do script:
```bash
cd laurent
```

3. Torne o script executável:
```bash
chmod +x laurent.sh
```

## 💻 Como Usar

Execute o script com:

```bash
./laurent.sh
```

Ou para funcionalidades completas, execute como sudo:

```bash
sudo ./laurent.sh
```

Após iniciar o script, você verá um menu interativo. Digite o número correspondente à função desejada e pressione Enter.

## 🛠️ Personalização

Você pode personalizar o script editando as seguintes seções:

- **Cores**: Modifique as variáveis de cor no início do script para alterar o esquema de cores
- **Serviços Verificados**: Edite o array `services` na função `check_services` para incluir os serviços relevantes para seu ambiente
- **Banner LAURENT**: Personalize a arte ASCII no início do script

## 🔍 Compatibilidade

O script foi projetado para funcionar em várias distribuições Linux, adaptando-se automaticamente aos diferentes gerenciadores de pacotes e sistemas:

- Debian/Ubuntu (apt)
- Fedora/RHEL/CentOS (dnf/yum)
- Arch Linux (pacman)

## 🤝 Contribuindo

Contribuições são bem-vindas! Se você tiver sugestões para melhorar o LAURENT:

1. Faça um fork do repositório
2. Crie um branch para sua feature (`git checkout -b feature/nova-funcionalidade`)
3. Commit suas mudanças (`git commit -m 'Adiciona nova funcionalidade'`)
4. Push para o branch (`git push origin feature/nova-funcionalidade`)
5. Abra um Pull Request

## 📝 Log de Alterações

### Versão 2.0.0
- Interface completamente redesenhada com ícones e melhor organização visual
- Detecção automática de gerenciadores de pacotes
- Adicionadas novas funcionalidades: verificação de serviços e limpeza de sistema
- Melhorias na exibição de informações de processo e rede
- Suporte aprimorado para Docker e Kong API Gateway

### Versão 1.0.0
- Versão inicial com funcionalidades básicas

## 📜 Licença

Este projeto está licenciado sob a licença MIT. Veja o arquivo [LICENSE](LICENSE) para mais detalhes.

## ✉️ Contato

Para dúvidas, sugestões ou problemas, abra uma issue neste repositório ou entre em contato:

- Email: seu-email@example.com
- Twitter: [@seuusuario](https://twitter.com/seuusuario)

---

⭐ **LAURENT** - Tornando a administração de sistemas Linux mais simples e eficiente!
