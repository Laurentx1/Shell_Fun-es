#!/bin/bash

# Limpa a tela
clear

# Definindo cores para o output
RED="\033[0;31m"
GREEN="\033[0;32m"
YELLOW="\033[1;33m"
BLUE="\033[0;34m"
PURPLE="\033[0;35m"
CYAN="\033[0;36m"
BOLD="\033[1m"
NC="\033[0m"  # Sem cor

# Banner LAURENT em ASCII art com cores melhoradas
LAURENT_ART=$(cat << "EOF"

 __         ______     __  __     ______     ______     __   __     ______
/\ \       /\  __ \   /\ \/\ \   /\  == \   /\  ___\   /\ "-.\ \   /\__  _\
\ \ \____  \ \  __ \  \ \ \_\ \  \ \  __<   \ \  __\   \ \ \-.  \  \/_/\ \/
 \ \_____\  \ \_\ \_\  \ \_____\  \ \_\ \_\  \ \_____\  \ \_\\"\_\    \ \_\
  \/_____/   \/_/\/_/   \/_____/   \/_/ /_/   \/_____/   \/_/ \/_/     \/_/

EOF
)

# Verifica se o script está sendo executado como root
check_root() {
    if [ "$EUID" -ne 0 ]; then
        echo -e "${YELLOW}⚠️  Aviso: Algumas funções podem requerer privilégios de root${NC}"
        return 1
    fi
    return 0
}

# Função para mostrar a barra de progresso
show_progress() {
    local duration=$1
    local step=$((duration/20))
    echo -ne "${CYAN}["
    for i in {1..20}; do
        echo -ne "#"
        sleep $step
    done
    echo -e "]${NC}"
}

# Função: Mostrar o cabeçalho
show_header() {
    clear
    echo -e "${CYAN}$LAURENT_ART${NC}"
    echo -e "${BOLD}${BLUE}┌───────────────────────────────────────────────────────┐${NC}"
    echo -e "${BOLD}${BLUE}│              Administração de Sistema                 │${NC}"
    echo -e "${BOLD}${BLUE}└───────────────────────────────────────────────────────┘${NC}"
    # Mostrar data e hora
    echo -e "${PURPLE}Data: $(date '+%d/%m/%Y %H:%M:%S')${NC}"
    echo ""
}

# Função: Mostrar informações do sistema
show_system_info() {
    show_header
    echo -e "${BOLD}${GREEN}📊 Informações do Sistema:${NC}"
    echo -e "${YELLOW}─────────────────────────────────────────────────────${NC}"
    echo -e "${CYAN}Sistema Operacional:${NC} $(cat /etc/os-release | grep "PRETTY_NAME" | cut -d '"' -f 2)"
    echo -e "${CYAN}Kernel:${NC} $(uname -r)"
    echo -e "${CYAN}Hostname:${NC} $(hostname)"
    echo -e "${CYAN}Uptime:${NC} $(uptime -p)"
    echo -e "${CYAN}CPU:${NC} $(lscpu | grep "Model name" | sed 's/Model name: *//g')"
    echo -e "${CYAN}Memória Total:${NC} $(free -h | awk '/^Mem:/ {print $2}')"
    echo -e "${CYAN}Memória Disponível:${NC} $(free -h | awk '/^Mem:/ {print $7}')"
    echo ""
}

# Função: Mostrar uso do disco
show_disk_usage() {
    show_header
    echo -e "${BOLD}${GREEN}💾 Uso do Disco:${NC}"
    echo -e "${YELLOW}─────────────────────────────────────────────────────${NC}"
    df -h | awk '{
        if (NR==1) {
            print $0
        } else {
            use=$5
            gsub(/%/,"",use)
            if (use+0 > 90) {
                print $0 " ⚠️ CRÍTICO"
            } else if (use+0 > 75) {
                print $0 " ⚠️ ATENÇÃO"
            } else {
                print $0
            }
        }
    }'
    echo ""
}

# Função: Mostrar processos ativos
show_top_processes() {
    show_header
    echo -e "${BOLD}${GREEN}⚙️ Top Processos Ativos:${NC}"
    echo -e "${YELLOW}─────────────────────────────────────────────────────${NC}"
    echo -e "${CYAN}Processos ordenados por uso de CPU:${NC}"
    ps aux --sort=-%cpu | head -11
    echo ""
    echo -e "${CYAN}Processos ordenados por uso de Memória:${NC}"
    ps aux --sort=-%mem | head -6
    echo ""
}

