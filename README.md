# Configurando Bridge Network no Virt-Manager (KVM/QEMU)

## Objetivo

Configurar o Virt-Manager/KVM para que as máquinas virtuais utilizem a mesma rede física do host.

---

# Cenário

### Antes (NAT padrão)

```text
VM -> virbr0 (NAT) -> Host -> Rede
```

### Depois (Bridge)

```text
VM -> br0 -> Interface Física -> Rede
```

Nesse modo:

- As VMs recebem IP da mesma rede física
- Funcionam como máquinas reais na LAN
- Melhor para:
  - Laboratórios
  - Servidores
---

### 1. Verificar interface de rede

Dentro do seu debian rode o comando no terminal:

```bash
ip a
```

Exemplo:

```text
2: enp5s0:
```
No meu caso a minha placa de rede é a `enp5s0`

---

### 2. Criar bridge

```bash
sudo nmcli connection add type bridge ifname br0
```

---

### 3. Adicionar interface física na bridge

```bash
sudo nmcli connection add type bridge-slave ifname enp5s0 master br0
```

---

### 4. Configurar IP estático

```bash
sudo nmcli connection modify bridge-br0 \
ipv4.addresses 172.16.0.60/24 \
ipv4.gateway 172.16.0.254 \
ipv4.dns "1.1.1.1 8.8.8.8" \
ipv4.method manual
```

---

### 5. Desabilitar STP

```bash
sudo nmcli connection modify bridge-br0 bridge.stp no
```

---

### 6. Ver conexões existentes

```bash
nmcli connection show
```

Ex:
```
anderson@anderson:~$ nmcli connection show 
NAME UUID TYPE DEVICE bridge-br0 c0b28d55-e584-449f-953e-3084b1bac005 
bridge br0 Wired connection 1 ee06f3d1-cf8e-43ca-99b6-7780ca2a0e9f ethernet enp5s0
```

---

### 7. Derrubar conexão antiga

```bash
sudo nmcli connection down "Wired connection 1"
```

---

### 8. Subir bridge

```bash
sudo nmcli connection up bridge-br0
```

---

### 9. Verificar funcionamento

```bash
ip a
```

Resultado esperado:

```text
br0: <BROADCAST,MULTICAST,UP,LOWER_UP>
inet 172.16.0.60/24
```

---

### 10. Configurar VM no Virt-Manager

Selecionar:

- Network source:
  - `Bridge device`

Escolher:

```text
br0
```

Modelo:

```text
virtio
```

---

# Resultado Final

| Dispositivo | IP |
|---|---|
| Host Debian | 172.16.0.60 |
| VM Debian | 172.16.0.100 |
| VM Windows | 172.16.0.101 |

---

# Observações

O `virbr0` pode continuar existindo sem problemas.

As VMs apenas não devem utilizá-lo.
