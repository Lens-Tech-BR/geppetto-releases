# Geppetto, antivírus e EDR

Guia para usuários e para o time de TI/SOC. Explica por que um antivírus ou EDR pode apontar o Geppetto como suspeito e o que fazer.

## Índice

1. [Resumo](#resumo)
2. [O que é o Geppetto](#o-que-é-o-geppetto)
3. [Por que pode ser apontado como suspeito](#por-que-pode-ser-apontado-como-suspeito)
4. [O que o Geppetto não faz](#o-que-o-geppetto-não-faz)
5. [Como conferir que o instalador é autêntico](#como-conferir-que-o-instalador-é-autêntico)
6. [Windows e macOS: avisos do sistema](#windows-e-macos-avisos-do-sistema)
7. [Como pedir liberação ao TI](#como-pedir-liberação-ao-ti)
8. [Como reportar falso positivo](#como-reportar-falso-positivo)
9. [Contato](#contato)

## Resumo

- O Geppetto é um aplicativo de janela que abre vários terminais e mostra a saída de assistentes de IA de linha de comando. Esse padrão de uso lembra o de algumas ameaças, por isso pode gerar alertas.
- Os instaladores atuais ainda não têm assinatura de código, o que reduz a reputação junto ao SmartScreen, ao Gatekeeper e a antivírus. A assinatura está planejada.
- O Geppetto não abre portas para a rede externa, não grava no registro do Windows, não cria inicialização automática, serviços ou tarefas agendadas e não injeta código em outros processos.
- Confira sempre o SHA-256 do instalador e baixe apenas dos links oficiais.
- Se houver alerta, peça ao TI uma exceção baseada na regra que disparou, seguindo a ordem de preferência descrita neste guia.

## O que é o Geppetto

O Geppetto é um orquestrador de assistentes de IA de linha de comando, como Claude Code, Codex, Gemini/Antigravity e outros, instalados pelo próprio usuário. Ele é uma janela nativa que abre vários terminais, um por agente, e mostra a saída deles dentro da própria janela.

## Por que pode ser apontado como suspeito

| Comportamento | Por que parece suspeito | Por que é legítimo |
|---|---|---|
| Um aplicativo de janela cria muitos processos filhos (terminais, os CLIs de IA, `node`, `git`, shells) e usa pseudo-consoles do Windows (ConPTY), sem janelas de console visíveis | Um programa gráfico que dispara shells e interpretadores em segundo plano lembra um dropper ou um executor remoto de comandos | Esse é o funcionamento normal de um orquestrador de terminais. O ConPTY é o mesmo mecanismo do Windows Terminal e do VS Code. Não há janelas de console porque a saída aparece na interface do app |
| Execução frequente e curta de um pequeno executável auxiliar do próprio Geppetto | Execuções repetidas e rápidas de um binário pouco conhecido lembram beaconing ou ferramentas de reconhecimento | O auxiliar apenas alimenta a interface com informações de status. Ele faz parte do aplicativo instalado |
| Com o "modo automático" ligado (desligado por padrão), o Geppetto repassa aos CLIs de terceiros as opções deles que pulam confirmações, algumas com "dangerously" ou "bypass" no nome | Linhas de comando com essas palavras ativam regras de detecção de evasão ou de execução sem controle | Essas opções pertencem aos CLIs de terceiros, não ao Geppetto, e só são usadas quando o usuário liga o modo automático. Uma política corporativa pode simplesmente manter o modo automático desligado |
| Leitura (somente leitura) das variáveis de ambiente do usuário no registro do Windows | Consultas ao registro são comuns em malware que faz reconhecimento do sistema | A leitura serve só para achar o PATH atualizado dos CLIs instalados. Nenhuma escrita é feita no registro |
| Gravação de configurações por sessão para os CLIs de IA na pasta de dados do próprio Geppetto, no perfil do usuário, e leitura somente-leitura do histórico local de uso de um dos CLIs | Ler dados de outro programa e gravar arquivos de configuração no perfil lembra coleta de dados | As configurações ficam na pasta do próprio Geppetto. A leitura do histórico serve apenas para exibir o consumo de contexto na tela |
| A janela usa o WebView2 da Microsoft hospedado num processo Python empacotado, com .NET carregado dentro do app | Algumas heurísticas associam essa combinação a carregadores de código | O WebView2 é o componente oficial da Microsoft para exibir interfaces web em aplicativos nativos. O empacotamento é apenas a forma como o app é distribuído |
| Comunicação interna local (loopback, 127.0.0.1) entre a janela e os componentes do app | Tráfego local entre processos pode ser confundido com um servidor ou um canal de comando | Essa comunicação nunca sai da máquina. Nada escuta conexões vindas de fora |
| Instalador sem assinatura de código | Binários sem assinatura têm reputação baixa e são tratados com mais desconfiança | É uma limitação temporária. A assinatura com o certificado do editor `lenstech.com.br` está planejada |

## O que o Geppetto não faz

- Não abre porta para a rede externa.
- Não grava no registro do Windows.
- Não cria inicialização automática, serviços nem tarefas agendadas.
- Não injeta código em outros processos.
- Não desativa antivírus.
- Não usa credenciais de outros aplicativos.

As únicas conexões externas do Geppetto são a checagem de licença e de atualização com a API da Lens Tech (`api.lenstech.com.br`) e, se ela não responder, a página pública de releases no GitHub. As chamadas que os próprios CLIs de IA fazem aos seus fornecedores são feitas por eles, não pelo Geppetto.

## Como conferir que o instalador é autêntico

1. Baixe o instalador apenas de uma destas fontes oficiais:
   - https://github.com/Lens-Tech-BR/geppetto-releases/releases
   - https://api.lenstech.com.br/download
2. Cada release publica `SHA256SUMS.txt` (Windows) e `SHA256SUMS-macos.txt` (macOS). Compare o hash calculado com o publicado.

Windows (PowerShell):

```powershell
Get-FileHash .\Geppetto-Setup-X.Y.Z.exe -Algorithm SHA256
```

macOS (Terminal):

```bash
shasum -a 256 Geppetto-X.Y.Z-macos-arm64.dmg
```

Se o hash for diferente do publicado, não instale e fale com o suporte.

## Windows e macOS: avisos do sistema

### Windows: SmartScreen e quarentena

- O instalador é instalado por usuário, em `%LOCALAPPDATA%\Programs\Geppetto`.
- Como o instalador ainda não tem assinatura (Authenticode), o SmartScreen pode exibir "O Windows protegeu o computador". Depois de conferir o SHA-256, clique em "Mais informações" e em "Executar assim mesmo".
- Se o antivírus ou o EDR mover arquivos para a quarentena, restaure-os e peça ao TI a exceção, conforme a seção abaixo.

### macOS: Gatekeeper

- Arraste o Geppetto para a pasta Applications.
- O aplicativo tem assinatura ad-hoc e não passou pela notarização da Apple, então o Gatekeeper pode bloquear a primeira abertura.
- Para liberar, use uma das opções:
  - No Terminal: `xattr -dr com.apple.quarantine /Applications/Geppetto.app`
  - Em Ajustes do Sistema, Privacidade e Segurança, clique em "Abrir Mesmo Assim".

## Como pedir liberação ao TI

Ordem de preferência para a exceção:

1. **Por editor/certificado**, quando os binários estiverem assinados. É a opção que sobrevive a atualizações.
2. **Por caminho**, somente se a instalação estiver numa pasta protegida (por exemplo, Program Files). Não recomendamos liberar por caminho numa pasta do perfil do usuário, porque qualquer programa do usuário poderia gravar lá e herdar a exceção.
3. **Por hash SHA-256**, como paliativo. Precisa ser refeito a cada nova versão.

Alertas comportamentais não desaparecem com a assinatura. Nesses casos, o analista precisa ver na console qual regra disparou (nome da detecção, processo pai e linha de comando) e criar a exceção para aquela regra.

Modelo de e-mail:

```text
Assunto: Pedido de liberação do Geppetto no antivírus/EDR

Olá, time de TI/SOC,

Preciso usar o Geppetto, aplicativo desktop da Lens Tech que orquestra
assistentes de IA de linha de comando. Ele foi apontado como suspeito
pelo nosso antivírus/EDR. Peço a avaliação do alerta e, se for
considerado falso positivo, uma exceção.

- Versão do Geppetto: <versão>
- SHA-256 do instalador: <sha256>
- Nome da detecção/regra: <nome da detecção>
- Processo pai: <processo pai>
- Linha de comando: <linha de comando>
- Equipamento/usuário: <equipamento e usuário>

Guia público com o comportamento do aplicativo e o que ele não faz:
https://github.com/Lens-Tech-BR/geppetto-releases/blob/main/docs/ANTIVIRUS-E-EDR.md

Os hashes oficiais estão em
https://github.com/Lens-Tech-BR/geppetto-releases/releases

Obrigado.
<seu nome>
```

## Como reportar falso positivo

Para antivírus de consumo, envie o arquivo ou o hash ao fabricante. Para EDR corporativo (CrowdStrike, SentinelOne, Defender for Endpoint, Palo Alto), o caminho é o TI/SOC da empresa, pela console.

| Fabricante | Onde reportar |
|---|---|
| Microsoft Defender (WDSI) | https://www.microsoft.com/en-us/wdsi/filesubmission |
| VirusTotal | https://www.virustotal.com (consulte pelo hash, sem enviar o arquivo, se não quiser publicá-lo) |
| Avast / AVG | https://www.avast.com/report-false-positive |
| Bitdefender | https://www.bitdefender.com/submit/ |
| ESET | https://support.eset.com/en/kb141 |
| Kaspersky | https://opentip.kaspersky.com/ |
| Sophos | https://support.sophos.com/support/s/filesubmission |
| Trend Micro | https://www.trendmicro.com/en_us/about/legal/detection-reevaluation.html |
| Malwarebytes | https://forums.malwarebytes.com/forum/122-false-positives/ |

## Contato

Fale com o suporte da Lens Tech pelo site https://lenstech.com.br.
