# 🍎 Hackintosh — Dell Latitude 5400

> macOS Monterey 12.7.6 rodando em hardware Dell com OpenCore.  
> Guia prático e direto ao ponto, baseado em experiência real.

## 📦 O que tem aqui

Este repositório contém uma **EFI completa e funcional** para o Dell Latitude 5400 com as configurações descritas abaixo.

É um guia no estilo **Ctrl+C / Ctrl+V** — ou seja, basta copiar a EFI para o pendrive e seguir os passos. Sem precisar montar do zero, sem precisar configurar kexts manualmente. Funcionou aqui, deve funcionar igual no seu.

> ⚠️ **Usando outro modelo de Latitude?**  
> Não saia copiando sem verificar! Outros modelos podem ter hardware diferente — chipset de Wi-Fi, touchpad, áudio, etc. Nesse caso, **revise o `config.plist`** (especialmente o SMBIOS e as DeviceProperties) e **confira se os kexts batem com o seu hardware** antes de usar.

---

## 💻 Especificações do Hardware

| Componente | Detalhe |
|---|---|
| **Modelo** | Dell Latitude 5400 |
| **CPU** | Intel Core i5-8265U (Whiskey Lake) |
| **GPU** | Intel UHD Graphics 620 |
| **Wi-Fi** | Intel Wireless-AC 9560 |
| **Bluetooth** | Intel Bluetooth |
| **Armazenamento** | SSD SATA/NVMe |
| **Touchpad** | Alps HID |

---

## ✅ Status de Compatibilidade

| Função | Status |
|---|---|
| Boot | ✅ Funcional |
| CPU (Power Management) | ✅ Funcional |
| GPU (aceleração) | ✅ Funcional |
| Áudio | ✅ Funcional |
| Wi-Fi | ✅ Funcional (via HeliPort) |
| Bluetooth | ⚠️ A verificar |
| Trackpad | ✅ Funcional |
| Teclado | ✅ Funcional |
| Brilho da tela | ✅ Funcional |
| Leitor de cartão SD | ✅ Funcional |
| USB | ✅ Funcional |
| Sleep/Wake | ⚠️ Parcial |
| iMessage / FaceTime | ⚠️ Requer configuração de SMBIOS |

---

## 📋 Pré-requisitos

- Pendrive de **16GB ou mais**
- Acesso a um computador com **Windows ou Linux** para preparar o pendrive
- Conexão com a internet durante a instalação
- **OpenCore** (versão utilizada: 1.0.x)

---

## 🚀 Guia de Instalação

### 1. Preparar o Pendrive

Formate o pendrive com as seguintes configurações:

- **Esquema de partição:** GPT
- **Formato:** FAT32
- **Nome:** qualquer (ex: `OPENCORE`)

> ⚠️ O pendrive **deve** ser formatado como **UEFI + GPT**. Partições MBR ou Legacy não funcionarão.

---

### 2. Estrutura do Pendrive

Após preparar, o pendrive deve conter **duas pastas** na raiz:

```
PENDRIVE/
├── EFI/
│   ├── BOOT/
│   │   └── BOOTx64.efi
│   └── OC/
│       ├── config.plist
│       ├── OpenCore.efi
│       ├── ACPI/
│       ├── Drivers/
│       ├── Kexts/
│       ├── Resources/
│       └── Tools/
└── com.apple.recovery.boot/
    ├── BaseSystem.dmg
    └── BaseSystem.chunklist
```

**Para baixar o Recovery do macOS Monterey**, use o `macrecovery.py` incluso no OpenCore:

```bash
# Dentro da pasta Utilities/macrecovery do OpenCore:
python macrecovery.py -b Mac-E43C1C25D4880AD6 -m 00000000000000000 download
```

---

### 3. Kexts Utilizados

