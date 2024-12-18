

# Cenário de Configuração: Sub-redes, Roteamento Estático e NAT

## Sub-redes Atribuídas

- **Rio de Janeiro (RJO):** `180.1.0.0/23`  
- **São Paulo (SPO):** `180.1.2.0/24`  
- **Curitiba (CTA):** `180.1.4.0/25`  
- **Belo Horizonte (BHE):** `180.1.6.0/26`  

## Interfaces no HWIC-2T

- **Serial disponíveis:**  
  - `Serial0/3/0`  -- BHE
  - `Serial0/3/1`  

- **Serial disponíveis:**  
  - `Serial0/2/0`  -- CTA
  - `Serial0/2/1`  

- **Serial disponíveis:**  
  - `Serial0/1/0`  -- SPO
  - `Serial0/1/1`  

## Interfaces GigaEthernet

- `GigaEthernet0/0` -- RJO
- `GigaEthernet0/1`

Cada local está conectado a uma dessas interfaces.

---

## Configuração das Interfaces em RJO

### 1. Configurar as Interfaces e Atribuir IPs

#### Interface para Rio de Janeiro
```plaintext
configure terminal
interface Gigaethernet0/0
ip address 180.1.0.1 255.255.254.0
no shutdown
description Conexão com Rio de Janeiro
```

#### Interface para São Paulo
```plaintext
interface Serial0/1/0
ip address 180.1.2.1 255.255.255.0
no shutdown
description Conexão com São Paulo
```

#### Interface para Curitiba
```plaintext
interface Serial0/2/0
ip address 180.1.4.1 255.255.255.128
no shutdown
description Conexão com Curitiba
```

#### Interface para Belo Horizonte
```plaintext
interface Serial0/3/0
ip address 180.1.6.1 255.255.255.192
no shutdown
description Conexão com Belo Horizonte
```

## Configuração de Rotas Estáticas

### 2. Adicionar Rotas Estáticas

#### Rota para Rio de Janeiro (180.1.0.0/23)
```plaintext
ip route 180.1.0.0 255.255.254.0 Gigaethernet0/0
```

#### Rota para São Paulo (180.1.2.0/24)
```plaintext
ip route 180.1.2.0 255.255.255.0 Serial0/1/0
```

#### Rota para Curitiba (180.1.4.0/25)
```plaintext
ip route 180.1.4.0 255.255.255.128 Serial0/2/0
```

#### Rota para Belo Horizonte (180.1.6.0/26)
```plaintext
ip route 180.1.6.0 255.255.255.192 Serial0/3/0
```

## Verificação e Salvar Configuração

### 3. Verificar Configurações de Interfaces e Rotas
```plaintext
show ip interface brief
show ip route
```

### 4. Salvar a Configuração
```plaintext
write memory
```

## Explicação dos Comandos
- **ip address:** Configura o endereço IP e a máscara de sub-rede da interface.
- **no shutdown:** Ativa a interface.
- **ip route:** Adiciona uma rota estática para uma sub-rede específica.
- **show ip route:** Exibe a tabela de roteamento para verificar se as rotas foram configuradas corretamente.
- **write memory:** Salva as configurações na memória NVRAM para que sejam persistentes após reinicializações.

---

## Configuração das Interfaces em BHE

### 1. Acesse o Modo de Configuração Global

### 2. Configure as Interfaces

#### Configurar a Interface GigabitEthernet0/0 (Rede Privada - Inside)
```plaintext
interface GigabitEthernet0/0
ip address 192.168.0.1 255.255.255.0
ip nat inside
no shutdown
description Rede Privada (Inside)
```

#### Configurar a Interface Serial0/3/0 (Rede Pública - Outside)
```plaintext
interface Serial0/3/0
ip address 180.1.6.2 255.255.255.192
ip nat outside
no shutdown
description Rede Pública (Outside)
```

### 3. Criar uma ACL para Definir os Endereços IP Privados

#### Crie uma lista de controle de acesso (ACL) para especificar os IPs da rede 192.168.0.0/24 que serão traduzidos:
```plaintext
access-list 1 permit 192.168.0.0 0.0.0.255
```

### 4. Configurar o NAT Dinâmico (Overload)

#### Configure o NAT dinâmico com sobreposição (PAT - Port Address Translation) para traduzir os endereços privados da rede 192.168.0.0/24 para o endereço IP público da interface Serial0/3/0:
```plaintext
ip nat inside source list 1 interface Serial0/3/0 overload
```

- **ip nat inside source list 1:** Define a lista de IPs privados que serão traduzidos.
- **interface Serial0/3/0:** Utiliza o IP da interface pública como endereço NAT.
- **overload:** Permite que vários endereços privados compartilhem o mesmo IP público usando diferentes portas.

### 5. Verificação e Teste

#### Verificar a Configuração do NAT
Use os seguintes comandos para verificar se o NAT está funcionando corretamente:
```plaintext
show ip nat statistics
```

### 6. Salvar a Configuração
```plaintext
write memory
```