# Função: Atualizar o sistema
update_system() {
    show_header
    echo -e "${BOLD}${GREEN}🔄 Atualizando o sistema...${NC}"
    echo -e "${YELLOW}─────────────────────────────────────────────────────${NC}"
    
    # Detectar o gerenciador de pacotes
    if command -v dnf &> /dev/null; then
        # Fedora/RHEL
        echo -e "Usando ${CYAN}DNF${NC} para atualizar o sistema..."
        sudo dnf check-update
        echo -e "${YELLOW}Iniciando atualização...${NC}"
        sudo dnf upgrade -y
    elif command -v apt &> /dev/null; then
        # Debian/Ubuntu
        echo -e "Usando ${CYAN}APT${NC} para atualizar o sistema..."
        sudo apt update
        echo -e "${YELLOW}Iniciando atualização...${NC}"
        sudo apt upgrade -y
    elif command -v pacman &> /dev/null; then
        # Arch Linux
        echo -e "Usando ${CYAN}PACMAN${NC} para atualizar o sistema..."
        sudo pacman -Sy
        echo -e "${YELLOW}Iniciando atualização...${NC}"
        sudo pacman -Su --noconfirm
    else
        echo -e "${RED}Gerenciador de pacotes não reconhecido.${NC}"
        return 1
    fi
    
    echo -e "${GREEN}✅ Atualização concluída!${NC}"
}

# Função: Abrir o Kong Manager
open_kong_manager() {
    show_header
    echo -e "${BOLD}${GREEN}🌐 Kong Manager:${NC}"
    echo -e "${YELLOW}─────────────────────────────────────────────────────${NC}"
    
    # Verificar se o Kong está em execução
    if command -v curl &> /dev/null; then
        if curl -s http://localhost:8001 &> /dev/null; then
            echo -e "${GREEN}✅ Kong API Gateway está em execução!${NC}"
        else
            echo -e "${RED}❌ Kong API Gateway não está em execução.${NC}"
            read -p "Deseja tentar iniciar o Kong? (s/n): " start_kong
            if [[ "$start_kong" =~ ^[Ss]$ ]]; then
                echo "Tentando iniciar o Kong..."
                if command -v docker &> /dev/null; then
                    docker ps | grep -q kong
                    if [ $? -eq 0 ]; then
                        echo "Reiniciando contêiner Kong..."
                        docker restart $(docker ps -q --filter "name=kong")
                    else
                        echo "Contêiner Kong não encontrado."
                    fi
                else
                    echo "Docker não encontrado. Tentando iniciar o serviço..."
                    sudo systemctl start kong || echo "Serviço Kong não encontrado."
                fi
            fi
        fi
    fi
    
    echo -e "${CYAN}Abrindo o Kong Manager no navegador...${NC}"
    
    # Tentar diferentes comandos para abrir o navegador
    if command -v xdg-open &> /dev/null; then
        xdg-open "http://localhost:8002" &
    elif command -v open &> /dev/null; then
        open "http://localhost:8002" &
    elif command -v start &> /dev/null; then
        start "http://localhost:8002" &
    else
        echo "Não foi possível abrir automaticamente. Abra manualmente: http://localhost:8002"
    fi
    
    echo ""
}

# Função: Exibir informações de rede
show_network_info() {
    show_header
    echo -e "${BOLD}${GREEN}🔌 Informações de Rede:${NC}"
    echo -e "${YELLOW}─────────────────────────────────────────────────────${NC}"
    
    # Interfaces de rede
    echo -e "${BOLD}${CYAN}Interfaces de rede:${NC}"
    ip -c addr show | grep -E "^[0-9]+: |inet " | sed 's/^/  /'
    
    # Conexões e portas abertas
    echo -e "\n${BOLD}${CYAN}Portas abertas e conexões:${NC}"
    netstat -tuln 2>/dev/null || ss -tuln
    
    # Ping para verificar conectividade
    echo -e "\n${BOLD}${CYAN}Teste de conectividade:${NC}"
    ping -c 3 8.8.8.8 2>/dev/null || echo "  Ping não disponível"
    
    echo ""
}

