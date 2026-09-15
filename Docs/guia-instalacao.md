# HACKINTOSH — **Dell OptiPlex 3040** (i5-6500 · HD 530 · 8 GB DDR3L · HDD 500 GB)
## Guia definitivo com **OpCore Simplify** — macOS Monterey 12.7.6 · macOS como sistema único

> **Hardware confirmado por você:** Dell **OptiPlex 3040** · **i5-6500** · BIOS de **14/07/2022** · Windows 10 **UEFI/GPT** · sem Wi-Fi · sem SSD · **2 pendrives de 8 GB** · **sem dual boot**.

---

# ÍNDICE
- [Seção 0 — Ficha técnica e veredito de compatibilidade](#seção-0)
- [Seção 1 — Preparação e backup](#seção-1)
- [Seção 2 — BIOS/UEFI do OptiPlex 3040](#seção-2)
- [Seção 3 — Particionamento no Windows](#seção-3)
- [Seção 4 — OpCore Simplify](#seção-4)
- [Seção 5 — Pendrives e arquivos de instalação](#seção-5)
- [Seção 6 — Instalação do macOS](#seção-6)
- [Seção 7 — Pós-instalação](#seção-7)
- [Seção 8 — Troubleshooting preventivo](#seção-8)
- [Apêndice A — Config esperada · B — Checklist · C — Links](#apêndice)

---

<a id="seção-0"></a>
# SEÇÃO 0 — FICHA TÉCNICA E VEREDITO DE COMPATIBILIDADE

### 0.1 O que tem dentro do seu OptiPlex 3040 (dados do manual oficial Dell)

| Componente | Especificação | Fonte |
|---|---|---|
| **CPU** | Intel Core **i5-6500** — 4 núcleos / 4 threads, 3,2→3,6 GHz, 6 MB cache, 65 W, **AVX2 ✅** | Skylake, socket LGA1151 |
| **Chipset** | **Intel H110** | Manual OptiPlex 3040, Tabela 6 |
| **Memória** | **DDR3L 1600 MHz**, 2 slots UDIMM, máx. 16 GB (2×8) | Manual, Tabela 2 |
| **Vídeo** | **Intel HD Graphics 530** (GT2, device-id `0x1912`) | Manual, Tabela 3 |
| **Saídas de vídeo** | **HDMI** (19 pinos) + **DisplayPort** (20 pinos) + VGA opcional | Manual, Tabela 10 |
| **Áudio** | **Realtek HDA Codec ALC3234** = **ALC255** (VEN `10EC`, DEV `0255`) | Manual, Tabela 4 |
| **Rede** | **Realtek RTL8111HSD-CG** Gigabit 10/100/1000 | Manual, Tabela 5 |
| **USB** | **SFF:** 2× USB 2.0 frontais + 2× traseiras · 2× USB 3.0 frontais + 2× traseiras = **8 portas** | Manual, Tabela 10 |
| | **Mini Tower:** **sem USB 2.0 frontal**, 2× USB 2.0 traseiras · 2× USB 3.0 frontais + 2× traseiras = **6 portas** | Manual, Tabela 10 |
| **Outros** | Porta serial 9 pinos (opcional), PS/2 teclado/mouse, áudio frontal "Universal Audio Jack", line-out traseiro | Manual, Tabela 10 |
| **BIOS** | Chip NVRAM de 16 MB · versão mais recente publicada pela Dell: **1.20.1 (09/08/2022)** | support.dell.com |

### 0.2 Veredito — o que funciona, o que precisa de kext, o que não funciona

| Componente | Status | Solução |
|---|---|---|
| i5-6500 | ✅ **Nativo até o macOS Monterey 12** | `SSDT-PLUG` + quirk `AppleXcpmCfgLock` (CFG Lock é oculto na BIOS Dell) |
| HD Graphics 530 | ✅ Nativa até o Monterey | `Lilu` + `WhateverGreen` + `AAPL,ig-platform-id = 00001219` |
| H110 | ✅ | SSDTs: `PLUG`, `EC-USBX`, `XOSI`, **`HPET`** |
| **ALC3234** | ✅ **Existe layout feito para OptiPlex** | `AppleALC` + **`alcid=11`** |
| RTL8111HSD | ✅ | **`RealtekRTL8111.kext`** |
| HDMI + DisplayPort | ✅ | Nativos com o framebuffer `00001219` |
| USB (6–8 portas) | ⚠️ Dentro do limite de 15, mas o macOS 11.3+ quebra o `XhciPortLimit` | **`UTBMap.kext` gerado no Windows antes de instalar** |
| Sleep | ⚠️ Problemático em Skylake iGPU | `Block Sleep: Enabled` na BIOS + `pmset` |
| **CFG Lock** | ⚠️ **Oculto na BIOS Dell** — não dá para desligar pela interface | Quirk `AppleXcpmCfgLock = YES` (o OCS já aplica) |
| **DVMT Pre-Allocated** | ⚠️ **Oculto, padrão 32 MB** | Funciona em 1080p. Se der tela preta: patch `framebuffer-stolenmem` |
| Wi-Fi | ✅ Inexistente no seu PC — problema zero | — |
| **Porta serial** | ⚠️ Causa tela preta / travamento em OptiPlex com Hackintosh | **Desativar na BIOS** (Passo 8) |
| macOS **Ventura 13+** | ⚠️ Só com *spoof* Kaby Lake | **Não use** — ver 0.3 |
| macOS **Sequoia 15** | ❌ Precisa de spoof + root patches do OCLP | **Não use** |

### 0.3 Por que macOS **Monterey 12.7.6** e não outro

| Versão | Veredito | Motivo |
|---|---|---|
| **Monterey 12.7.6** | ✅ **ESCOLHIDA** | Última versão com suporte **nativo** a Skylake + HD 530. SMBIOS `iMac17,1` (que é literalmente um Mac com i5-6500 + HD 530), **zero spoof, zero OCLP, zero patch de kernel**. |
| Big Sur 11.7.11 | 🟡 Plano B | Funciona nativo, mas obsoleto (11.7.11, fev/2026, só renovou certificados). Apps e Safari atuais já o abandonaram. |
| Ventura 13 | 🟠 Evite | Exige spoof KBL (`device-id 59120000`, framebuffer `00001259`, SMBIOS iMac18,x). Mais pontos de falha em áudio/sleep. |
| Sonoma 14 / Sequoia 15 | ❌ Não | Spoof KBL **+** root patches do OpenCore Legacy Patcher. Em 8 GB + HDD 5400 rpm, risco alto de boot loop e sistema inutilizável. |

### 0.3.1 ⚠️ "Mas o OCS diz que suporta macOS Tahoe 26!" — por que isso NÃO se aplica ao seu caso

O OCS lista `macOS High Sierra → macOS Tahoe` na tabela de compatibilidade. Aquilo significa **"a ferramenta consegue gerar um EFI para essa versão"**, não **"isso vai funcionar no seu OptiPlex 3040"**. Quatro barreiras independentes, cada uma suficiente para desqualificar:

**1. O suporte a Tahoe no OCS depende de um FORK do OCLP, não do oficial**
O próprio README do OCS avisa:
> *"Only OpenCore-Patcher **3.0.0 from the lzhoang2801/OpenCore-Legacy-Patcher** repository provides support for macOS Tahoe 26 with early patches. **Official Dortania releases or older patches will NOT work** with macOS Tahoe 26."*

Verifiquei nos dois repositórios:

| Repositório | Versão atual | Data |
|---|---|---|
| `dortania/OpenCore-Legacy-Patcher` (**oficial**) | **2.5.0** | 08/09/2026 |
| `lzhoang2801/OpenCore-Legacy-Patcher` (**fork pessoal**) | 3.0.0 | 02/12/2025 |

Ou seja: Tahoe exige um patcher mantido por **uma pessoa só**, com patches descritos pelo próprio OCS como *"early patches"* (estágio inicial). Não é a equipe do OCLP.

**2. Sua HD 530 simplesmente não tem driver no Tahoe**
A Apple removeu o suporte às iGPUs Skylake a partir do **Sonoma**. O que existe é gambiarra em duas camadas:
- *spoof* para Kaby Lake (WhateverGreen ≥ 1.6.1 + `device-id` KBL + SMBIOS `iMac18,x`)
- **e** root patches do OCLP para reinstalar os drivers gráficos

E o repositório que documenta exatamente o seu cenário (`5T33Z0/OCLP4Hackintosh`, guia Skylake) abre com:
> ⚠️ *"Don't install macOS Tahoe if you don't have a compatible iGPU/GPU in your system!"*

Sem aceleração gráfica, o macOS Tahoe é inutilizável — não é "fica lento", é interface travada, sem transparência, sem vídeo decente.

**3. O aviso mais forte vem dos próprios desenvolvedores do OCLP**
> *"**WARNING!** Installing macOS Tahoe on unsupported hardware is **NOT** supported by the OpenCore Legacy Patcher developers. Numerous users have attempted installation, often resulting in serious issues including **hardware malfunctions and complete data loss**."*

Você está justamente no cenário que esse aviso descreve: hardware não suportado + disco único + sem dual boot.

**4. O desenvolvimento do OCLP está praticamente parado**
O issue de rastreamento do suporte a Tahoe (`dortania/OpenCore-Legacy-Patcher#1167`) continua aberto **sem milestone**. O desenvolvedor principal do projeto foi contratado pela Apple, e a comunidade relata que o ritmo caiu a quase zero. Não há data prevista.

**Some a isso o seu hardware:** 8 GB de RAM + HDD 5400 rpm já são o gargalo no Monterey. O Tahoe, com a interface Liquid Glass, é bem mais pesado que o Monterey. Mesmo se tudo funcionasse, seria uma experiência ruim.

**🟢 Veredito: fique no Monterey 12.7.6.**

**Se a curiosidade for grande, o caminho seguro é este:**
1. Instale o **Monterey** e deixe funcionando 100%.
2. Só então, se quiser, crie uma **segunda partição APFS** de ~80 GB e tente o Tahoe ali.
3. Com a **imagem de disco do Passo 2** à mão, o risco fica contido e reversível.

⚠️ Mas nunca no disco inteiro, nunca antes do Monterey estar estável, e nunca sem backup validado.

### 0.4 Problemas CONHECIDOS deste modelo específico

1. **`alcid=11` é o layout certo.** No banco de dados do próprio OCS, o codec `10EC-0255` tem a entrada literal: *"Realtek ALC255(3234) for Dell Optiplex series by Heporis"* → **id 11**. Alternativas no mesmo banco: **12** (*"ALC255, Dell Optiplex 7040 MT"*) e **66** (*"Dell Optiplex 7060/7070MT (Separate LineOut)"* — use só se o line-out traseiro precisar ser tratado como saída separada).
2. **O line-out traseiro (conector azul) do 3040** é historicamente o mais chato. Se o fone frontal funcionar e o traseiro não, teste `alcid=66`.
3. **Porta serial** causa tela preta após o boot em OptiPlex. **Desative.**
4. **CFG Lock e DVMT** ficam escondidos na BIOS Dell. Não tente destravar com shell modificado (risco de brick) — o quirk `AppleXcpmCfgLock` resolve o CFG Lock sem tocar no firmware.
5. **Sleep** em Skylake iGPU é instável no Monterey. Não é bloqueante: bloqueie o sleep e use o PC ligado.
6. **APFS em HDD 5400 rpm** = sistema lento. Instalação leva 1,5–3 h. É limitação física, não bug.

### 0.5 Material necessário

- [ ] **Pendrive 1 — 8 GB** (instalador). USB 2.0 de preferência; vai em porta **USB 2.0 traseira (preta)**.
- [ ] **Pendrive 2 — 8 GB** (recuperação do Windows 10)
- [ ] **Pendrive 3 — qualquer tamanho** (só para atualizar a BIOS — depois você reutiliza o 1 ou o 2)
- [ ] HD externo ou nuvem para o backup
- [ ] **Teclado e mouse USB com fio** (o 3040 tem PS/2, mas PS/2 não funciona no instalador do macOS)
- [ ] Cabo de rede no RJ-45
- [ ] Monitor no **HDMI ou DisplayPort** (evite adaptador passivo DP→VGA)
- [ ] 2–3 h para preparar + 2–4 h de instalação

---

<a id="seção-1"></a>
# SEÇÃO 1 — PREPARAÇÃO E BACKUP

### ⚠️ PASSO 1 — Backup dos dados pessoais
- **O quê:** copiar tudo que importa para fora do PC.
- **Ferramenta:** Explorador de Arquivos + HD externo / nuvem.
- **Como:**
  1. Copie a pasta `C:\Users\SEU_USUARIO` inteira.
  2. Copie também: favoritos/perfis de navegador, PST de e-mail, licenças e chaves de programas, pasta `Documentos` de outros usuários.
  3. Anote senhas salvas no navegador (ou sincronize com sua conta).
- ⚠️ **Você vai apagar o Windows no Passo 28. Depois disso não existe volta.**
- **Resultado esperado:** os arquivos abrem a partir do backup em outro dispositivo. Teste 2 ou 3 arquivos, não confie só na cópia.

### ⚠️ PASSO 2 — Imagem completa do disco (sua rede de segurança real)
- **O quê:** clone bit a bit do Disco 0.
- **Ferramenta:** **Macrium Reflect** (trial) ou **AOMEI Backupper Standard** (gratuito).
- **Como:**
  1. Instale o programa no Windows.
  2. Crie uma **Image Backup** de **todo o Disco 0**, marcando **"incluir partições ocultas"** (EFI + Recovery da Dell).
  3. Salve no HD externo.
  4. Gere também o **USB de recuperação** do próprio programa (se couber no pendrive 2, melhor).
- **Resultado esperado:** arquivo `.mrimg`/`.adi` de ~100–300 GB no HD externo. **É isto que devolve o PC exatamente ao estado atual em ~30 min.**

### ⚠️ PASSO 3 — Desativar Fast Startup e hibernação
*Por que existe:* com Fast Startup ligado o Windows deixa o NTFS marcado como "sujo". O macOS recusa montar, e você pode corromper dados ao escrever ali.
- **O quê:** desligar inicialização rápida e hibernação.
- **Ferramenta:** CMD como Administrador + Painel de Controle.
- **Como:**
  1. CMD como admin → `powercfg /h off` → Enter.
  2. Painel de Controle → **Opções de Energia** → *Escolher o que os botões de energia fazem* → *Alterar configurações não disponíveis no momento* → **desmarcar "Ligar inicialização rápida"** → salvar.
- **Resultado esperado:** `C:\hiberfil.sys` deixa de existir e a opção some do menu de energia.

### PASSO 4 — Conferir modo de boot e estilo de partição
- **O quê:** confirmar UEFI + GPT (pré-requisito absoluto).
- **Ferramenta:** `msinfo32` + `diskpart`.
- **Como:**
  1. `Windows + R` → `msinfo32` → Enter. Em **Resumo do Sistema**, confira **Modo da BIOS = UEFI**.
  2. Anote também: **Modelo do sistema** (`OptiPlex 3040`), **Processador** (`i5-6500`), **Versão/data do BIOS**.
  3. Opcional: **Arquivo → Exportar…** e salve um `.txt` para consultar depois.
  4. CMD como admin → `diskpart` → `list disk` → `select disk 0` → `detail disk` → confira **Estilo de Partição = GPT** → `exit`.
- ⚠️ **Não digite mais nada no diskpart.** `clean`, `create` e `format` apagam dados.
- **Resultado esperado:** UEFI + GPT confirmados. ⚠️ **Se aparecer `Legacy` ou `MBR`, PARE e me avise** — o fluxo de particionamento muda completamente.
- ℹ️ Se o Windows estiver em Modo de Segurança, o "Modo da BIOS" pode aparecer errado. Reinicie normalmente e confira de novo.

### ⚠️ PASSO 5 — Atualizar a BIOS para a última versão
*Por que existe:* sua BIOS é de 14/07/2022. A última publicada pela Dell para o OptiPlex 3040 é a **1.20.1, de 09/08/2022**, marcada como **Importância: Crítica** (correções de CVE). Fazer isso **antes** de mexer em partições elimina uma variável de risco — e o 3040 é um modelo encerrado, então não haverá outra versão depois.
- **O quê:** flash de BIOS.
- **Ferramenta:** `support.dell.com` + pendrive 3 (não precisa ser bootável).
- **Como:**
  1. Em `https://www.dell.com/support/home/pt-br`, busque **OptiPlex 3040** (ou informe a Service Tag da etiqueta do gabinete).
  2. Drivers e downloads → categoria **BIOS** → baixe a versão mais recente (arquivo no formato `OptiPlex_3040_1.XX.X.exe`).
  3. **Confira o SHA-256** publicado na página contra o arquivo baixado (CertUtil no Windows: `certutil -hashfile arquivo.exe SHA256`).
  4. Copie o `.exe` para o pendrive 3.
  5. Reinicie pressionando **F12** repetidamente → em **Other Options**, escolha **BIOS Flash Update** → botão **`...`** → selecione o arquivo no pendrive → **Begin Flash Update** → **Yes**.
  6. **Não desligue nem desconecte nada durante o flash.**
- **Resultado esperado:** na tela inicial do Dell, a versão passa a ser a nova. O Windows 10 continua iniciando normalmente.
- ⚠️ **Se você não se sente confortável fazendo flash de BIOS, pule este passo.** Não é obrigatório para o Hackintosh funcionar. Nunca faça flash "modificado" para destravar CFG Lock — risco real de brick.

### ⚠️ PASSO 6 — Criar o pendrive 2: recuperação do Windows 10
*Por que existe:* sem dual boot, se o macOS não der boot você fica sem sistema nenhum. Este pendrive é o único caminho de volta.
- **O quê:** instalador oficial do Windows 10 em USB.
- **Ferramenta:** `https://www.microsoft.com/pt-br/software-download/windows10` → **Baixar a ferramenta agora** (Media Creation Tool).
- **Como:** rode a ferramenta → **Criar mídia de instalação (USB)** → selecione o **pendrive 2** → aguarde.
- ⚠️ **Não formate mais este pendrive durante o processo.**
- **Resultado esperado:** pendrive 2 bootável em UEFI com o instalador do Windows 10. Guarde-o.

---

<a id="seção-2"></a>
# SEÇÃO 2 — BIOS/UEFI DO OPTIPLEX 3040

### ⚠️ PASSO 7 — Entrar na BIOS e carregar os padrões
- **Ferramenta:** tecla **F2** repetidamente no logo Dell.
- **Como:** navegue até **Exit** (ou "Restore Settings") → **Load Defaults** / **Factory Settings** → confirme → depois aplique os ajustes do Passo 8.
- **Resultado esperado:** BIOS em estado limpo, sem configuração herdada.

### ⚠️ PASSO 8 — Aplicar as configurações (menu Dell, caminho exato)

| # | Caminho no menu Dell | Ajustar para | Por quê |
|---|---|---|---|
| 1 | **General → Boot Sequence → Boot List Option** | **UEFI** | Seu Windows é UEFI. Legacy quebra o boot. |
| 2 | **General → Advanced Boot Options → Enable Legacy Option ROMs** | **Desmarcado** | CSM ligado causa falha de inicialização da GPU |
| 3 | **System Configuration → SATA Operation** | **AHCI** | Padrão Dell. ⚠️ **Se estiver em "RAID On", NÃO mude** — o Windows dá `INACCESSIBLE_BOOT_DEVICE` |
| 4 | **System Configuration → Serial Port** | **Disabled** | ⚠️ **Causa tela preta após o boot em OptiPlex** |
| 5 | **System Configuration → Integrated NIC** | **Enabled** (com "w/ PXE" ou não) | Sem isso, sem Ethernet no macOS |
| 6 | **System Configuration → USB Configuration** | **Tudo habilitado** | Todas as portas precisam estar ativas |
| 7 | **System Configuration → Audio** | **Enabled** | Necessário para o AppleALC |
| 8 | **System Configuration → Miscellaneous Devices** | Deixar habilitado | — |
| 9 | **Security → Secure Boot → Secure Boot Enable** | **Disabled** | OpenCore não é assinado → `Security Violation` |
| 10 | **Security → PTT Security → PTT On** | **Disabled** | Não suportado pelo macOS |
| 11 | **Security → Intel Software Guard Extensions → Intel SGX Enable** | **Disabled** | Não suportado pelo macOS |
| 12 | **Security → CPU XD Support** | **Enabled** | Deixar ligado |
| 13 | **Performance → Intel SpeedStep** | **Enabled** | Necessário para o XCPM (gerenciamento de energia) |
| 14 | **Performance → C-States Control** | **Enabled** | Idem |
| 15 | **Performance → Intel TurboBoost** | **Enabled** | Idem |
| 16 | **Power Management → USB Wake Support** | **Disabled** | Evita wake espontâneo |
| 17 | **Power Management → Wake on LAN/WLAN** | **Disabled** (ou "LAN only") | Idem |
| 18 | **Power Management → Deep Sleep Control** | **Disabled** | Evita USB/teclado mortos após suspensão |
| 19 | **Power Management → Block Sleep** | **Enabled** ⚠️ | Skylake iGPU tem sleep instável. Volte aqui no fim se quiser suspender de verdade |
| 20 | **POST Behavior → Fastboot** | **Minimal** | "Full/Thorough" completo impede o OpenCore |
| 21 | **Virtualization Support → Virtualization** | **Enabled** | — |
| 22 | **Virtualization Support → VT for Direct I/O (VT-d)** | **Disabled** | Recomendação consolidada da comunidade em Dell. (Se você precisa de VT-d ligado, deixe enabled — o OCS aplica `DisableIoMapper: YES`) |
| 23 | **Maintenance → Service Tag / Asset Tag** | Não mexer | — |

- **Como:** aplique item por item. Ao terminar, **Apply** → **Exit**.
- **Resultado esperado:** o Windows 10 inicia normalmente com a nova BIOS. **Se não iniciar, volte na BIOS e revise antes de continuar.**

### PASSO 9 — Anotar o mapa de portas USB do seu gabinete
*Por que existe:* no Passo 17 você vai mapear as portas. Saber onde elas estão fisicamente evita errar.
- **O quê:** desenhar/anotar o layout.
- **Ferramenta:** papel ou o bloco de notas.
- **Como:** de acordo com o manual do OptiPlex 3040:
  - **SFF:** 2× USB 2.0 **frontais** + 2× USB 2.0 **traseiras** + 2× USB 3.0 **frontais** + 2× USB 3.0 **traseiras** = **8 portas**
  - **Mini Tower:** **não há USB 2.0 frontal** → 2× USB 2.0 traseiras + 2× USB 3.0 frontais + 2× USB 3.0 traseiras = **6 portas**
- **Resultado esperado:** você sabe exatamente quantas portas vai testar.
- ℹ️ As 2.0 e as 3.0 traseiras ficam lado a lado; as 3.0 são azuis. Marque-as com fita antes de começar.

---

<a id="seção-3"></a>
# SEÇÃO 3 — PARTICIONAMENTO NO WINDOWS

> **Regra de ouro:** toda alteração de partição é feita **aqui, no Windows**. No macOS você só vai apagar a partição que você mesmo criou.

### ⚠️ PASSO 10 — Fotografar o estado atual do disco
- **O quê:** registrar as partições antes de mexer.
- **Ferramenta:** `diskmgmt.msc` (Gerenciamento de Disco).
- **Como:** abra o Gerenciamento de Disco e **tire um print** do Disco 0. Num OptiPlex 3040 com Windows 10 de fábrica você deve ver algo como:
  ```
  Disco 0 (GPT, 465 GB)
  ├── Partição de Sistema EFI        ~ 100–500 MB
  ├── (MSR - Reservada)              ~  16 MB
  ├── C:  (Windows)                  ~ 460 GB
  └── Partição de recuperação        ~ 500 MB – 1 GB
  ```
- **Resultado esperado:** print salvo + anotação de quantos GB livres há no `C:`.

### ⚠️ PASSO 11 — Encolher o Windows e criar a partição `MACINST`
- **O quê:** abrir 150 GB para o macOS e rotulá-los como exFAT.
- **Ferramenta:** Gerenciamento de Disco (ou **MiniTool Partition Wizard Free** se o Windows não deixar encolher o suficiente).
- **Como:**
  1. Botão direito em **`C:`** → **Diminuir Volume**.
  2. Em "Digite o espaço a diminuir" informe **150000** (≈150 GB) → **Diminuir**.
     - ⚠️ Se o Windows oferecer menos de 100 GB por causa de arquivos imoveis, **não force**: rode `powercfg /h off` (Passo 3) e repita, ou use o MiniTool.
  3. Botão direito no **espaço não alocado** → **Novo Volume Simples** → Avançar → tamanho total → letra (ex.: `E:`) → formato **exFAT** → rótulo **`MACINST`**.
  4. ⚠️ **Se o assistente mostrar "Tamanho da unidade de alocação", escolha `1024 bytes` (padrão).** O macOS **não monta** exFAT com unidade de alocação acima de 1024 KB (limitação documentada no README do UnPlugged). Se errar isso, o Passo 24 falha.
  5. Finalizar.
- **Resultado esperado:**
  ```
  Disco 0 (GPT, 465 GB)
  ├── EFI            (intacta)
  ├── MSR            (intacta)
  ├── C:  Windows    (~310 GB, mesma letra, inicia normalmente)
  ├── MACINST exFAT  (~150 GB)   ← NOVA
  └── Recuperação    (intacta)
  ```
  Confira contra o print do Passo 10: **nada sumiu.**

---

<a id="seção-4"></a>
# SEÇÃO 4 — OPENCORE SIMPLIFY (OCS)

### PASSO 12 — Baixar o OCS da fonte oficial
- **O quê:** baixar a ferramenta.
- **Fonte única legítima:** `https://github.com/lzhoang2801/OpCore-Simplify`
  - Autor: **lzhoang2801** (Hoang Hong Quan) · Licença BSD-3-Clause
  - ⚠️ **Existem dezenas de forks falsos** (`firste2026/OpCore-SimplifyGUI-`, `w1842893/OpCore-Simplify2`, `davidtran1010/opcore-simplify`, `missyouling/OpCore-Simplify-GUI`, `Toxicyenom/...`, `MultimediaLucario/...`). **Nenhum é o oficial.** Confira a URL caractere por caractere.
- **Como:** botão verde **`<> Code`** → **Download ZIP**. (O projeto **não publica Releases** — o ZIP do branch `main` é o método oficial.) Extraia em `C:\OCS` (caminho **sem acento e sem espaço**).
- **Resultado esperado:** `C:\OCS\OpCore-Simplify-main\OpCore-Simplify.bat`

### PASSO 13 — Instalar o Python 3
*Por que existe:* o OCS e o `macrecovery.py` são scripts Python.
- **Ferramenta:** CMD normal.
- **Como:**
  ```
  winget install -e --id Python.Python.3.12
  ```
  (alternativa: `python.org`, marcando **"Add python.exe to PATH"**)
  Feche e reabra o CMD, teste com `python --version`.
- **Resultado esperado:** imprime `Python 3.12.x`.

### PASSO 14 — Rodar o OCS e exportar o relatório de hardware
*Por que existe:* o OCS lê as tabelas ACPI e o PCI do seu PC real. É isso que faz ele acertar `RealtekRTL8111` e `ALC255` sem você digitar nada.
- **Ferramenta:** `C:\OCS\OpCore-Simplify-main\OpCore-Simplify.bat`
- **Como:**
  1. Rode o `.bat`. Se o SmartScreen bloquear: **Mais informações → Executar assim mesmo**.
  2. Menu principal → **`E. Export hardware report`**.
  3. **Saia do programa quando ele pedir** — ele precisa reiniciar para reler o ACPI com o dump carregado.
  4. Reabra o `.bat` e selecione o relatório exportado.
  5. Leia o **Compatibility Checker**.
- **Resultado esperado:** o checker lista e identifica:
  - `Intel Core i5-6500`
  - `Intel HD Graphics 530` (device-id `0x1912`)
  - `Realtek RTL8111HSD` → deve incluir `RealtekRTL8111.kext`
  - `Realtek ALC255/ALC3234` (VEN `10EC`, DEV `0255`)
  - **Salve um print desta tela.** Se precisar pedir ajuda em fórum, é o documento mais útil.
- ⚠️ Se a rede aparecer como **Intel I219** em vez de Realtek, ok — o OCS inclui `IntelMausi.kext`. Só anote.

### ⚠️ PASSO 15 — Escolher a versão do macOS no OCS
- **O quê:** forçar o Monterey.
- **Como:** no menu de seleção de versão, escolha **Monterey (12)**.
- ⚠️ **CRÍTICO: o OCS seleciona por padrão "a versão compatível mais recente"**, que neste hardware vem com *spoof* Kaby Lake (Ventura/Sonoma/Sequoia). **Não aceite o padrão.**
- **Resultado esperado:** cabeçalho do OCS mostrando **macOS 12 / Monterey**.

### ⚠️ PASSO 16 — Configurar as opções do OCS (atenção máxima)

| Opção no OCS | Configuração para o OptiPlex 3040 |
|---|---|
| **ACPI (tela "Customize ACPI Patch Selections")** | Seleção final correta para o 3040 — **manter os que o OCS pré-marcou** (`[*]`): `4 BUS0`, `6 FakeEC`, `12 MCHC`, `15 PLUG`, `24 USBX` — e **digitar `9, 26`** para adicionar: `9 FixHPET` ⚠️ (conflitos de IRQ que impedem o ALC3234 de carregar em Dell) e `26 XOSI` (habilita dispositivos travados por cheque de OS). **Nunca marcar:** `5 Disable Devices` (desativaria sua HD 530!), `13 PMC`/`19 RTCAWAC` (só série 300+), `14 PM Legacy` (só Ivy e anteriores), `2 APIC`/`18 RTC0`/`22 UNC` (só HEDT), `1 ALS`/`3 BATP`/`10 GPIO`/`16 PNLF` (só laptop), `23 USB Reset` (só mapeamento no macOS), `11 IMEI`/`17 RMNE` (3040 tem ME e Ethernet reais). Tela final: `[*]` em **4, 6, 9, 12, 15, 24, 26** — e só esses. Depois `B` (Back). |
| **SMBIOS** | **`iMac17,1`** — é literalmente o iMac 2015 com **i5-6500 + HD 530**. ⚠️ Se vier `iMac18,1`, `iMac18,2` ou `Macmini8,1`, **troque manualmente** — é sinal de que ele está aplicando spoof KBL. |
| **iGPU / framebuffer** | `AAPL,ig-platform-id = 00001219` (HD 530 desktop dirigindo display). **NÃO** deve existir `device-id = 12590000`, `AAPL,ig-platform-id = 00001259` nem boot-arg `-igfxsklaskbl`. |
| **Codec Layout ID** | Escolha **"Realtek ALC255(3234) for Dell Optiplex series by Heporis" → id `11`** |
| **Kext de rede** | Confirmar `RealtekRTL8111.kext` na lista |
| **Kernel Quirks** | `AppleXcpmCfgLock = YES` (o CFG Lock é oculto na BIOS Dell) · `XhciPortLimit = **NO**` |
| **Misc → Boot → LauncherOption** | **`Full`** — registra o OpenCore como entrada de boot permanente na NVRAM (essencial, pois não haverá dual boot) |
| **"Disable SIP"** | ⚠️ **Desmarcar se possível.** No Monterey nativo você não precisa desligar o SIP; ligado é mais seguro e o iServices é mais previsível. |
| **"Force Intel GPU into VESA mode" / `-radvesa`** | **Desmarcado.** VESA = sem aceleração gráfica. |
| **Wi-Fi / itlwm / AirportItlwm** | Não aplicável (3040 não tem Wi-Fi). Ignorar se o OCS oferecer. |
| **Boot-args** | `-v keepsyms=1 alcid=11` |

- **Resultado esperado:** o resumo na tela bate item por item com esta tabela.

### PASSO 17 — Gerar o EFI
- **Ferramenta:** opção **`Build OpenCore EFI`** no OCS.
- **Como:** selecione e aguarde. Ele baixa o OpenCorePkg e os kexts direto das fontes oficiais (Dortania Builds e releases do GitHub).
- **Resultado esperado:** pasta `EFI/` criada, com `EFI/BOOT/` e `EFI/OC/`.

### ⚠️ PASSO 18 — Mapear as portas USB ANTES de instalar
*Por que existe:* no macOS 11.3+ o quirk `XhciPortLimit` é quebrado e causa **boot loop**. A Dortania recomenda explicitamente mapear as portas **antes** de instalar o macOS 12+. Fazer isso no Windows é o método mais confiável (vê todas as portas de uma vez).
- **Ferramentas:**
  - USBToolBox (ferramenta): `https://github.com/USBToolBox/tool/releases` → **`Windows.zip`** (o `.exe` solto às vezes é barrado pelo antivírus)
  - USBToolBox (kext): `https://github.com/USBToolBox/kext/releases` → `USBToolBox-1.2.0-RELEASE.zip`
- **Como:**
  1. Abra `Windows.exe` → **`D` (Discover Ports)**.
  2. Espete um pendrive em **cada porta USB do gabinete**, uma de cada vez, esperando a porta acender na lista. No Windows basta **1 dispositivo por porta** (a detecção da porta companheira 2.0/3.0 é automática).
  3. **`S` (Select Ports and Build Kext)** → **`P` (Enable All Populated Ports)** — seleciona **só as portas que você testou**. ⚠️ **Não use `A` (Select All):** ele marca também as portas "fantasma" que o controlador H110 reporta mas que não existem no gabinete, e a validação reclama `Port X is missing a connector type`.
  4. Confira o total (**≤ 15**). Se aparecer a tela `Selection Validation` com "missing a connector type" (tipicamente portas 11–16 no 3040): digite `B`, depois os números das portas **não populadas** separados por vírgula (ex.: `11, 12, 13, 14, 15, 16`) para desmarcá-las, e `K` de novo. Se alguma porta **populada** ficar sem tipo, defina manualmente com a sintaxe `T:<portas>:<tipo>` — USB 2.0 preta = `0`, USB 3.0 azul (e a companheira dela) = `3`, interna = `255` (ex.: `T:7,8:0`).
  5. **`K` (Build UTBMap.kext)**.
- **Resultado esperado:** `UTBMap.kext` na pasta `dist`.
- **Integrar ao EFI:**
  1. Copie `UTBMap.kext` para `EFI/OC/Kexts/`.
  2. No `config.plist`, `Kernel → Add`: adicione a entrada com `BundlePath = UTBMap.kext`, **`ExecutablePath` vazio**, `PlistPath = Contents/Info.plist`.
  3. Ordem na lista: **`USBToolBox.kext` ANTES de `UTBMap.kext`**.
  4. Confirme `XhciPortLimit = false`.
  5. ⚠️ **Remova `UTBDefault.kext`** de `EFI/OC/Kexts/` **e a entrada dele em `Kernel → Add`** — `UTBDefault` (todas as portas) e `UTBMap` (seu mapa) **conflitam**. O `UTBDefault` que o OCS marca serve **só para a fase de instalação**.

### PASSO 19 — Validar o EFI antes de tentar dar boot

**19.1 Estrutura de pastas**
```
EFI/
├── BOOT/BOOTX64.efi
└── OC/
    ├── config.plist
    ├── ACPI/     → SSDT-EC-USBX.aml, SSDT-PLUG.aml, SSDT-XOSI.aml, SSDT-HPET.aml, SSDT-RTC...
    ├── Drivers/  → OpenRuntime.efi, HfsPlus.efi (+ OpenCanopy.efi opcional)
    ├── Kexts/    → ver 19.2
    └── Tools/    → OpenShell.efi, ocvalidate.exe (você copia no 19.5)
```

**19.2 Kexts obrigatórios em `EFI/OC/Kexts/`**
- [ ] `Lilu.kext`
- [ ] `VirtualSMC.kext`
- [ ] `WhateverGreen.kext`
- [ ] `AppleALC.kext`
- [ ] `RealtekRTL8111.kext`
- [ ] `USBToolBox.kext` + `UTBMap.kext`

**19.3 `config.plist`** — como abrir: baixe o **ProperTree** (`https://github.com/corpnewt/ProperTree`), extraia e rode `python ProperTree.py` no CMD. ⚠️ Se o duplo clique em `.py` abrir a **Microsoft Store**: Configurações → Aplicativos → Configurações avançadas de aplicativo → **Aliases de execução de aplicativo** → desative `python.exe`/`python3.exe` do "Instalador de Aplicativos". Se `python` "não for reconhecido", use o caminho completo: `"%LocalAppData%\Programs\Python\Python312\python.exe" ProperTree.py`. **Emergência (edição de uma linha):** Bloco de Notas — troque só o `<string>...</string>` logo abaixo de `<key>BundlePath</key>` e valide com o OCValidate depois.
- [ ] Abre **sem erro de XML** (se der erro de parse, o arquivo está corrompido → gere o EFI de novo)
- [ ] `Kernel → Add` lista **todos** os kexts da pasta, com `ExecutablePath` correto
- [ ] `Kernel → Quirks → AppleXcpmCfgLock` = **true**
- [ ] `Kernel → Quirks → XhciPortLimit` = **false**
- [ ] `DeviceProperties → PciRoot(0x0)/Pci(0x2,0x0)` → `AAPL,ig-platform-id` = `00001219`
- [ ] `PlatformInfo → Generic → SystemProductName` = **`iMac17,1`**, com `MLB`, `SystemSerialNumber` e `SystemUUID` preenchidos e diferentes entre si
- [ ] `Misc → Boot → LauncherOption` = **Full**
- [ ] `Misc → Security → ScanPolicy` = **0** ⚠️ (valores restritivos como `17760515` fazem o OpenCore esconder volumes APFS internos — inclusive o `macOS Installer` no meio da instalação)
- [ ] `NVRAM → 7C436110-AB2A-4BBB-A880-FE41995C9F82 → boot-args` contém **`-v keepsyms=1 alcid=11`**

**19.4 ⚠️ Validar o serial (evita conflito com um Mac de verdade e bloqueio de iServices)**
1. Baixe o **GenSMBIOS**: `https://github.com/corpnewt/GenSMBIOS` → `GenSMBIOS.bat` → opção **1** → digite **`iMac17,1`**.
2. Os valores vão na seção **`PlatformInfo → Generic`** do `config.plist` (**não** no NVRAM!): `Serial` → **`SystemSerialNumber`**, `Board Serial (MLB)` → **`MLB`**, `SmUUID` → **`SystemUUID`**. O `SystemProductName` permanece **`iMac17,1`**. Troque só o conteúdo entre `<string>` e `</string>`.
3. Consulte o serial em `https://checkcoverage.apple.com`.
   - ✅ **"Número de série inválido"** = perfeito: não pertence a um Mac real.
   - ❌ Apareceu um Mac real → **gere outro serial.**

**19.5 Validação oficial com o OCValidate**
`OCValidate.exe` **não** vem dentro da pasta `EFI` — ele está no zip do OpenCorePkg, em `Utilities/ocvalidate/ocvalidate.exe` (verificado no `OpenCore-1.0.7-RELEASE.zip`).
1. Do zip do OpenCorePkg (baixado no Passo 20), extraia **apenas** esse `.exe` e **copie para `EFI\OC\`**.
2. Depois de montar o pendrive (Passo 22), dê boot por ele e selecione **OpenShell** no menu do OpenCore (pressione `Espaço` se estiver oculto).
3. No shell:
   ```
   fs0:
   cd EFI\OC
   OCValidate.exe config.plist
   ```
4. **Resultado esperado:** `No issues found.` Se aparecer qualquer erro, **não instale** — corrija no OCS, gere o EFI de novo e valide de novo.

---

<a id="seção-5"></a>
# SEÇÃO 5 — PENDRIVES E ARQUIVOS DE INSTALAÇÃO

> ⚠️ **Não baixe "ISO de macOS" de sites aleatórios.** São imagens modificadas, sem verificação, com histórico de malware. Tudo abaixo vem **direto dos servidores da Apple** via `gibMacOS` + `macrecovery`.

### ⚠️ PASSO 20 — Baixar os arquivos oficiais do macOS Monterey
- **Ferramenta A — gibMacOS:** `https://github.com/corpnewt/gibMacOS` → **Code → Download ZIP** → extraia em `C:\gibMacOS`.
  - ⚠️ **Aviso do próprio autor no README do gibMacOS:** *"outros estão tentando usar o nome gibMacOS para distribuir malware"*. Baixe **só** dessa URL, nunca de re-upload.
  - **Como:** rode `gibMacOS.bat` → navegue até **macOS 12 (Monterey)** no catálogo de versões **públicas** (não use beta) → baixe.
  - **Resultado esperado:** `C:\gibMacOS\macOS Downloads\macOS 12\...\InstallAssistant.pkg` (**≈ 12 GB**).
  - ⚠️ **Não use o `MakeInstall.bat`** do gibMacOS: desde o Big Sur a Apple mudou a distribuição e ele **não consegue mais** criar pendrive do macOS 11+ a partir do Windows. É exatamente por isso que usamos o **UnPlugged**.

- **Ferramenta B — macrecovery:** `https://github.com/acidanthera/OpenCorePkg/releases` → baixe o **RELEASE zip** → extraia em `C:\OpenCore`. No CMD:
  ```
  cd C:\OpenCore\Utilities\macrecovery
  python macrecovery.py -b Mac-E43C1C25D4880AD6 -m 00000000000000000 download
  ```
  *(`Mac-E43C1C25D4880AD6` é o board ID que a Dortania lista para o Monterey.)*
  - **Resultado esperado:** pasta `com.apple.recovery.boot` com `BaseSystem.dmg` + `BaseSystem.chunklist`.
  - ℹ️ O recovery do Monterey **não** tem o bug do Sonoma+ (cujo recovery não monta FAT32/exFAT). Por isso este método funciona limpo no seu caso.

- **Ferramenta C — UnPlugged:** `https://github.com/corpnewt/UnPlugged` → pegue o arquivo **`UnPlugged.command`** da raiz do repositório.

### ⚠️ PASSO 21 — Baixar USBToolBox (se ainda não fez o Passo 18)
Já coberto no Passo 18. Só confirme que você tem `UTBMap.kext` e `USBToolBox.kext` antes de continuar.

### ⚠️ PASSO 22 — Preparar o pendrive 1 (8 GB, uma partição só)
*Por que só uma partição:* com 8 GB não há espaço para o instalador de 12 GB. O pendrive vira apenas um **recovery USB**.

⚠️ **Confira três vezes o número do disco. `clean` apaga tudo.**
- **Ferramenta:** `diskpart` (CMD como admin).
- **Como:** insira o **pendrive 1** *depois* de abrir o diskpart (para identificá-lo pelo tamanho, ~7,4 GB):
  ```
  diskpart
  list disk
  select disk N        <- N = PENDRIVE 1 (confira o tamanho!)
  clean
  convert mbr
  create partition primary
  format fs=fat32 quick label=OPENCORE
  assign letter=O
  exit
  ```
- **Copie para `O:`:**
  1. a pasta **`EFI`** inteira (gerada pelo OCS)
  2. a pasta **`com.apple.recovery.boot`** inteira
  3. o arquivo **`UnPlugged.command`**
  4. confirme que **`ocvalidate.exe`** está em `O:\EFI\OC\`
- ⚠️ **Ejete pelo ícone da bandeja antes de remover.** Gravação incompleta = instalador corrompido.
- **Resultado esperado:**
  ```
  O: → EFI\  +  com.apple.recovery.boot\  +  UnPlugged.command
  ```
  **~900 MB usados, ~6,5 GB livres.**
- **Conferência OBRIGATÓRIA (evita o erro "OpenCore só mostra Windows"):**
  ```
  O:\
  ├── EFI\OC\Drivers\  → OpenRuntime.efi + OpenHfsPlus.efi (ou HfsPlus.efi)
  ├── com.apple.recovery.boot\  → BaseSystem.dmg + BaseSystem.chunklist
  └── UnPlugged.command
  ```
  ⚠️ `com.apple.recovery.boot` fica na **raiz da partição, ao lado de `EFI`** — nunca dentro de `EFI`. Sem o `BaseSystem.dmg` aí, ou sem o driver HFS+ habilitado em `UEFI → Drivers` do config.plist, o OpenCore lista **apenas o Windows** e nenhuma entrada de macOS aparece.

### ⚠️ PASSO 23 — Colocar o instalador na partição `MACINST`
*Por que existe:* o `InstallAssistant.pkg` tem ~12 GB e **não cabe em nenhum dos seus pendrives de 8 GB**. A partição exFAT do HDD é o lugar dele — e o recovery do Monterey consegue lê-la.
- **Ferramenta:** Explorador de Arquivos.
- **Como:**
  1. Copie `InstallAssistant.pkg` de `C:\gibMacOS\macOS Downloads\macOS 12\...` para **`MACINST (E:)`**.
  2. Copie também o **`UnPlugged.command`** para `MACINST` (por garantia).
  3. Remova com segurança.
- **Resultado esperado:** `MACINST` contém `InstallAssistant.pkg` (~12 GB) + `UnPlugged.command`.
- ℹ️ **A `MACINST` NUNCA aparece no picker do OpenCore — e isso é o correto.** Ela é exFAT (volume de dados); o OpenCore não tem driver exFAT e o picker só lista entradas bootáveis. A entrada de instalação é a `macOS Base System` (recovery de ~800 MB no pendrive 1, Passo 22).

---

<a id="seção-6"></a>
# SEÇÃO 6 — INSTALAÇÃO DO macOS

### PASSO 24 — Dar boot pelo OpenCore
- **Ferramenta:** tecla **F12** (menu de boot único da Dell).
- **Como:**
  1. Ligue/reinicie pressionando **F12** repetidamente.
  2. Selecione o **pendrive 1 em modo UEFI** (`UEFI: USB Flash Drive`).
     - ⚠️ Use uma **porta USB 2.0 traseira (preta)**. Nunca a frontal nem hub.
  3. (Opcional, recomendado na primeira vez) Selecione **OpenShell** e rode o `OCValidate.exe config.plist` do Passo 19.5.
- **Resultado esperado:** menu do OpenCore com as entradas **`macOS Base System`** e **`Windows Boot Manager`**.
- **Erros possíveis nesta etapa:**
  - `OCB: LoadImage failed - Security Violation` → o Secure Boot ainda está ligado (Passo 8, item 9).
  - `OCABC: Memory pool allocation failure` → BIOS: procure "Windows 8.1/10 UEFI Mode" e mude para **Other OS**.
  - Tela preta logo após o logo Dell → a **porta serial** ainda está habilitada (Passo 8, item 4).
- ⚠️ **Não mude a ordem de boot permanente ainda.** Use F12 até tudo estar estável.

### ⚠️ PASSO 25 — Formatar SOMENTE a partição `MACINST`
- **Ferramenta:** Utilitário de Disco do recovery.
- **Como:**
  1. Menu do OpenCore → **`macOS Base System`** → Enter.
  2. Abra **Utilitário de Disco**.
  3. Menu **Visualizar → Mostrar Todos os Dispositivos**.
  4. ⚠️ **PARE E CONFIRA.** O HDD de 500 GB deve listar a partição **Windows (NTFS)**, a **EFI** e a **Recuperação**. **Se você não enxergar o Windows, ABORTE** — você está no disco errado ou na visualização errada.
  5. Selecione **o volume `MACINST` (exFAT)** — **NÃO o disco pai**, **NÃO** o `Windows`, **NÃO** o `EFI`.
  6. **Apagar** → Nome: **`Macintosh HD`** · Formato: **APFS** · Esquema: **Mapa de Partição GUID** → Apagar.
  7. Feche o Utilitário de Disco.
- **Resultado esperado:** volume `Macintosh HD` (APFS) na lista **e o `Windows` ainda listado e intacto**.
- ℹ️ Apagar o `MACINST` apaga junto o `InstallAssistant.pkg` que estava nele. **Não tem problema** — mas, se quiser garantia extra, faça o Passo 26 **primeiro** e só apague quando o UnPlugged pedir o volume de destino.

### PASSO 26 — Instalar via UnPlugged
- **Ferramenta:** Terminal do recovery + `UnPlugged.command`.
- **Como:**
  1. Abra **Terminal** (menu Utilitários → Terminal).
  2. Confirme que a partição exFAT está montada:
     ```
     ls /Volumes
     ```
     Deve aparecer `MACINST`. ⚠️ **Se não aparecer, não continue** — volte ao Passo 11, item 4 (unidade de alocação exFAT).
  3. Execute:
     ```
     cd /Volumes/MACINST
     ./UnPlugged.command
     ```
     (se falhar: `bash UnPlugged.command`)
  4. Origem do instalador: prefira **`Fully expand InstallAssistant.pkg`** (mais lento, zero risco de incompatibilidade de app).
  5. Volume de destino: **`Macintosh HD`**.
  6. Confirme com `y`.
  7. Quando o instalador gráfico abrir, **você precisa clicar até o fim**: **Continuar** → **Concordar** → na tela de disco, selecione **`Macintosh HD`** → **Continuar/Instalar** → aguarde a barra de cópia e o **reinício automático**.
- ⚠️ **NÃO FECHE o Terminal** — fechar encerra o instalador.
- ⚠️ **O reinício automático é o sinal de que a 1ª fase foi agendada.** Se o instalador foi fechado antes disso, nada é gravado no HDD e o OpenCore **nunca** mostrará `macOS Installer` — rode o UnPlugged de novo e complete os cliques.
- **Resultado esperado:** o instalador copia a 1ª fase e o PC reinicia sozinho.

### PASSO 27 — Durante a instalação: o que é normal
- ⚠️ **O HDD NÃO vai oferecer o macOS sozinho durante a instalação — e isso é normal.** O OpenCore ainda mora só no pendrive; ele só é copiado para o HDD no Passo 28. Instalar numa partição do HDD é o procedimento padrão; o "sumiço" do macOS a cada reinício não é defeito da partição.
- **A cada reinício:** `F12` → **pendrive 1 (UEFI)** → no OpenCore selecione **`macOS Installer`** (entrada nova que aparece depois da 1ª fase; se não ver, aperte `Espaço`). `macOS Base System` é só o recovery do pendrive. Se não houver `macOS Installer`: (a) confira que a 1ª fase completou — o instalador tem que ter reiniciado **sozinho** após os cliques do Passo 26, item 7; (b) confira `ScanPolicy = 0` no `config.plist` (Passo 19.3). Não é a partição: instalar num container APFS dentro de partição de disco GPT compartilhado é o fluxo padrão. Se o PC cair no Windows no meio do caminho, **nada estragou** — reinicie pelo pendrive.
- 💡 **Para não perder nenhum reinício:** durante a instalação, ponha o pendrive no **topo do boot** (F2 → General → Boot Sequence). Depois do Passo 29, volte e ponha o HDD em primeiro.
- **Tempo esperado no HDD de 500 GB: 1h30 a 3h.** (Em SSD seria ~30 min.)
- **3 a 6 reinicializações.** A barra de progresso **recomeça várias vezes** e pode ficar 10–20 min parada na mesma porcentagem. **Isso é normal em HDD.**
- A cada reinício: **F12 → pendrive 1 (UEFI)** → selecione **`macOS Installer`** (depois `Macintosh HD`) no menu do OpenCore.
- ⚠️ **Nunca desligue na força durante a cópia.** Se travar de verdade (>40 min sem nenhuma mudança na tela), reinicie e repita o boot — o instalador retoma de onde parou.
- **Resultado esperado:** tela de configuração inicial do macOS. **Crie a conta offline** (sem Apple ID) — o login no iCloud fica para o Passo 34.

---

<a id="seção-7"></a>
# SEÇÃO 7 — PÓS-INSTALAÇÃO (macOS como sistema único)

### ⚠️ PASSO 28 — Mover o OpenCore para o HDD
- **Ferramenta:** **MountEFI** (`https://github.com/corpnewt/MountEFI`) ou **Hackintool**.
- **Como:**
  1. Instale o MountEFI no macOS recém-instalado.
  2. Monte a partição **EFI do disco interno** e a do **pendrive 1**.
  3. Copie a pasta `EFI` do pendrive para a **EFI do disco interno** (substitua se existir).
  4. Ejete as duas.
- **Resultado esperado:** reiniciando **sem** o pendrive (F12 → HDD interno), o OpenCore aparece e o macOS inicia.

### PASSO 29 — Tornar o macOS o boot padrão
- **Ferramenta:** BIOS (**F2**).
- **Como:**
  1. **General → Boot Sequence** → coloque o **HDD interno / `macOS`** em **primeiro lugar**.
  2. Como você configurou `LauncherOption = Full` (Passo 16), o OpenCore também se registra como entrada de boot na NVRAM.
  3. **Apply → Exit**, reinicie sem pendrive.
- **Resultado esperado:** o PC liga direto no macOS.
- ⚠️ **O pendrive 1 vira seu pendrive de emergência. Não o formate.**
- ⚠️ **Não apague o Windows ainda.** Só no Passo 30, e apenas depois de **3 boots limpos** com áudio, rede e vídeo funcionando.

### ⚠️ PASSO 30 — Apagar o Windows e entregar o disco inteiro ao macOS
*Irreversível. Só execute com o backup do Passo 2 validado.*
- **Ferramenta:** Utilitário de Disco do macOS.
- **Como:**
  1. Abra **Utilitário de Disco** → **Visualizar → Mostrar Todos os Dispositivos**.
  2. Selecione a partição **`Windows` (NTFS)** → **Apagar**.
  3. Selecione o volume **`Macintosh HD`** → botão **Partição** → clique no espaço livre deixado pelo Windows → **`–` (remover)** → **Aplicar**.
  4. Aguarde (pode levar vários minutos em HDD).
- **Resultado esperado:** o disco inteiro vira um único volume APFS `Macintosh HD` (~460 GB).
- ⚠️ Se o botão de expansão ficar **cinza**, o espaço livre não é contíguo. **Não force.** 150 GB de APFS já funcionam — ou refaça com um apagamento completo do disco numa manutenção futura.
- ⚠️ **Nunca apague a partição EFI.** É onde mora o OpenCore. Sem ela o PC não liga.

### PASSO 31 — Verificar e finalizar o mapeamento USB
- **Ferramenta:** **Hackintool** → aba **USB**, ou USBToolBox versão macOS.
- **Como:**
  1. Se você já fez o Passo 18, só **teste**: espete um pendrive em cada porta e confirme que todas aparecem em Hackintool → USB.
  2. Se não fez: `https://github.com/USBToolBox/tool/releases` (versão macOS) → descubra as portas → gere `UTBMap.kext` → coloque em `EFI/OC/Kexts/` + `Kernel → Add`, com `USBToolBox.kext` antes. `XhciPortLimit = false`.
  3. ⚠️ Ao colocar o `UTBMap.kext`, **remova o `UTBDefault.kext`** (a pasta em `EFI/OC/Kexts/` **e** a entrada em `Kernel → Add`). Os dois mapas juntos conflitam — o `UTBDefault` era só o "quebra-galho" da instalação.
- **Resultado esperado:** todas as portas do gabinete funcionando, inclusive após suspensão.

### PASSO 32 — Áudio (ALC3234) — a ordem correta de teste
- **Ferramenta:** Ajustes de Sistema → Som; se precisar, editor de plist + MountEFI.
- **Como, em ordem:**
  1. **Teste o padrão:** Ajustes → Som → há um dispositivo de saída? Toque um som. O `alcid=11` já deve estar ativo (Passo 16).
  2. **Fone frontal funciona mas o line-out traseiro não?** Troque para **`alcid=66`** (*"Dell Optiplex 7060/7070MT — Separate LineOut"*).
  3. **Nada funciona?** Troque para **`alcid=12`** (*"Dell Optiplex 7040 MT"*), depois **`alcid=3`** (*Mirone*).
     - Como trocar: monte a EFI com MountEFI → edite `NVRAM → 7C436110-... → boot-args` → reinicie.
  4. **Nenhum dispositivo aparece de jeito nenhum?** O `FixHPET` não foi aplicado. Rode o **SSDTTime** (`https://github.com/corpnewt/SSDTTime`), gere `SSDT-HPET`, coloque em `EFI/OC/ACPI/` e adicione em `ACPI → Add`.
- **Resultado esperado:** som saindo pela porta que você usa.
- ℹ️ O `alcid` é só um boot-arg: testar outro não quebra nada e é revertido trocando de volta.

### PASSO 33 — Rede Ethernet (RTL8111HSD)
- **Como:** conecte o cabo no RJ-45 → Ajustes de Sistema → Rede → deve aparecer uma interface com endereço IP.
- **Resultado esperado:** internet funcionando. **Sem isso não há App Store, iCloud nem atualizações.**
- ⚠️ Se não aparecer interface nenhuma, o `RealtekRTL8111.kext` não está em `Kernel → Add`, ou o item **Integrated NIC** está desativado na BIOS (Passo 8, item 5).

### PASSO 34 — Aceleração de vídeo (HD 530)
- **Ferramenta:** **Hackintool** → aba **System**; "Sobre Este Mac".
- **Como:**
  1. Confirme **Intel HD Graphics 530 · 1536 MB de VRAM**.
  2. Teste na prática: arraste janelas com transparência, abra o Mapas, role uma página pesada, assista um vídeo no Safari.
  3. Teste **HDMI** e **DisplayPort** separadamente (o 3040 tem os dois).
- **Resultado esperado:** QE/CI ativo, resolução nativa, "Milhões de cores".
- ⚠️ **VRAM de 7 MB** = framebuffer errado → confira `AAPL,ig-platform-id = 00001219` (Passo 19.3).
- ⚠️ **Tela preta em monitor 4K** = DVMT de 32 MB (oculto na BIOS Dell). Solução: adicionar `framebuffer-stolenmem` = `00000002` (32 MB) ou `00000004` (64 MB) em `DeviceProperties → PciRoot(0x0)/Pci(0x2,0x0)`, ou usar um monitor 1080p.

### PASSO 35 — Sleep (problema conhecido do Skylake)
- **Ferramenta:** Terminal.
- **Como:**
  ```
  sudo pmset -a hibernatemode 0
  sudo pmset -a standby 0
  sudo pmset -a womp 0
  sudo pmset -a disksleep 0
  ```
- Mantenha **Block Sleep: Enabled** na BIOS por enquanto.
- **Resultado esperado:** o Mac não acorda sozinho nem congela após suspender. Só tente habilitar o sleep de verdade depois que todo o resto estiver funcionando — e só se você realmente precisar.

### PASSO 36 — SMBIOS e iServices
- **Como:**
  1. Só com áudio, rede e vídeo funcionando: Ajustes de Sistema → **Apple ID** → faça login.
  2. Se der erro de autenticação, gere um **novo serial** com o GenSMBIOS (Passo 19.4) e atualize o `config.plist`.
- **Resultado esperado:** iMessage, FaceTime e App Store logados.

### PASSO 37 — Ajustes de desempenho para HDD + 8 GB
- **Como:**
  1. Ajustes → Geral → **Itens de Login**: deixe o mínimo.
  2. **Não** ative o Time Machine apontando para o mesmo HDD.
  3. Desative a indexação de pastas grandes:
     ```
     sudo mdutil -i off /caminho/da/pasta
     ```
  4. Desative "Otimizar Armazenamento" e efeitos de transparência se o sistema parecer pesado.
- **Resultado esperado:** sistema utilizável. **Será visivelmente mais lento que num SSD** — é limitação física do HDD 5400 rpm, não configuração errada.
- 💡 Se algum dia puder, um **SSD SATA de 120 GB por ~R$ 100** transforma este PC. É o upgrade com melhor custo-benefício possível aqui.

### PASSO 38 — Kit de diagnóstico (guarde os links)
| Ferramenta | Para quê |
|---|---|
| **Hackintool** | GPU, áudio (layout), USB, PCIe, gerar patch |
| **IORegistryExplorer** (Apple) | Ver dispositivos carregados e suas propriedades |
| **`sudo dmesg`** no Terminal | Erros de kext, ACPI e audio |
| **OCValidate.exe** (OpenShell) | Validar o `config.plist` a cada mudança |
| **MountEFI** | Montar a partição EFI |
| **GenSMBIOS** | Gerar Serial / MLB / SmUUID |
| **SSDTTime** | Gerar SSDTs (HPET, PLUG, EC-USBX...) |

---

<a id="seção-8"></a>
# SEÇÃO 8 — TROUBLESHOOTING PREVENTIVO

## Os 10 erros mais comuns neste setup — e como evitar ANTES

| # | Erro | Sintoma | Como evitar |
|---|---|---|---|
| 1 | **`XhciPortLimit` ligado no Monterey** | Boot loop ou trava no logotipo da maçã | Mapear as portas **antes** de instalar (Passo 18) e deixar `XhciPortLimit = false`. **Causa nº 1 de boot loop no macOS 12+.** |
| 2 | **Pendrive do instalador em porta errada** | Instalador congela nos primeiros segundos | Porta **USB 2.0 traseira (preta)**. Nunca frontal, nunca hub. |
| 3 | **`MACINST` não monta no recovery** | `ls /Volumes` não mostra a partição; UnPlugged não acha o pacote | Unidade de alocação exFAT **≤ 1024 KB** (Passo 11, item 4). Recrie a partição se necessário. |
| 4 | **Tentar colocar o `InstallAssistant.pkg` no pendrive de 8 GB** | "Espaço insuficiente" | É esperado: ~12 GB contra ~6,5 GB livres. O pacote **fica na `MACINST`** (Passo 23). |
| 5 | **Aceitar a versão de macOS que o OCS sugere por padrão** | Tela preta, GPU sem aceleração, sistema pesado, boot aleatório | **Forçar Monterey 12** (Passo 15), SMBIOS **`iMac17,1`**, framebuffer **`00001219`**, sem `device-id` KBL, sem `-radvesa`. |
| 6 | **Sem áudio** | Nenhum dispositivo de saída | Ordem: `alcid=11` → `66` → `12` → `3`. Se nada aparecer, falta o `SSDT-HPET` (Passo 32). |
| 7 | **Porta serial habilitada** | Tela preta depois do logo Dell | **System Configuration → Serial Port → Disabled** (Passo 8, item 4). |
| 8 | **Secure Boot / Legacy Option ROMs ligados** | `Security Violation` ou falha na inicialização da GPU | Passo 8, itens 1, 2 e 9. |
| 9 | **Apagar o Windows cedo demais** | macOS não dá boot e você fica sem sistema | Só execute o Passo 30 depois de **3 boots limpos**. Até lá o Windows é o plano B (F12 → `Windows Boot Manager`) e o **pendrive 2** é o plano C. |
| 10 | **Interromper a instalação por impaciência** | Sistema corrompido, precisa recomeçar | Em HDD é normal ficar 10–20 min na mesma porcentagem. **Só considere travado depois de 40 min sem nenhuma mudança na tela.** |
| 11 | **OpenCore lista só `Windows` (sem `macOS Base System`)** | Instalador nunca entra; pendrive que "macOS não existe" | `com.apple.recovery.boot` (com `BaseSystem.dmg`) na **raiz do pendrive, ao lado de `EFI`**; driver HFS+ (`OpenHfsPlus.efi`/`HfsPlus.efi`) presente em `EFI/OC/Drivers` **e** habilitado em `UEFI → Drivers`; `ScanPolicy = 0` (Passo 22, Conferência). |
| 12 | **Esperar que a partição `MACINST`/exFAT apareça no picker** | Perda de tempo procurando entrada que não existe | Volume de dados **nunca é entrada de boot**: o OpenCore não tem driver exFAT e o picker só lista APFS/HFS+/ESP/`com.apple.recovery.boot`. O instalador entra via `macOS Base System`; o pkg é lido **depois**, pelo recovery (macOS lê exFAT nativamente). |
| 13 | **EFI duplicada: OpenCore instalado também na ESP interna** | Edita o pendrive e "nada adianta" — o menu que roda é o do HDD | Teste: remover o pendrive e ligar. Se o picker aparecer sem pendrive, o boot está vindo do HDD. No F12, escolha a entrada com o **nome do pendrive** ("UEFI: <marca>"), nunca o "OpenCore"/"Windows Boot Manager" do disco. |
| 14 | **Porta USB 3.0 com falha de leitura no UEFI** | `BaseSystem.dmg` não monta; entrada some sem erro | Bootar o pendrive numa porta **USB 2.0** (pretas/frontais do 3040). |
| 15 | **Cópia corrompida do `BaseSystem.dmg`** | Tamanho certo, conteúdo truncado; HFS+ não monta, entrada não lista | Comparar SHA256 origem × pendrive com `certutil -hashfile … SHA256`; recopiar pela USB 2.0 se diferir. |

## Plano de contingência (em ordem)

| Situação | O que fazer |
|---|---|
| O macOS não dá boot, mas o pendrive 1 funciona | F12 → pendrive 1 → selecione `Macintosh HD`. Depois investigue com `-v keepsyms=1`. |
| O OpenCore da EFI interna corrompeu | F12 → pendrive 1 → macOS inicia. Recopie a pasta `EFI` (Passo 28). |
| Quer voltar para o Windows (antes do Passo 30) | F12 → **`Windows Boot Manager`**. |
| Apagou o Windows e precisa dele de volta | **Pendrive 2** (Passo 6) → reinstalação limpa do Windows 10. |
| Precisa do PC exatamente como estava antes de tudo | **Imagem de disco do Passo 2** + USB de recuperação do Macrium/AOMEI. |

---

<a id="apêndice"></a>
# APÊNDICE

## A — Valores esperados no `config.plist` (OptiPlex 3040 + Monterey)

| Seção | Chave | Valor esperado |
|---|---|---|
| `DeviceProperties` | `PciRoot(0x0)/Pci(0x2,0x0) → AAPL,ig-platform-id` | `00001219` |
| `Kernel → Quirks` | `AppleXcpmCfgLock` | `true` |
| `Kernel → Quirks` | `XhciPortLimit` | `false` |
| `Kernel → Quirks` | `DisableIoMapper` | `true` |
| `PlatformInfo → Generic` | `SystemProductName` | `iMac17,1` |
| `Misc → Boot` | `LauncherOption` | `Full` |
| `NVRAM → boot-args` | | `-v keepsyms=1 alcid=11` |
| `ACPI → Add` | SSDTs | `SSDT-EC-USBX`, `SSDT-PLUG`, `SSDT-XOSI`, `SSDT-HPET` |
| `Kernel → Add` | kexts | `Lilu`, `VirtualSMC`, `WhateverGreen`, `AppleALC`, `RealtekRTL8111`, `USBToolBox`, `UTBMap` |

## B — Checklist de validação final (marque antes de apagar o Windows)

- [ ] OCValidate retorna `No issues found.`
- [ ] macOS inicia sem o pendrive
- [ ] "Sobre Este Mac" mostra **Intel Core i5** e **Intel HD Graphics 530 · 1536 MB**
- [ ] Ethernet funciona com IP válido
- [ ] Áudio toca som na porta que você usa
- [ ] **Todas** as portas USB do gabinete funcionam
- [ ] HDMI **e** DisplayPort testados
- [ ] 3 reinicializações limpas consecutivas
- [ ] Backup do Passo 2 testado e acessível
- [ ] Pendrive 1 (emergência) e pendrive 2 (Windows) guardados

## C — Links oficiais (só estes)

| Item | URL |
|---|---|
| **OpCore Simplify** (oficial) | `https://github.com/lzhoang2801/OpCore-Simplify` |
| gibMacOS | `https://github.com/corpnewt/gibMacOS` |
| UnPlugged | `https://github.com/corpnewt/UnPlugged` |
| OpenCorePkg (macrecovery + OCValidate) | `https://github.com/acidanthera/OpenCorePkg/releases` |
| AppleALC | `https://github.com/acidanthera/AppleALC` |
| USBToolBox (ferramenta / kext) | `https://github.com/USBToolBox/tool/releases` · `https://github.com/USBToolBox/kext/releases` |
| GenSMBIOS | `https://github.com/corpnewt/GenSMBIOS` |
| MountEFI | `https://github.com/corpnewt/MountEFI` |
| SSDTTime | `https://github.com/corpnewt/SSDTTime` |
| Hackintool | `https://github.com/headkaze/Hackintool/releases` |
| Manual do OptiPlex 3040 (Dell) | `https://www.dell.com/support/manuals/pt-br/optiplex-3040m-desktop` |
| BIOS do OptiPlex 3040 | `https://www.dell.com/support/home/pt-br` → busque "OptiPlex 3040" |
| Windows 10 (pendrive 2) | `https://www.microsoft.com/pt-br/software-download/windows10` |
| Guia Dortania (referência) | `https://dortania.github.io/OpenCore-Install-Guide/` |
| Skylake Desktop (Dortania) | `https://dortania.github.io/OpenCore-Install-Guide/config.plist/skylake.html` |

## D — Estimativa de tempo

| Fase | Tempo |
|---|---|
| Backup + imagem de disco + pendrive 2 | 1–3 h |
| BIOS (padrões + ajustes + atualização opcional) | 30–45 min |
| Particionamento | 15 min |
| OCS (EFI) + mapeamento USB + validação | 45 min |
| Downloads (gibMacOS ~12 GB) | 30–90 min |
| Pendrive 1 + instalação | 2–4 h (HDD) |
| Pós-instalação + apagar o Windows | 1 h |
| **Total** | **~1 a 2 dias de trabalho intercalado** |
