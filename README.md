# Descritivo da Arquitetura de Segurança do Ambiente Híbrido

Este documento apresenta a arquitetura de segurança proposta para um ambiente híbrido que integra infraestrutura On Premise e Cloud Pública, construída com base nos requisitos da avaliação da Cantu. O diagrama foi elaborado para incluir o máximo de informações relevantes sem comprometer a clareza visual. Alguns detalhes técnicos mais específicos, como configurações avançadas de firewall, políticas internas de hardening e fluxos completos de autenticação, foram intencionalmente omitidos para evitar poluição visual e manter o foco nos principais componentes da arquitetura.

---

## 1. Ambiente On Premise (Escritório e Armazém)

A infraestrutura local abriga aproximadamente 250 colaboradores e é segmentada em múltiplas VLANs para reduzir superfícies de ataque e aumentar a governança da rede.

### VLAN Armazém e IoT
Rede dedicada para dispositivos do armazém, incluindo leitores industriais e câmeras de CFTV. Essa segmentação impede tráfego desnecessário e mantém dispositivos IoT isolados do ambiente corporativo.

### VLAN Corporativa
Segmento destinado a usuários internos, impressoras e ramais IP. Aqui é aplicado controle de acesso e políticas de tráfego, garantindo maior segurança nas comunicações entre serviços internos.

### Cluster VMware ESXi
O ambiente virtualizado abriga três servidores críticos: File Server, Active Directory e Print Server. Cada máquina virtual opera com um agente EDR, oferecendo visibilidade e proteção contra ameaças.  
O Active Directory se integra ao Azure AD via RBAC, permitindo administração centralizada de identidades.

### NGFW e Acesso Seguro
Um NGFW protege o perímetro local com inspeção profunda de pacotes, IPS e controle de conteúdo.  
Os colaboradores remotos acessam os sistemas internos com segurança por meio do Zscaler ZTNA, combinado com TLS e EDR. O usuário remoto se autentica no Azure AD e, após validação, o tráfego permitido é encaminhado pela Zscaler para os recursos da empresa com inspeção e criptografia. Esse modelo substitui a necessidade de VPN tradicional, reduz a superfície de ataque e garante que apenas dispositivos confiáveis e usuários autorizados acessem o ambiente On Premise e o ambiente de ecommerce na cloud.

### Conexão Site-to-Site
O escritório e a cloud são interligados por VPN IPSec com TLS 1.3, garantindo comunicação criptografada e resiliência do tráfego entre ambientes.

---

## 2. Ambiente de Ecommerce em Cloud Pública

A aplicação de ecommerce, responsável por grande parte da receita da empresa, opera de forma segregada dentro de uma VPC exclusiva, com diversos mecanismos de proteção.

### Proteção Perimetral na Cloud
O ambiente é protegido por um Cloud WAF aliado a um mecanismo anti-DDoS. Essa camada garante que somente tráfego legítimo chegue ao cluster, mitigando ataques volumétricos e requisições maliciosas.

### Balanceamento de Carga
Um load balancer distribui requisições entre múltiplos pods, garantindo escalabilidade, estabilidade da aplicação e alta disponibilidade.

### Kubernetes Cluster
O cluster executa:
- Frontend em Node.js  
- Backend em .NET Core  
- Serviço de sanitização de inputs  

Toda comunicação interna utiliza mTLS, assegurando integridade e autenticação entre microserviços.

### Banco de Dados MongoDB
O MongoDB opera em ReplicaSet com instância primária e secundária.  
Características principais:
- Criptografia TLS 1.3 em trânsito  
- Criptografia AES-256 em repouso  
- Failover automático em caso de indisponibilidade da instância primária  

### Armazenamento e Snapshots
Backups, logs e snapshots são enviados ao Azure Blob.  
O gerenciamento de chaves é feito por KMS, reforçando a segurança do armazenamento.

---

## 3. Infraestrutura de E-mails

O e-mail é a principal ferramenta de comunicação e recebe proteção por múltiplas camadas:

- Gateway com antimalware e sandbox para inspeção de anexos  
- DLP para prevenir vazamento de informações  
- Microsoft Exchange integrado ao ecossistema de segurança  

Essa combinação protege mensagens internas e externas, assegurando sigilo e integridade.

---

## 4. Monitoramento Centralizado

Toda a telemetria gerada nos ambientes On Premise e Cloud é consolidada em:

- Microsoft Sentinel (SIEM)  
- Azure Monitor  
- Microsoft Defender  

Essas ferramentas permitem visibilidade completa do ambiente, detecção de anomalias, correlação de eventos e resposta rápida a incidentes.

---

## 5. MFA e IAM

O controle de identidades é centralizado com RBAC, MFA e políticas de IAM que abrangem usuários, aplicações, serviços e recursos críticos. Isso reduz riscos de acessos indevidos e aumenta a segurança operacional.

---

# Tríade da Segurança da Informação

A arquitetura foi projetada para sustentar os pilares de Confidencialidade, Integridade e Disponibilidade em todas as camadas.

---

## Confidencialidade

- Criptografia TLS 1.3 em todas as comunicações sensíveis  
- Criptografia AES-256 para dados armazenados  
- Segmentação por VLAN  
- Acesso remoto via ZTNA  
- DLP para proteção de informações sensíveis  
- WAF, NGFW e sandbox assegurando filtragem avançada  

Esses mecanismos garantem que somente pessoas e sistemas autorizados tenham acesso aos dados.

---

## Integridade

- mTLS entre serviços no Kubernetes  
- Sanitização de entradas na aplicação backend  
- RBAC centralizado no Azure AD e AD On Premise  
- Monitoramento contínuo via SIEM e Defender  
- ReplicaSet do MongoDB garantindo consistência  

A solução assegura que dados e comunicações não sejam alterados de maneira indevida e que toda atividade seja auditável.

---

## Disponibilidade

O pilar mais fortalecido da arquitetura ocorre por meio de redundância, resiliência e escalabilidade:

- Failover automático do MongoDB ReplicaSet  
- Load balancing para distribuição de carga  
- Mitigação de DDoS preservando a operação do ecommerce  
- Kubernetes com múltiplos pods replicados  
- Snapshots e storage no Azure Blob  
- Cluster VMware com alta disponibilidade  
- VPN redundante reforçando conexão entre ambientes  
- Contenção de falhas mediante VLANs dedicadas  
- Ação automática de EDR para resposta rápida a incidentes  

A soma desses componentes mantém o ambiente em estado resiliente e operacional mesmo diante de falhas internas ou ataques externos.

---

# Conclusão

A arquitetura proposta oferece uma solução híbrida robusta, alinhada às melhores práticas de segurança e com ênfase estratégica em disponibilidade, garantindo continuidade dos serviços críticos, principalmente o ecommerce. A integração entre ferramentas avançadas de monitoramento, controles de identidade, segmentação de rede, proteção perimetral e replicação de dados cria um ambiente resiliente, seguro e preparado para evolução futura.

