### [ISSUE-01] Subir GLPI + OCS Inventory (Inventário Central)
**Objetivo:** Ter inventário unificado (Hostname ↔ MAC ↔ Usuário ↔ VLAN/Setor ↔ IP).  
**Pré-requisitos:** VM Linux provisionada.  
**Passos:**
1. Instalar Docker e Docker Compose.  
2. Subir containers GLPI + OCS Inventory.  
3. Criar campos customizados (Setor/VLAN, Privilégio, Local).  
4. Distribuir agentes OCS nos endpoints.  
**Critérios de Aceite:** ≥80% dos endpoints reportando em até 14 dias.  
**Riscos:** Duplicidade de ativos → habilitar merge por MAC.  
**Dependências:** Nenhuma.  

---

### [ISSUE-02] Criar Plano de Endereçamento por VLAN
**Objetivo:** Formalizar tabela de sub-redes, gateways e reservas.  
**Pré-requisitos:** Mapeamento atual de VLANs.  
**Passos:**
1. Mapear VLANs existentes e confirmar sub-redes desejadas.  
2. Criar planilha/NetBox com VLAN ID, Sub-rede, Gateway, Pool DHCP, Reservas.  
3. Validar com TI/gestores.  
**Critérios de Aceite:** Tabela aprovada e publicada em NetBox ou repositório Git.  
**Riscos:** Conflitos de IP → executar descoberta antes.  
**Dependências:** ISSUE-01.  

---

### [ISSUE-03] Coletar PCs fora da VLAN/sem IP fixo
**Objetivo:** Listar máquinas fora da VLAN correta e sem reserva.  
**Pré-requisitos:** DHCP ativo no RouterOS.  
**Passos:**
1. Exportar leases no RouterOS.  
2. Cruzar com inventário GLPI/OCS.  
3. Classificar em correto/incorreto/sem reserva.  
**Critérios de Aceite:** Planilha com mapeamento completo por setor.  
**Riscos:** Hosts com múltiplas NICs confundirem análise.  
**Dependências:** ISSUE-02.  

---

### [ISSUE-04] Ajustar Bridge e VLAN Filtering no RouterOS
**Objetivo:** Ativar bridge VLAN filtering com segurança.  
**Pré-requisitos:** Console físico/seguro + safe-mode.  
**Passos:**
1. Habilitar safe-mode.  
2. Configurar bridge, portas de trunk e acesso.  
3. Criar tabela VLAN na bridge.  
4. Ativar vlan-filtering.  
**Critérios de Aceite:** Conectividade mantida, sem vazamento de tráfego entre VLANs.  
**Riscos:** Lockout remoto → usar porta console.  
**Dependências:** ISSUE-02.  

---

### [ISSUE-05] Criar Interfaces L3 por VLAN no RouterOS
**Objetivo:** Ter gateways dedicados por VLAN.  
**Passos:**
1. Criar interfaces VLAN no RouterOS.  
2. Adicionar IPs/gateways correspondentes.  
**Critérios de Aceite:** Gateways respondendo ping em cada VLAN.  
**Riscos:** Conflito de IP com gateway antigo.  
**Dependências:** ISSUE-04.  

---

### [ISSUE-06] Configurar DHCP por VLAN + Pools
**Objetivo:** Entregar IPs corretos e reservas por MAC.  
**Passos:**
1. Criar pools por VLAN.  
2. Criar servidores DHCP vinculados a cada VLAN.  
3. Definir opções de rede (DNS, NTP).  
**Critérios de Aceite:** Hosts recebem IP conforme setor.  
**Riscos:** Outro servidor DHCP ativo → desabilitar.  
**Dependências:** ISSUE-05.  

---

### [ISSUE-07] Criar Reservas DHCP por MAC
**Objetivo:** Padronizar IP fixo via reserva.  
**Passos:**
1. Gerar lista de MACs e IPs por setor.  
2. Importar reservas via arquivo `.rsc`.  
3. Validar em Leases do RouterOS.  
**Critérios de Aceite:** ≥90% dos PCs com reserva atribuída.  
**Riscos:** IP duplicado → escanear previamente.  
**Dependências:** ISSUE-06.  

---

### [ISSUE-08] Configurar Firewall inter-VLAN (mínimo privilégio)
**Objetivo:** Bloquear tráfego desnecessário entre setores.  
**Passos:**
1. Criar listas de servidores essenciais.  
2. Permitir apenas fluxos aprovados.  
3. Bloquear o resto entre VLANs.  
**Critérios de Aceite:** Apenas comunicações necessárias funcionando.  
**Riscos:** Quebra de aplicações legadas.  
**Dependências:** ISSUE-05, ISSUE-07.  

---

### [ISSUE-09] Implantar Quarentena Automática para Dispositivos Não Mapeados
**Objetivo:** Isolar dispositivos não registrados no inventário.  
**Passos:**
1. Criar address-list QUARANTINE.  
2. Adicionar lease-script em servidores DHCP.  
3. Validar logs e timeouts.  
**Critérios de Aceite:** Dispositivos não registrados bloqueados em até 1h.  
**Riscos:** Falsos positivos iniciais.  
**Dependências:** ISSUE-01, ISSUE-03.  

---

### [ISSUE-10] Documentar Tudo no NetBox
**Objetivo:** Centralizar VLANs, IPs e equipamentos.  
**Passos:**
1. Criar sites, racks, devices no NetBox.  
2. Registrar VLANs e sub-redes.  
3. Associar interfaces a portas físicas.  
**Critérios de Aceite:** 100% VLANs e ≥80% das portas cadastradas.  
**Dependências:** ISSUE-02, ISSUE-07.  

---

### [ISSUE-11] Configurar Backup Automatizado do RouterOS
**Objetivo:** Garantir cópias versionadas das configs.  
**Passos:**
1. Criar scheduler de backup/export diário.  
2. Armazenar em servidor externo seguro.  
**Critérios de Aceite:** Backup diário salvo fora do roteador.  
**Riscos:** Exposição em transporte inseguro.  
**Dependências:** ISSUE-04–07.  

---

### [ISSUE-12] Migrar Setores por Ondas
**Objetivo:** Corrigir PCs por setor gradualmente.  
**Passos:**
1. Escolher setor piloto (ex: Financeiro).  
2. Ajustar switchports, forçar renew DHCP.  
3. Validar inventário e acesso aos servidores.  
4. Replicar para demais setores.  
**Critérios de Aceite:** 100% setores com VLAN/IP corretos.  
**Riscos:** Impressoras/APs em portas erradas → revisar antes.  
**Dependências:** ISSUE-07, ISSUE-08.  
