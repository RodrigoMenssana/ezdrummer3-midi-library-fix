# Guia técnico — Correção do erro “No MIDI library found” no EZdrummer 3 (Windows)

**Status do caso documentado:** resolvido e validado  
**Data do atendimento:** 12/11/2026  
**Aplicações:** Toontrack EZdrummer 3 (standalone e VST3), Steinberg Cubase 15  
**Sistema:** Windows; comandos executados em PowerShell 7  
**Escopo:** diagnóstico e correção de bibliotecas de grooves MIDI não reconhecidas.

> **Atenção:** este guia inclui uma alteração pontual no Registro do Windows. Execute-a somente quando o diagnóstico indicar a existência do valor `MidiPath` no caminho documentado. Antes da exclusão, faça backup e confirme que ele foi criado. O procedimento não é uma solução universal para todo erro de biblioteca MIDI.

## Índice

- [1. Sintoma](#1-sintoma)
- [2. Diagnóstico da causa neste caso](#2-causa-constatada-neste-caso)
- [3. Diagnóstico antes de qualquer alteração](#3-diagnóstico-antes-de-alterar-qualquer-configuração)
- [4. Backup do Registro](#4-backup-obrigatório-do-registro)
- [5. Correção aplicada](#5-correção-aplicada--remover-somente-o-valor-legado-midipath)
- [6. Validação](#6-validação-após-a-correção)
- [7. Resultados](#7-resultado-e-critérios-de-encerramento)
- [8. Cuidados e alternativas](#8-cuidados-e-alternativas-para-outros-usuários)
- [9. Referências oficiais](#9-referências-oficiais)

## 1. Sintoma

Na aba **Grooves**, o EZdrummer 3 exibia **“No MIDI library found”**. As bibliotecas de sons e as pastas dos grooves MIDI existiam no disco, mas o programa não apresentava os grooves no navegador. Na aba **Settings > Advanced**, a opção **Restore MIDI Database** estava desabilitada.

## 2. Causa constatada neste caso

Foi encontrado um valor legado chamado `MidiPath` nesta chave do Registro:

```text
HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Toontrack\Superior\EZdrummer
```

O valor apontava para:

```text
C:\Program Files (x86)\Common Files\Toontrack\EZDrummer\MIDI
```

Entretanto, os arquivos MIDI estavam presentes sob o caminho personalizado configurado no programa:

```text
C:\Users\SoaresRo\Documents\Library\EZDrummer\Midi
```

Depois da exclusão **somente do valor `MidiPath`**, o EZdrummer 3 passou a reconhecer e reproduzir os grooves em modo standalone e como plugin VST3 no Cubase 15. A sequência de testes sustenta fortemente que a entrada legada provocava o problema **nesta instalação**.

> Os caminhos exibidos foram anonimizados com `<USUARIO>`. São exemplos do atendimento, não caminhos universais. Consulte o diretório efetivamente configurado em **Settings > Libraries/Paths**.

## 3. Diagnóstico antes de alterar qualquer configuração

### 3.1. Conferir o caminho das bibliotecas

1. Abra o EZdrummer 3.
2. Acesse **Settings > Libraries/Paths**.
3. Registre o diretório indicado em **Sound and MIDI Libraries**.
4. Não altere o caminho ainda.

**No caso documentado:** `C:\Users\SoaresRo\Documents\Library\EZDrummer`.

O caminho padrão frequentemente usado no Windows é:

```text
C:\Program Files (x86)\Common Files\Toontrack\EZDrummer
```

O uso de uma pasta personalizada, por si só, não caracteriza problema.

### 3.2. Conferir a existência física das bibliotecas

1. Abra o Explorador de Arquivos.
2. Acesse a pasta registrada em **Sound and MIDI Libraries**.
3. Verifique se há pastas de expansões EZX e a subpasta `Midi`.
4. Abra `Midi` e confirme se existem pastas dos pacotes MIDI, como a do EZdrummer 3.
5. Abra uma dessas pastas para verificar a presença de subpastas de grooves.

**No caso documentado:** foram encontradas pastas de expansões EZX e de diversos pacotes MIDI. A pasta principal do EZdrummer 3 tinha categorias de estilos musicais. Isso confirmou a existência da estrutura, embora não verificasse individualmente a integridade de todos os arquivos MIDI.

**Se não houver arquivos MIDI:** não passe diretamente à alteração do Registro. Verifique a instalação dos pacotes no **Toontrack Product Manager** e os caminhos de instalação.

### 3.3. Conferir a opção de restauração

Abra **Settings > Advanced** e localize **Restore MIDI Database**.

- Se estiver disponível, ela pode ser uma alternativa de diagnóstico/reconstrução, conforme a orientação da Toontrack. Registre a situação antes de executá-la.
- **No caso documentado:** o botão estava desabilitado; portanto, não foi executado.

### 3.4. Consultar a entrada `MidiPath` sem modificá-la

Abra o **PowerShell** e execute:

```powershell
reg.exe query "HKLM\SOFTWARE\WOW6432Node\Toontrack\Superior\EZdrummer" /v MidiPath
```

**Interpretação:**

- Se o valor `MidiPath` existir, anote seu conteúdo e compare com o local real das bibliotecas.
- Se a chave ou o valor não existir, **não execute a exclusão** descrita na seção 5; o diagnóstico precisa seguir outra linha.
- A simples diferença entre caminhos é um indício, não comprovação definitiva de causalidade.

**No caso documentado:** o valor existia e apontava para o caminho padrão antigo, diferente do diretório personalizado em uso.

## 4. Backup obrigatório do Registro

Antes de modificar o Registro, feche o EZdrummer 3 e o Cubase.

### 4.1. Exportar a chave no PowerShell

Execute:

```powershell
$backup = Join-Path $env:USERPROFILE 'EZdrummer_Registro_Backup.reg'
reg.exe export "HKLM\SOFTWARE\WOW6432Node\Toontrack\Superior\EZdrummer" "$backup" /y
```

**Resultado esperado:** `A operação foi concluída com êxito.`

Confirme que o backup existe e não está vazio:

```powershell
Test-Path -LiteralPath $backup -PathType Leaf
(Get-Item -LiteralPath $backup).Length
```

O primeiro resultado deve ser `True` e o segundo deve ser maior que zero. Se a exportação falhar ou o arquivo não puder ser verificado, **não avance para a exclusão**. Guarde o arquivo em um local seguro e persistente, preferencialmente também fora do computador. Por ser um backup de Registro, ele pode conter configurações da instalação e deve ser tratado como arquivo administrativo.

**Comando que efetivamente funcionou neste atendimento:**

```powershell
reg.exe export "HKLM\SOFTWARE\WOW6432Node\Toontrack\Superior\EZdrummer" "$env:TEMP\EZdrummer_Registro_Backup.reg" /y
```

Neste atendimento, o Windows confirmou o sucesso da exportação para a pasta temporária. **Não há registro de que o arquivo tenha sido posteriormente copiado para um local permanente.** Para repetir o procedimento com segurança, copie-o para um destino persistente e confira a cópia antes de prosseguir. Por exemplo:

```powershell
$origem = Join-Path $env:TEMP 'EZdrummer_Registro_Backup.reg'
$destino = Join-Path $env:USERPROFILE 'EZdrummer_Registro_Backup.reg'
Copy-Item -LiteralPath $origem -Destination $destino -ErrorAction Stop
Test-Path -LiteralPath $destino -PathType Leaf
(Get-Item -LiteralPath $destino).Length
```

Se qualquer etapa falhar, interrompa a correção até obter um backup válido.

### 4.2. Erro comum no PowerShell

As tentativas iniciais falharam ao usar a sintaxe do Prompt de Comando (`%USERPROFILE%` e `%TEMP%`) diretamente no PowerShell. Nesse terminal, use `$env:USERPROFILE` e `$env:TEMP`, respectivamente.

Exemplos corretos:

```powershell
$env:USERPROFILE
$env:TEMP
```

A falha ao gravar o `.reg` nessas tentativas não demonstrava corrupção do Registro; estava associada ao caminho de destino informado de forma inadequada para o terminal utilizado.

## 5. Correção aplicada — remover somente o valor legado `MidiPath`

**Pré-requisitos:**

- Bibliotecas e arquivos MIDI encontrados no disco.
- Valor `MidiPath` presente na chave especificada.
- Backup da chave exportado, com existência e tamanho verificados; cópia em local persistente recomendada.
- EZdrummer 3 e Cubase fechados.

Abra o **PowerShell como administrador** e execute:

```powershell
reg.exe delete "HKLM\SOFTWARE\WOW6432Node\Toontrack\Superior\EZdrummer" /v MidiPath /f
```

**Resultado esperado:** `A operação foi concluída com êxito.`

O comando remove **apenas o valor `MidiPath`**, não a chave inteira, e não exclui arquivos `.mid`, bibliotecas EZX, presets ou projetos do Cubase. **Não** substitua esse comando por outro que exclua a chave completa.

> Em um computador diferente, a remoção só deve ocorrer quando a entrada estiver presente e houver justificativa para seguir esta orientação da Toontrack. Se o comando falhar, não repita com caminhos ou parâmetros improvisados.

## 6. Validação após a correção

### 6.1. Teste em modo standalone

1. Abra o EZdrummer 3 diretamente no Windows, fora do Cubase.
2. Confirme que ele inicia sem erros.
3. Entre na aba **Grooves**.
4. Verifique se a mensagem **“No MIDI library found”** desapareceu.
5. Confirme que os pacotes e grooves MIDI são apresentados.

**Resultado obtido:** inicialização normal; mensagem eliminada; grooves disponíveis.

### 6.2. Teste no Cubase 15

1. Feche a instância standalone.
2. Abra o Cubase 15.
3. Crie um **projeto vazio temporário**, sem modificar produções existentes.
4. Insira o **EZdrummer 3 VST3** em uma Instrument Track.
5. Abra **Grooves**, confira as bibliotecas e reproduza um groove.

**Resultado obtido:** plugin iniciou sem erros; bibliotecas disponíveis; reprodução do groove normal.

## 7. Resultado e critérios de encerramento

| Verificação | Resultado do caso |
|---|---|
| Arquivos/pastas MIDI presentes | Confirmado |
| `Restore MIDI Database` disponível antes da correção | Não; botão desabilitado |
| `MidiPath` legado encontrado | Sim |
| Backup da chave do Registro | Exportação confirmada pelo Windows; verificação posterior de integridade e cópia permanente não registradas |
| Exclusão específica de `MidiPath` | Concluída com sucesso |
| EZdrummer 3 standalone | Inicialização normal |
| Navegador de grooves standalone | Bibliotecas reconhecidas |
| EZdrummer 3 VST3 no Cubase 15 | Inicialização normal |
| Reprodução de groove MIDI | Funcionando |
| Erro “No MIDI library found” | Resolvido |

**Parecer:** ocorrência encerrada após validação funcional em ambos os ambientes. A correção foi eficaz nesta instalação; não é garantida para outras causas do mesmo sintoma. Não foram necessárias reinstalações, movimentações de bibliotecas ou outras alterações.

## 8. Cuidados e alternativas para outros usuários

- **Não mova manualmente bibliotecas** apenas porque o EZdrummer mostra o erro. Primeiro examine o caminho configurado e os arquivos existentes.
- **Não exclua o Registro inteiro.** O procedimento oficial pertinente a este caso remove apenas o valor `MidiPath`.
- **Não importe um backup de outro computador.** O `.reg` exportado é específico daquela instalação.
- Se a exclusão de `MidiPath` não resolver, consulte o **Toontrack Product Manager**, verifique o estado de instalação das bibliotecas MIDI e procure o suporte da fabricante. Ela observa que alguns casos podem exigir reinstalação do conteúdo MIDI.
- **Restore MIDI Database** é um recurso distinto da correção do Registro; não foi utilizado neste atendimento, pois estava desabilitado.
- Se precisar desfazer a alteração, procure orientação técnica antes de importar o backup: importar um `.reg` pode sobrescrever configurações alteradas posteriormente. A restauração precisa considerar o estado atual do sistema.
- Após a solução, preserve os caminhos e as instalações que estão funcionando.

## 9. Referências oficiais

1. Toontrack — [Installation (tutorials & common issues)](https://www.toontrack.com/faq/installation-faq/). FAQ oficial que indica que entradas `MidiPath` não devem existir em instalações atuais e orienta o diagnóstico.
2. Toontrack — [HELP: No MIDI Library Found](https://www.toontrack.com/forums/topic/help-no-midi-library-found/). Discussão com recomendações para conferir o caminho das bibliotecas, restaurar o banco de dados e remover `MidiPath` em casos apropriados.
3. Toontrack — [PM doesn’t Install Ez Drummer MIDI file](https://www.toontrack.com/forums/topic/pm-doesnt-install-ez-drummer-midi-file/). Resposta da equipe da Toontrack com o comando específico de exclusão do valor `MidiPath`.
4. Toontrack — [EZdrummer 3 Release Notes](https://www.toontrack.com/release-notes/ezdrummer-3/). Notas de versão com referências à função **Restore MIDI Database**.

---

**Registro resumido da ocorrência:** erro de biblioteca MIDI → confirmação dos caminhos e arquivos → identificação de `MidiPath` divergente → backup do Registro → exclusão pontual do valor → testes standalone e VST3 aprovados → encerramento.