| Kext | Função |
|---|---|
| `Lilu.kext` | Base obrigatória |
| `VirtualSMC.kext` | Emulação de SMC |
| `WhateverGreen.kext` | Correções de GPU |
| `AppleALC.kext` | Áudio |
| `IntelMausi.kext` | Ethernet Intel |
| `RestrictEvents.kext` | Correções diversas |
| `AirportItlwm.kext` | Wi-Fi Intel (alternativa ao itlwm) |
| `itlwm.kext` | Wi-Fi Intel (recomendado — mais estável) |
| `IntelBluetoothFirmware.kext` | Bluetooth Intel |
| `BlueToolFixup.kext` | Correção Bluetooth |
| `VoodooPS2Controller.kext` | Teclado e trackpad |
| `VoodooI2C.kext` + `VoodooI2CHID.kext` | Touchpad avançado |
| `SMCBatteryManager.kext` | Status da bateria |
| `SMCDellSensors.kext` | Sensores Dell |
| `NVMeFix.kext` | Correção NVMe |
| `USBMap.kext` | Mapeamento USB customizado |

---

### 4. Configurações da BIOS

Acesse a BIOS do Latitude 5400 pressionando **F2** na inicialização.

**Desativar:**
- Secure Boot
- Fast Boot
- VT-d

**Ativar:**
- UEFI Boot
- XHCI Hand-off
- SATA Mode: **AHCI**

> ⚠️ Se o Windows foi instalado com SATA em modo **RAID/RST**, é necessário migrar para AHCI via registro antes de mudar a BIOS, caso contrário o Windows não iniciará.

---

### 5. Boot pelo Pendrive

1. Reinicie o notebook
2. Pressione **F12** para abrir o menu de boot
3. Selecione o pendrive em modo **UEFI**
4. No menu do OpenCore, selecione o Recovery do macOS
5. Siga o instalador normalmente

---

### 6. Instalação no SSD

Durante a instalação, formate o SSD pelo **Utilitário de Disco**:
- **Formato:** APFS
- **Esquema:** GUID

Após a instalação, copie a EFI do pendrive para o SSD via Terminal:

```bash
# Listar discos
diskutil list

# Montar EFI do SSD (substitua disk0s1 pelo identificador correto)
sudo diskutil mount disk0s1

# Montar EFI do pendrive
sudo diskutil mount disk2s1

# Copiar EFI
sudo cp -r /Volumes/OPENCORE/EFI /Volumes/EFI/
```

Após isso, o sistema iniciará **sem necessidade do pendrive**.

---

## 📶 Wi-Fi — Observação Importante

> O `AirportItlwm.kext` pode causar **Kernel Panic** ao tentar conectar em algumas redes.  
> A solução mais estável é usar o **`itlwm.kext`** em conjunto com o app **HeliPort**.

### Instalação do HeliPort

1. Baixe em: [github.com/OpenIntelWireless/HeliPort/releases](https://github.com/OpenIntelWireless/HeliPort/releases)
2. Instale o `.dmg` normalmente
3. Se aparecer aviso de segurança: clique com **botão direito → Open**
4. Para iniciar com o sistema:

```bash
osascript -e 'tell application "System Events" to make login item at end with properties {path:"/Applications/HeliPort.app", hidden:false}'
```

---

## 🛠️ Ferramentas Utilizadas

| Ferramenta | Link |
|---|---|
| OpenCore | [github.com/acidanthera/OpenCorePkg](https://github.com/acidanthera/OpenCorePkg/releases) |
| ProperTree | [github.com/corpnewt/ProperTree](https://github.com/corpnewt/ProperTree) |
| GenSMBIOS | [github.com/corpnewt/GenSMBIOS](https://github.com/corpnewt/GenSMBIOS) |
| MountEFI | [github.com/corpnewt/MountEFI](https://github.com/corpnewt/MountEFI) |
| HeliPort | [github.com/OpenIntelWireless/HeliPort](https://github.com/OpenIntelWireless/HeliPort/releases) |

---

## 📚 Referências

- [Dortania OpenCore Install Guide](https://dortania.github.io/OpenCore-Install-Guide/)
- [OpenIntelWireless — itlwm](https://github.com/OpenIntelWireless/itlwm)
- [Acidanthera Kexts](https://github.com/acidanthera)

---

## ⚠️ Aviso Legal

Este guia é fornecido apenas para fins educacionais.  
Instalar o macOS em hardware não-Apple viola o EULA da Apple.  
Use por sua conta e risco.

---

> Feito com ☕ e muita paciência.  
> Se te ajudou, deixa uma ⭐ no repositório!
