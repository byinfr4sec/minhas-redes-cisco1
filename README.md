## **Visão Geral**

Esta simulação representa uma **empresa com 4 departamentos** (Administração, TI, Financeiro, Público) + servidores, com:
- **Segmentação:** 5 VLANs (10-Admin, 20-TI, 30-Financeiro, 40-Público, 50-Servidores)
- **Redundância:** HSRP entre R1 (primário) e R2 (backup)
- **Segurança:** ACL no Firewall (libera só HTTP/HTTPS)
- **Serviços:** DHCP por VLAN, AppServer (HTTP), DBServer (MySQL)

**Author:** Rafael Pereira - Infr4Sec
**Stack:** Cisco Packet Tracer 8.2+  
**Tempo de Construção:** 1 hora  
**Dificuldade:** Intermediário (CCNA nível)

---

## **Topologia**

```
[Internet]
   ↑ (200.0.0.0/30)
  [FW] ← ACLs (libera 80/443)
   ↑ (10.0.0.0/30)
[Core1] ←→ [Core2] ← HSRP VIP 10.0.0.3
   |         |
[Access1] [Access2] [Access3]
  |         |         |
[PCs Admin/TI] [PCs Fin] [PCs Pub]
   |         |
[AppServer] [DBServer]
```

### **Equipamentos**
| Equipamento | Quantidade | Portas Principais |
|-------------|------------|-------------------|
| ISR 4331 (Roteador) | 2 | Gi0/0/0 (Internet), Gi0/0/1 (Trunk) |
| 2960-24TT (Switch) | 5 | Fa0/1 (Trunk), Fa0/2+ (Access) |
| PC-PT | 8 | Fa0 (DHCP) |
| Server-PT | 2 | Fa0 (Manual IP) |

---

## **Endereçamento IP**

| VLAN | Departamento | Rede | Máscara | Gateway | IPs Úteis |
|------|--------------|------|---------|---------|-----------|
| 10 | Administração | 10.0.1.0 | /25 (255.255.255.128) | 10.0.1.1 | 126 |
| 20 | TI | 10.0.2.0 | /27 (255.255.255.224) | 10.0.2.1 | 30 |
| 30 | Financeiro | 10.0.3.0 | /28 (255.255.255.240) | 10.0.3.1 | 14 |
| 40 | Público | 10.0.4.0 | /25 (255.255.255.128) | 10.0.4.1 | 126 |
| 50 | Servidores | 10.0.5.0 | /30 (255.255.255.252) | 10.0.5.1 | 2 |
| 999 | Link Core | 10.0.0.0 | /30 (255.255.255.252) | — | 2 |

- **AppServer:** 10.0.5.2 (HTTP ativo)
- **DBServer:** 10.0.5.3 (MySQL ativo)

---

## **Configurações Principais**

### **VLANs (Core1/Core2)**
```bash
vlan 10
 name ADMINISTRACAO
vlan 30
 name FINANCEIRO
! (repita para 20,40,50)
```

### **Trunk (Core1 → Access)**
```bash
interface FastEthernet0/4
 switchport mode trunk
 switchport trunk allowed vlan 30,999
```

### **Subinterfaces (R1)**
```bash
interface GigabitEthernet0/0/1.30
 encapsulation dot1Q 30
 ip address 10.0.3.1 255.255.255.240
 no shutdown
```

### **DHCP (R1)**
```bash
ip dhcp pool FINANCEIRO
 network 10.0.3.0 255.255.255.240
 default-router 10.0.3.1
```

### **HSRP (R1/R2)**
```bash
interface GigabitEthernet0/0/1
 standby 1 ip 10.0.0.3
 standby 1 priority 110  ! R1 (primário)
```

### **ACL (FW)**
```bash
access-list 101 permit tcp any any eq 80
access-list 101 deny ip any any
interface GigabitEthernet0/0/0
 ip access-group 101 in
```

---

## **Como Usar**

1. **Baixe o .pkt:** [rede-corporativa-v1.pkt](rede-corporativa-v1.pkt)
2. **Abra no Packet Tracer 8.2+:**
   - File → Open → Selecione o arquivo
   - Todos os cabos verdes, configs salvas
3. **Teste:**
   - Ping entre VLANs: `ping 10.0.3.2` (de PC-Admin para PC-Fin)
   - Web: Navegador → `http://10.0.5.2`
   - Redundância: Desligue R1 → rede continua
4. **Modifique:** Adicione Wi-Fi ou OSPF!

**Requisitos:** Cisco Packet Tracer (grátis para estudantes em [netacad.com](https://www.netacad.com))

---

## **Testes Realizados**

| Teste | Resultado |
|-------|-----------|
| DHCP em todas VLANs | ✅ IPs atribuídos |
| Ping inter-VLAN | ✅ `!!!!!` |
| Acesso AppServer | ✅ Página HTTP |
| Bloqueio ACL (PUB → DB) | ✅ Falha |
| HSRP (desligar R1) | ✅ R2 assume |


## **Licença**

**MIT License** — Use, modifique e compartilhe livremente.

---

**Feito com ❤️ por um engenheiro de redes que odeia downtime.**  
**Contato: analista.rafaelps@gmail.com | LinkedIn: [linkedin.com/in/seu-perfil](https://www.linkedin.com/in/byrafanet/)**