# Função: Exibir logs recentes do sistema
show_system_logs() {
    show_header
    echo -e "${BOLD}${GREEN}📋 Logs do Sistema:${NC}"
    echo -e "${YELLOW}─────────────────────────────────────────────────────${NC}"
    
    # Verificar qual sistema de log está disponível
    if command -v journalctl &> /dev/null; then
        echo -e "${CYAN}Últimas 20 linhas dos logs do sistema (journalctl):${NC}"
        journalctl -n 20 --no-pager
    elif [ -f "/var/log/syslog" ]; then
        echo -e "${CYAN}Últimas 20 linhas dos logs do sistema (syslog):${NC}"
        sudo tail -n 20 /var/log/syslog
    elif [ -f "/var/log/messages" ]; then
        echo -e "${CYAN}Últimas 20 linhas dos logs do sistema (messages):${NC}"
        sudo tail -n 20 /var/log/messages
    else
        echo -e "${RED}Não foi possível encontrar logs do sistema.${NC}"
    fi
    
    echo ""
}

# Função: Verificar status do Docker
show_docker_status() {
    show_header
    echo -e "${BOLD}${GREEN}🐳 Status do Docker:${NC}"
    echo -e "${YELLOW}─────────────────────────────────────────────────────${NC}"
    
    if ! command -v docker &> /dev/null; then
        echo -e "${RED}Docker não está instalado no sistema.${NC}"
        return 1
    fi
    
    # Status do serviço
    echo -e "${CYAN}Status do serviço Docker:${NC}"
    systemctl status --no-pager docker | head -n 5
    
    # Informações do Docker
    echo -e "\n${CYAN}Versão do Docker:${NC}"
    docker --version
    
    # Contêineres em execução
    echo -e "\n${CYAN}Contêineres em execução:${NC}"
    docker ps --format "table {{.ID}}\t{{.Names}}\t{{.Status}}\t{{.Ports}}" 2>/dev/null || echo "  Nenhum contêiner em execução."
    
    # Uso de recursos dos contêineres
    echo -e "\n${CYAN}Uso de recursos dos contêineres:${NC}"
    docker stats --no-stream --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}" 2>/dev/null || echo "  Não foi possível obter estatísticas."
    
    echo ""
}

# Função: Reiniciar o serviço de rede
restart_network() {
    show_header
    echo -e "${BOLD}${GREEN}🔄 Reiniciando o serviço de rede...${NC}"
    echo -e "${YELLOW}─────────────────────────────────────────────────────${NC}"
    
    # Detectar o sistema de init e reiniciar a rede corretamente
    if command -v systemctl &> /dev/null; then
        # systemd
        if systemctl list-unit-files | grep -q NetworkManager; then
            echo "Reiniciando NetworkManager..."
            sudo systemctl restart NetworkManager
        elif systemctl list-unit-files | grep -q networking; then
            echo "Reiniciando serviço networking..."
            sudo systemctl restart networking
        else
            echo -e "${RED}Serviço de rede não encontrado.${NC}"
            return 1
        fi
    elif command -v service &> /dev/null; then
        # System V
        echo "Reiniciando serviço de rede (System V)..."
        sudo service network restart || sudo service networking restart
    else
        echo -e "${RED}Não foi possível determinar o sistema de init.${NC}"
        return 1
    fi
    
    echo -e "${GREEN}✅ Serviço de rede reiniciado.${NC}"
    echo ""
    
    # Verificar status da rede após reiniciar
    echo -e "${CYAN}Verificando status da conexão:${NC}"
    ip a | grep -E "^[0-9]+: |inet " | sed 's/^/  /'
    echo ""
}

# Função: Verificar serviços
check_services() {
    show_header
    echo -e "${BOLD}${GREEN}🔍 Verificando Serviços Essenciais:${NC}"
    echo -e "${YELLOW}─────────────────────────────────────────────────────${NC}"
    
    # Lista de serviços para verificar
    services=("sshd" "docker" "nginx" "httpd" "apache2" "postgresql" "mysql" "mariadb" "kong")
    
    for service in "${services[@]}"; do
        if systemctl list-unit-files | grep -q "$service"; then
            status=$(systemctl is-active "$service")
            if [ "$status" = "active" ]; then
                echo -e "${service}: ${GREEN}✅ Ativo${NC}"
            else
                echo -e "${service}: ${RED}❌ Inativo${NC}"
            fi
        fi
    done
    
    echo ""
}

# Função: Limpeza do sistema
clean_system() {
    show_header
    echo -e "${BOLD}${GREEN}🧹 Limpeza do Sistema:${NC}"
    echo -e "${YELLOW}─────────────────────────────────────────────────────${NC}"
    
    echo -e "${CYAN}Iniciando limpeza...${NC}"
    
    # Detectar o gerenciador de pacotes
    if command -v dnf &> /dev/null; then
        # Fedora/RHEL
        echo "Limpando cache do DNF..."
        sudo dnf clean all
        echo "Removendo pacotes antigos..."
        sudo dnf autoremove -y
    elif command -v apt &> /dev/null; then
        # Debian/Ubuntu
        echo "Limpando cache do APT..."
        sudo apt clean
        echo "Removendo pacotes antigos..."
        sudo apt autoremove -y
    elif command -v pacman &> /dev/null; then
        # Arch Linux
        echo "Limpando cache do Pacman..."
        sudo pacman -Sc --noconfirm
        echo "Removendo pacotes órfãos..."
        if command -v paru &> /dev/null; then
            paru -c
        elif command -v yay &> /dev/null; then
            yay -Yc
        else
            sudo pacman -Rns $(pacman -Qtdq) --noconfirm 2>/dev/null || echo "Nenhum pacote órfão encontrado."
        fi
    else
        echo -e "${RED}Gerenciador de pacotes não reconhecido.${NC}"
    fi
    
    # Limpeza geral
    echo "Limpando arquivos temporários..."
    sudo rm -rf /tmp/* 2>/dev/null || echo "Não foi possível limpar /tmp completamente."
    
    # Limpeza de logs
    echo "Limpando logs antigos..."
    if command -v journalctl &> /dev/null; then
        sudo journalctl --vacuum-time=7d
    fi
    
    # Limpeza do Docker se instalado
    if command -v docker &> /dev/null; then
        echo "Limpando recursos não utilizados do Docker..."
        docker system prune -f
    fi
    
    echo -e "${GREEN}✅ Limpeza concluída!${NC}"
    echo ""
}

# Loop principal do menu
while true; do
    show_header
    echo -e "${YELLOW}${BOLD}MENU PRINCIPAL${NC}"
    echo -e "${YELLOW}─────────────────────────────────────────────────────${NC}"
    echo -e "  ${CYAN}1)${NC} 📊 Informações do Sistema"
    echo -e "  ${CYAN}2)${NC} 💾 Uso do Disco"
    echo -e "  ${CYAN}3)${NC} ⚙️ Processos Ativos"
    echo -e "  ${CYAN}4)${NC} 🔄 Atualizar Sistema"
    echo -e "  ${CYAN}5)${NC} 🌐 Abrir Kong Manager"
    echo -e "  ${CYAN}6)${NC} 🔌 Informações de Rede"
    echo -e "  ${CYAN}7)${NC} 📋 Logs do Sistema"
    echo -e "  ${CYAN}8)${NC} 🐳 Status do Docker"
    echo -e "  ${CYAN}9)${NC} 🔄 Reiniciar Serviço de Rede"
    echo -e "  ${CYAN}10)${NC} 🔍 Verificar Serviços"
    echo -e "  ${CYAN}11)${NC} 🧹 Limpeza do Sistema"
    echo -e "  ${CYAN}0)${NC} 🚪 Sair"
    echo -e "${YELLOW}─────────────────────────────────────────────────────${NC}"
    read -p "Digite sua escolha: " choice
    echo ""

    case $choice in
        1)
            show_system_info
            ;;
        2)
            show_disk_usage
            ;;
        3)
            show_top_processes
            ;;
        4)
            update_system
            ;;
        5)
            open_kong_manager
            ;;
        6)
            show_network_info
            ;;
        7)
            show_system_logs
            ;;
        8)
            show_docker_status
            ;;
        9)
            restart_network
            ;;
        10)
            check_services
            ;;
        11)
            clean_system
            ;;
        0)
            clear
            echo -e "${GREEN}$LAURENT_ART${NC}"
            echo -e "${BLUE}Obrigado por usar o script LAURENT!${NC}"
            echo -e "${BLUE}Saindo...${NC}"
            exit 0
            ;;
        *)
            show_header
            echo -e "${RED}⚠️ Opção inválida! Tente novamente.${NC}"
            ;;
    esac

    read -p "Pressione [Enter] para continuar..."
done
