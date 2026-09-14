# Como converter um save de Final Fantasy X do PS2 para Final Fantasy X HD Remaster no PS3

## Guia público, detalhado e seguro

**Última revisão:** 14 de setembro de 2026  
**Jogo:** Final Fantasy X  
**Origem:** PlayStation 2  
**Destino:** Final Fantasy X HD Remaster no PlayStation 3  
**Sistema usado no computador:** Windows

> Este procedimento modifica dados de save e não é um recurso oficial da Square Enix ou da Sony. Faça tudo por sua conta e risco. Nunca trabalhe sobre a única cópia do seu save. O método descrito aqui foi montado a partir de uma conversão real que terminou com o save reconhecido e carregado pelo jogo no PS3.

---

## 1. Resultado em uma frase

O fluxo que funcionou foi:

```text
Memory Card do PS2
        ↓
backup em .psu com wLaunchELF/uLaunchELF
        ↓
extração do arquivo bruto com PS2 Save Builder
        ↓
conversão pelo FFX HD Cross-platform Save Converter
        ↓
arquivo SAVES de PS3 ainda descriptografado
        ↓
substituição do SAVES dentro de um save-base real do próprio PS3
        ↓
criptografia e reconstrução de integridade no Bruteforce Save Data
        ↓
cópia pelo XMB e carregamento no FFX HD Remaster
```

A parte mais importante é esta: **não se cria um save de PS3 completo do zero**. É muito mais seguro criar um save-base legítimo no próprio PS3 e conservar sua estrutura, seu `PARAM.SFO`, seu `PARAM.PFD`, seus ícones e seus dados de proprietário. Dentro dessa estrutura, substitui-se somente o arquivo de jogo chamado `SAVES`.

---

## 2. O que foi confirmado no caso real

No caso que originou este guia:

- o arquivo bruto do PS2 era `BISLPS-25088FF090600`;
- o nome indica um save de **Final Fantasy X International**, associado ao ID japonês `SLPS-25088`;
- o save original tinha aproximadamente 99 horas e estava em Bevelle/Via Purifico;
- o destino usado no PS3 era a edição europeia do HD Remaster, com pasta iniciada por `BLES01880`;
- foi criado no PS3 um save-base curto, com cerca de 23 minutos;
- o conversor produziu um novo arquivo chamado `SAVES`;
- uma tentativa de finalizar tudo pelo Apollo Save Tool falhou com o código `8002920E`;
- a finalização pelo Bruteforce Save Data funcionou;
- na lista do jogo, o save convertido apareceu inicialmente com o tempo estranho `24:17:12`, mas foi reconhecido e carregado;
- dados estranhos na prévia do slot são uma limitação conhecida desse tipo de conversão e tendem a se corrigir depois que o jogo é carregado e salvo novamente em um save point.

Os nomes e os tempos acima servem apenas como exemplos de diagnóstico. O seu arquivo, ID regional, tempo e pasta provavelmente serão diferentes.

---

## 3. O que você vai precisar

### No PS2

- um PS2 capaz de executar homebrew, normalmente por Free McBoot;
- `wLaunchELF` ou `uLaunchELF`, geralmente iniciado por um arquivo `BOOT.ELF`;
- o Memory Card que contém o save;
- um pendrive compatível com o PS2, preferencialmente simples e formatado em FAT32.

### No computador

- Windows;
- Python 3;
- [Final Fantasy X HD Cross-platform Save Converter](https://github.com/mrhappyasthma/Final-Fantasy-X-HD-Cross-platform-Save-Converter);
- PS2 Save Builder;
- Bruteforce Save Data, também chamado de BSD;
- espaço para manter várias cópias de segurança.

### No PS3

- Final Fantasy X HD Remaster instalado e funcionando;
- acesso ao mesmo usuário do PS3 no qual o save final será jogado;
- um save-base criado pelo próprio jogo nesse usuário;
- um pendrive ou HD externo em FAT32, com a estrutura padrão `PS3/SAVEDATA`.

### Opcional

- [Apollo Save Tool para PS3](https://github.com/bucanero/apollo-ps3), útil para inspecionar, copiar e fazer backup de saves;
- um programa de FTP e um gerenciador como multiMAN, caso seja necessário mover arquivos dentro do PS3. Essa rota não é necessária para o método principal e foi justamente onde ocorreu o erro `8002920E` no caso real.

> Você não precisa baixar uma ISO do jogo para realizar a conversão. Uma ISO só teria utilidade para testar o save em emulador ou conferir a região da versão de PS2. Isso é um desvio opcional, não uma etapa do processo.

---

## 4. Regras de segurança antes de começar

Crie pelo menos estas quatro cópias:

1. **Backup A:** exportação original do Memory Card do PS2, sem alterações.
2. **Backup B:** arquivo bruto extraído do `.psu`.
3. **Backup C:** pasta original do save-base criada pelo próprio PS3.
4. **Backup D:** pasta final, já montada e criptografada, antes de copiá-la ao PS3.

Sugestão de organização:

```text
FFX_CONVERSAO
├── 01_PS2_ORIGINAL_PSU
├── 02_PS2_ARQUIVO_BRUTO
├── 03_PS3_SAVE_BASE_ORIGINAL
├── 04_SAVES_CONVERTIDO
└── 05_PS3_FINAL_CRIPTOGRAFADO
```

Nunca mova o único original. Sempre copie.

### Dados que não devem ser publicados

Se você for pedir ajuda em fórum, Reddit, Discord ou GitHub, não publique:

- Account ID da PSN;
- User ID do PS3;
- Console ID, PSID ou IDPS;
- códigos exibidos em ferramentas de resign;
- IP local do PS3;
- nome da conta do Windows;
- caminho completo das pastas pessoais do computador;
- `PARAM.SFO` ou `PARAM.PFD` de um save pessoal, salvo quando houver uma razão técnica clara e você compreender o risco;
- a pasta completa do save de outra pessoa.

Em capturas de tela, cubra esses campos. Em guias públicos, use caminhos genéricos como:

```text
C:\FFX_CONVERSAO\...
X:\PS3\SAVEDATA\...
```

O método deste guia reduz a necessidade de inserir IDs manualmente porque usa como base um save criado pelo próprio usuário e pelo próprio PS3 de destino.

---

## 5. Entender os arquivos envolvidos

| Arquivo ou pasta | Origem | Função |
| --- | --- | --- |
| pasta do jogo dentro de `mc0:/` ou `mc1:/` | Memory Card do PS2 | Contém o save original e seus metadados |
| arquivo `.psu` | wLaunchELF/uLaunchELF | Pacote de backup que preserva atributos do Memory Card |
| arquivo como `BISLPS-25088FF090600` | extraído do `.psu` | Dados brutos do save de FFX no PS2; é a entrada do conversor |
| `SAVES` | produzido pelo conversor | Dados de FFX no formato interno esperado pelo HD Remaster do PS3, ainda descriptografados |
| `PARAM.SFO` | save-base do PS3 | Metadados do save, título, pasta e dados relacionados ao proprietário |
| `PARAM.PFD` | save-base do PS3 | Estrutura de integridade, hashes e proteção dos arquivos do save |
| `ICON0.PNG` e outros recursos | save-base do PS3 | Ícone e possíveis imagens do save no XMB |

Não confunda estas duas camadas:

- o conversor corrige a estrutura e o checksum interno do **Final Fantasy X**;
- o Bruteforce Save Data cuida da criptografia e da integridade da pasta de save do **PlayStation 3**.

As duas coisas são necessárias.

---

## 6. Etapa A: retirar o save do Memory Card do PS2

### 6.1 Preparar o pendrive

1. Formate o pendrive em FAT32.
2. Coloque nele o `BOOT.ELF` correspondente ao wLaunchELF/uLaunchELF, caso sua instalação do Free McBoot ainda não tenha o programa configurado.
3. Insira o Memory Card com o save no PS2.
4. Insira o pendrive no PS2.
5. Inicie o wLaunchELF/uLaunchELF.

O wLaunchELF é o sucessor do uLaunchELF; os dois nomes aparecem em guias antigos. O programa funciona como gerenciador de arquivos do PS2.

### 6.2 Localizar o save

1. Abra `FileBrowser`.
2. Entre em `mc0:/` se o Memory Card estiver no slot 1, ou em `mc1:/` se estiver no slot 2.
3. Procure a pasta do Final Fantasy X.

O nome muda conforme a região e a edição. Exemplos de identificadores que podem aparecer:

- `SLPS-25088`: Final Fantasy X International japonês;
- `SCES-50490`: edição europeia/PAL;
- identificadores `SLUS` ou nomes iniciados por `BASLUS`: edições norte-americanas.

No caso real, a confirmação veio pelo arquivo `BISLPS-25088FF090600`.

Se houver vários saves de FFX, confira no jogo qual é o slot certo antes de exportar. Anote local, tempo, nome do personagem e data de gravação.

### 6.3 Criar o `.psu`

1. Destaque ou marque a pasta inteira do save no Memory Card.
2. Abra o menu de operações, normalmente com `R1`.
3. Escolha `Copy`.
4. Volte e entre em `mass:/`, que representa o pendrive.
5. Abra novamente o menu com `R1`.
6. Escolha **`psuPaste`**, não o `Paste` comum.
7. Aguarde a conclusão sem desligar o console ou remover o pendrive.

### Por que usar `psuPaste`?

O formato PSU preserva atributos, datas e metadados específicos do Memory Card. Copiar a pasta como arquivos comuns pode produzir um backup incompleto ou difícil de reimportar. Para uma exportação destinada ao computador, `psuPaste` é a opção correta.

### 6.4 Conferir no computador

1. Ejete o pendrive com segurança.
2. Conecte-o ao computador.
3. Copie o `.psu` para `01_PS2_ORIGINAL_PSU`.
4. Duplique o arquivo e deixe uma cópia intocada.
5. Verifique se o arquivo tem tamanho maior que zero.

---

## 7. Etapa B: extrair o arquivo bruto do `.psu`

1. Abra o PS2 Save Builder.
2. Use `File > Open` e selecione o `.psu`.
3. O programa exibirá os arquivos existentes dentro do pacote.
4. Procure o arquivo principal cujo nome contém o ID da edição do jogo.
5. Não escolha `icon.sys`, arquivos de ícone ou imagens.
6. Clique com o botão direito no arquivo principal e escolha `Extract`.
7. Salve-o em `02_PS2_ARQUIVO_BRUTO`.

No caso real, o arquivo correto era:

```text
BISLPS-25088FF090600
```

O repositório do conversor cita, de forma geral, arquivos iniciados por `BISLPS` ou `BASLUS`. Em outras regiões, o prefixo pode variar. O critério é identificar o arquivo grande de dados do jogo, e não os recursos visuais do save.

### Como verificar que você extraiu o arquivo certo

- o nome deve lembrar o ID regional da versão de PS2;
- ele não deve ser um PNG, um ícone nem `icon.sys`;
- o conversor deve aceitá-lo como entrada PS2;
- se testado na mesma versão do jogo de PS2, deve corresponder ao progresso esperado.

---

## 8. Etapa C: resolver a região do save de PS2

Esta foi uma fonte real de confusão.

`SCES-50490` e `SLPS-25088` não são o mesmo lançamento:

- `SCES-50490` identifica uma edição europeia/PAL;
- `SLPS-25088` identifica Final Fantasy X International japonês;
- traduções feitas por fãs podem usar como base qualquer uma dessas edições;
- uma ISO traduzida ser “europeia” não transforma um save de `SLPS-25088` em save de `SCES-50490`.

Se o jogo de PS2 não reconhecer o save durante um teste, a primeira hipótese deve ser incompatibilidade de região. Não renomeie cegamente a pasta nem o arquivo principal. Isso pode fazer o save aparecer sem tornar seus dados realmente compatíveis.

Para a conversão descrita aqui, o arquivo bruto `BISLPS-25088FF090600` foi aceito pelo conversor e transformado diretamente no `SAVES` de PS3. Portanto, não foi necessário converter o save de uma região de PS2 para outra antes da migração ao HD Remaster.

---

## 9. Etapa D: converter o arquivo bruto do PS2

### 9.1 Preparar o conversor

1. Acesse o repositório do [FFX HD Cross-platform Save Converter](https://github.com/mrhappyasthma/Final-Fantasy-X-HD-Cross-platform-Save-Converter).
2. Baixe o projeto completo, por exemplo por `Code > Download ZIP`.
3. Extraia o ZIP inteiro para uma pasta.
4. Não mova apenas o `ffx.pyw`: ele depende da pasta `Modules` e de outros arquivos do projeto.
5. Instale Python 3, se ainda não estiver instalado.

O próprio código do conversor exige Python 3.

### 9.2 Executar

Tente primeiro abrir `ffx.pyw` com duplo clique. Se nada acontecer:

1. abra o Terminal ou Prompt de Comando dentro da pasta do projeto;
2. execute:

```bat
py -3 ffx.pyw
```

Se o comando `py` não existir, tente:

```bat
python ffx.pyw
```

### 9.3 Configurar a conversão

Na janela do programa:

1. em `Game`, escolha `Final Fantasy X`;
2. em `Save File Type`, escolha `PS2`;
3. em `Target Console`, escolha `PS3 (decrypted)`;
4. clique em `Convert save file`;
5. selecione o arquivo bruto extraído pelo PS2 Save Builder;
6. confirme a conversão;
7. localize o novo arquivo chamado `SAVES`.

Copie esse arquivo para `04_SAVES_CONVERTIDO` e não o edite manualmente.

### Observação importante

O resultado é `SAVES` **descriptografado**. Ele ainda não pode ser colocado sozinho no PS3. Primeiro precisa entrar dentro de um save-base válido e passar pela criptografia/reconstrução do `PARAM.PFD`.

---

## 10. Etapa E: criar um save-base real no PS3

Esta é a parte que evita a necessidade de inventar metadados ou usar códigos de conta de terceiros.

1. No usuário do PS3 em que você pretende jogar, inicie exatamente a versão de Final Fantasy X HD Remaster que receberá o save.
2. Comece um jogo novo ou use um progresso descartável.
3. Grave normalmente em um save point.
4. Saia do jogo.
5. No XMB, abra `Jogo > Utilitário de Dados Salvos (PS3)`.
6. Localize o save de Final Fantasy X HD Remaster.
7. Pressione `Triângulo > Copiar` e copie-o para um dispositivo FAT32.
8. No computador, abra:

```text
X:\PS3\SAVEDATA\
```

9. Copie a pasta inteira do save para `03_PS3_SAVE_BASE_ORIGINAL`.
10. Faça uma segunda cópia para trabalhar.

Na edição europeia usada no caso real, a pasta era semelhante a:

```text
BLES01880SAVEDIR---------------
```

Não suponha que esse nome servirá para todos. Use a pasta criada pela sua própria edição do jogo.

Uma estrutura mínima observada foi:

```text
PS3
└── SAVEDATA
    └── BLES01880SAVEDIR---------------
        ├── ICON0.PNG
        ├── PARAM.PFD
        ├── PARAM.SFO
        └── SAVES
```

Sua pasta pode conter arquivos visuais adicionais. Preserve todos eles.

### Por que o save-base precisa vir do próprio PS3?

Porque ele já contém:

- o ID correto da versão de destino;
- metadados válidos;
- a estrutura esperada pelo XMB;
- dados do usuário correto;
- um `PARAM.PFD` legítimo para ser reconstruído;
- ícones e recursos próprios daquela edição.

Baixar um save-base aleatório da internet adiciona problemas de região, propriedade e resign desnecessários.

---

## 11. Etapa F: descriptografar o save-base no Bruteforce Save Data

> Bruteforce Save Data é uma ferramenta antiga. Obtenha-a de uma fonte reputável, analise o instalador com seu antivírus e, se possível, use-a offline ou em uma máquina virtual. Não desative permanentemente proteções do sistema apenas para executá-la.

### 11.1 Se o programa pedir um perfil do PS3

Algumas instalações do Bruteforce já possuem um perfil configurado; outras pedem que o usuário crie um. Use somente dados extraídos de um save feito pelo **seu próprio PS3 e pelo seu próprio usuário**.

Em versões antigas do programa, o fluxo costuma ser:

1. abrir `Settings > Global Settings`;
2. criar ou selecionar um perfil local;
3. apontar para o `PARAM.SFO` do save-base quando solicitado;
4. deixar a ferramenta ler os identificadores necessários ou preencher localmente os campos pedidos, como User ID e Console ID/PSID;
5. salvar a configuração apenas no computador usado para a conversão.

Esses valores não devem ser incluídos em capturas de tela, tutoriais, arquivos ZIP ou pedidos públicos de ajuda. Não copie os valores de outro usuário e não use os números de exemplo de guias antigos. Cada console e usuário podem ter dados diferentes.

Como o save-base deste procedimento foi criado pelo próprio usuário de destino, não é necessário transformá-lo em save de outra conta. A configuração serve para a ferramenta abrir e reconstruir corretamente a proteção do save local.

### 11.2 Descriptografar

1. Abra o Bruteforce Save Data.
2. Configure em `Path for SAVEDATA folders` a pasta que contém os diretórios de saves, por exemplo:

```text
C:\FFX_CONVERSAO\TRABALHO\PS3\SAVEDATA
```

Não aponte diretamente para o arquivo `SAVES`.

3. Aguarde o programa listar o save de Final Fantasy X HD Remaster.
4. Selecione o save correto.
5. Use a opção de descriptografar os arquivos, normalmente:

```text
Decrypt PFD > Decrypt All Files
```

Em algumas versões, o texto aparece apenas como `Decrypt all files`.

6. Confirme a operação.
7. Verifique se o estado do save ficou verde.
8. Verifique se a ferramenta criou um marcador como:

```text
~files_decrypted_by_pfdtool.txt
```

Esse marcador indica quais arquivos estão descriptografados; ele não é um arquivo de jogo.

---

## 12. Etapa G: substituir somente o `SAVES`

Com o save-base descriptografado:

1. abra a pasta de trabalho do save-base;
2. renomeie temporariamente o `SAVES` original para algo como `SAVES.base.bak`, ou mantenha-o fora da pasta em seu backup;
3. copie para a pasta o `SAVES` produzido pelo conversor;
4. confirme que o nome final é exatamente `SAVES`, sem extensão;
5. não substitua `PARAM.SFO`;
6. não substitua `PARAM.PFD` manualmente;
7. não substitua os ícones;
8. não misture arquivos vindos de outro save de PS3.

O conteúdo da pasta deve continuar sendo o save-base legítimo, com apenas os dados de jogo trocados.

### Verificação útil

Antes e depois da substituição, confira:

- tamanho do arquivo;
- data de modificação;
- nome exato;
- ausência de extensão oculta, como `SAVES.bin` ou `SAVES.txt`.

Se quiser uma conferência rigorosa, calcule o SHA-256 do `SAVES` convertido antes de copiá-lo e novamente dentro da pasta. Os hashes devem ser iguais.

No PowerShell:

```powershell
Get-FileHash .\SAVES -Algorithm SHA256
```

O hash é seguro para compartilhar; o arquivo pessoal não é.

---

## 13. Etapa H: criptografar e reconstruir o save

Volte ao Bruteforce Save Data, mantendo o mesmo save selecionado.

Use:

```text
Encrypt PFD > Encrypt Decrypted Files
```

No caso real, este detalhe foi decisivo:

- usar `Encrypt Decrypted Files`;
- não usar `Update PFD` antes;
- não escolher `Encrypt All Files` como primeira tentativa;
- não aplicar depois um segundo checksum pelo Apollo.

O conversor já preparou o checksum interno de FFX. Nesta etapa, o Bruteforce deve criptografar o arquivo substituído e atualizar a proteção do conjunto do PS3.

Quando terminar:

1. confirme que não houve mensagem de erro;
2. confira se o estado já não aparece como descriptografado;
3. remova qualquer arquivo auxiliar `~files_decrypted_by_pfdtool.txt` que a própria ferramenta não tenha removido;
4. não remova arquivos legítimos do save-base;
5. copie a pasta concluída para `05_PS3_FINAL_CRIPTOGRAFADO`.

---

## 14. Etapa I: devolver o save ao PS3

No dispositivo FAT32, monte a estrutura exatamente assim:

```text
X:\PS3\SAVEDATA\PASTA_DO_SAVE\
```

Exemplo da edição europeia usada no teste:

```text
X:\PS3\SAVEDATA\BLES01880SAVEDIR---------------\
```

Dentro da pasta devem estar o `PARAM.SFO`, o `PARAM.PFD`, o `SAVES` criptografado e todos os arquivos legítimos do save-base.

Depois:

1. ejete o dispositivo com segurança;
2. conecte-o ao PS3;
3. no XMB, abra `Jogo > Utilitário de Dados Salvos (PS3)`;
4. abra o dispositivo USB;
5. selecione o save de Final Fantasy X HD Remaster;
6. pressione `Triângulo > Copiar`;
7. se já existir um save no armazenamento interno, só confirme a substituição depois de garantir que o backup original está seguro;
8. inicie o jogo normalmente;
9. escolha **`Load`**, não `Continue`;
10. tente carregar o slot convertido.

---

## 15. O slot apareceu com tempo ou personagens errados. E agora?

Isso pode ser normal.

O próprio projeto do conversor registra dois efeitos conhecidos:

- o nome personalizado do protagonista pode voltar para Tidus;
- a prévia da lista de saves pode mostrar tempo de jogo e integrantes do grupo incorretos.

No caso real, o save original tinha aproximadamente 99 horas, mas a lista do HD Remaster mostrou `24:17:12` antes do primeiro carregamento. Ainda assim, o jogo reconheceu e abriu o save.

Faça o teste correto:

1. tente carregar o slot mesmo que a prévia pareça estranha;
2. confira dentro do jogo o local, personagens, itens, equipamentos e progresso;
3. vá até um save point;
4. salve em outro slot, preservando o primeiro convertido;
5. volte à tela inicial;
6. confira novamente a prévia e carregue o novo slot.

O novo salvamento feito pelo próprio HD Remaster tende a reconstruir os metadados visuais corretamente.

---

## 16. Erros encontrados na conversão real e suas soluções

### Erro 1: testar o save com a versão errada do jogo de PS2

**Sintoma:** o save não aparece ou não é reconhecido ao testar uma ISO.

**Causa provável:** confusão entre `SCES-50490` e `SLPS-25088`.

**Solução:** identificar a região pelo nome do arquivo do save e usar a mesma edição apenas para testes. No caso real, `BISLPS-25088FF090600` apontava para FFX International/`SLPS-25088`, não para a edição europeia `SCES-50490`.

**Lição:** não é preciso testar por ISO para converter ao PS3. Se testar, a região precisa coincidir.

### Erro 2: extrair ou selecionar o arquivo errado dentro do `.psu`

**Sintoma:** o conversor recusa o arquivo, encerra ou produz saída inválida.

**Causa provável:** seleção de `icon.sys`, imagem ou outro recurso.

**Solução:** abrir o `.psu` no PS2 Save Builder e extrair o arquivo grande cujo nome contém o ID do jogo, como `BISLPS-25088FF090600`.

### Erro 3: executar apenas o `ffx.pyw`

**Sintoma:** a janela não abre ou aparece erro de módulo ausente.

**Causa provável:** o script foi retirado da pasta do projeto sem a pasta `Modules`.

**Solução:** extrair e manter o repositório inteiro unido; executar o `ffx.pyw` dentro dele com Python 3.

### Erro 4: o PS3 continua mostrando o save-base curto

**Sintoma:** depois da suposta importação, o jogo ainda mostra o progresso descartável, por exemplo 23 minutos.

**Causa:** o `SAVES` convertido não chegou à pasta final, foi colocado no diretório errado ou a operação de importação/recriptografia não foi concluída.

**Solução:** parar, voltar ao backup e confirmar tamanho, data e hash do `SAVES` em cada estágio. Se o progresso curto continua, você ainda está carregando o `SAVES` do save-base.

### Erro 5: tentativa pelo diretório temporário do Apollo falha

**Procedimento tentado:**

1. copiar o save-base `BLES01880` com Apollo;
2. exportar ou descriptografar `SAVES` para um diretório temporário semelhante a:

```text
/dev_hdd0/tmp/apollo/PASTA_DO_SAVE/SAVES
```

3. substituir esse arquivo por FTP, usando um gerenciador como multiMAN;
4. voltar ao Apollo e tentar aplicar checksum, alterações e resign.

**Resultado:** erro `8002920E`.

**Diagnóstico prático:** trocar um arquivo dentro do temporário não garantiu que Apollo produzisse uma pasta final com criptografia e `PARAM.PFD` válidos para aquele fluxo. O diretório temporário não deve ser confundido com um save já pronto para o XMB.

**Solução que funcionou:** abandonar aquela cópia de trabalho, restaurar o save-base original e completar a descriptografia, a substituição do `SAVES` e `Encrypt Decrypted Files` no Bruteforce Save Data.

### Erro 6: Bruteforce não lista o save

**Causas prováveis:**

- caminho apontando para a pasta individual, e não para o diretório `SAVEDATA`;
- estrutura `PS3/SAVEDATA` quebrada;
- `PARAM.SFO` ausente;
- save-base copiado de forma incompleta;
- dispositivo ou pasta sem permissão de escrita.

**Solução:** manter a pasta completa criada pelo PS3 e apontar `Path for SAVEDATA folders` para o diretório que contém a pasta do save.

### Erro 7: save fica verde no Bruteforce

**Isso não é necessariamente erro.** Verde costuma indicar que os arquivos estão descriptografados e disponíveis para edição.

Depois de substituir `SAVES`, finalize com:

```text
Encrypt PFD > Encrypt Decrypted Files
```

### Erro 8: “dados corrompidos” no XMB

**Causas mais comuns:**

- `SAVES` ainda descriptografado;
- `PARAM.PFD` não atualizado;
- pasta regional errada;
- `PARAM.SFO` vindo de outra edição ou de outra pessoa;
- estrutura de diretórios errada;
- arquivo renomeado com extensão invisível;
- cópia interrompida.

**Solução:** não tente reparar a pasta corrompida em sequência. Volte ao save-base intacto do próprio PS3, repita a descriptografia, substitua somente `SAVES` e use `Encrypt Decrypted Files`.

### Erro 9: o save aparece, mas a prévia mostra tempo errado

**Exemplo real:** `24:17:12` em vez das aproximadamente 99 horas esperadas.

**Solução:** carregar o save. Se o progresso interno estiver correto, gravar novamente pelo próprio jogo em outro slot. A prévia errada, sozinha, não prova corrupção.

### Erro 10: o jogo abre pelo `Continue`, mas entra no lugar errado

**Causa:** `Continue` pode abrir o último slot previamente usado, não necessariamente o convertido.

**Solução:** entrar em `Load` e escolher manualmente o slot convertido.

### Erro 11: `mass:/` não aparece no wLaunchELF

**Causas prováveis:** pendrive incompatível, sistema de arquivos não suportado ou conexão tardia.

**Soluções:**

- usar FAT32;
- testar outro pendrive, de preferência menor e mais antigo;
- conectar antes de iniciar o wLaunchELF;
- reiniciar o FileBrowser;
- evitar adaptadores e partições complexas.

### Erro 12: backup copiado com `Paste` comum

**Risco:** perda de atributos próprios do Memory Card.

**Solução:** refazer a exportação usando `Copy` na origem e `psuPaste` em `mass:/`.

---

## 17. Por que a rota do Bruteforce funcionou melhor

O método bem-sucedido separou claramente três responsabilidades:

1. **PS2 Save Builder:** retirar do pacote `.psu` o arquivo bruto correto.
2. **FFX HD Cross-platform Save Converter:** transformar os dados internos do FFX e recalcular o checksum do jogo.
3. **Bruteforce Save Data:** inserir esses dados numa estrutura legítima de PS3, criptografá-los e reconstruir a proteção PFD.

Na tentativa pelo Apollo, as fases ficaram misturadas entre diretório temporário, substituição por FTP, checksum e resign. Quando apareceu `8002920E`, já não havia garantia de que todas as camadas estavam sincronizadas.

Apollo continua sendo uma ferramenta útil. Porém, para este caso específico e para esta sequência testada, o método do PC tornou mais visível qual arquivo estava descriptografado e em qual momento a criptografia seria refeita.

---

## 18. Checklist curto da rota comprovada

- [ ] Fazer backup do Memory Card.
- [ ] Exportar a pasta do FFX com `psuPaste` para gerar `.psu`.
- [ ] Abrir o `.psu` no PS2 Save Builder.
- [ ] Extrair o arquivo principal do jogo, não os ícones.
- [ ] Abrir `ffx.pyw` com Python 3.
- [ ] Escolher entrada `PS2` e destino `PS3 (decrypted)`.
- [ ] Gerar `SAVES`.
- [ ] Criar um save-base no próprio FFX HD Remaster do PS3.
- [ ] Copiar e duplicar a pasta completa do save-base.
- [ ] Descriptografar o save-base no Bruteforce Save Data.
- [ ] Substituir somente `SAVES`.
- [ ] Usar `Encrypt PFD > Encrypt Decrypted Files`.
- [ ] Colocar a pasta final em `PS3/SAVEDATA` no dispositivo FAT32.
- [ ] Copiar pelo Utilitário de Dados Salvos do XMB.
- [ ] Abrir o jogo e escolher `Load`.
- [ ] Carregar mesmo que a prévia inicialmente pareça estranha.
- [ ] Conferir o progresso dentro do jogo.
- [ ] Salvar novamente, de preferência em outro slot.

---

## 19. Árvore de decisão para diagnóstico

### O conversor não aceita a entrada

Voltar ao `.psu` e conferir se o arquivo principal foi extraído, em vez de um ícone ou do próprio pacote `.psu`.

### O Bruteforce não encontra o save

Conferir a estrutura `PS3/SAVEDATA/PASTA_DO_SAVE` e apontar o programa para `SAVEDATA`.

### O XMB mostra “dados corrompidos”

O problema está na camada do PS3: criptografia, `PARAM.PFD`, `PARAM.SFO`, região ou estrutura da pasta. Voltar ao save-base intacto.

### O XMB aceita, mas o jogo não mostra o slot

Conferir se a pasta corresponde à mesma região/Title ID da instalação de destino e se o arquivo final se chama exatamente `SAVES`.

### O jogo mostra o slot com informações estranhas

Tentar carregar. A prévia pode estar incorreta depois da conversão.

### O jogo carrega o progresso curto do save-base

O `SAVES` novo não foi realmente incorporado. Comparar data, tamanho e hash e repetir a etapa de substituição.

### O jogo carrega o progresso antigo correto

Ir a um save point, gravar em outro slot, reiniciar o jogo e testar novamente. Conversão concluída.

---

## 20. O que não fazer

- Não trabalhar sobre a única cópia do Memory Card.
- Não usar uma ISO de região diferente para concluir que o save está perdido.
- Não selecionar `icon.sys` como entrada do conversor.
- Não mover `ffx.pyw` para longe da pasta `Modules`.
- Não colocar o `SAVES` descriptografado diretamente no PS3.
- Não substituir `PARAM.SFO` ou `PARAM.PFD` por arquivos aleatórios.
- Não usar um save-base de outra pessoa quando você pode criar o seu.
- Não aplicar sucessivamente várias rotinas de checksum “para garantir”.
- Não insistir numa pasta que já apresentou `8002920E`; voltar ao backup limpo.
- Não confiar apenas no tempo mostrado na prévia do slot.
- Não escolher `Continue` como teste definitivo.
- Não publicar IDs de conta, console, IPs ou caminhos pessoais.
- Não distribuir ISO, arquivos do jogo ou saves pessoais junto com o guia.

---

## 21. Como pedir ajuda sem expor dados pessoais

Uma boa mensagem de suporte contém:

```text
Origem: FFX de PS2, região/ID [INFORMAR]
Nome genérico do arquivo bruto: [INFORMAR]
Formato exportado: .psu
Destino: FFX HD Remaster de PS3, Title ID [INFORMAR]
Conversor: commit/versão [INFORMAR]
Etapa que falhou: [INFORMAR]
Mensagem ou código de erro: [INFORMAR]
Tamanho do arquivo SAVES antes/depois: [INFORMAR]
SHA-256 do SAVES: [INFORMAR]
O save-base foi criado no próprio PS3: sim/não
O SAVES foi recriptografado: sim/não
```

Você pode publicar o hash, os tamanhos e os IDs comerciais das versões do jogo. Não publique identificadores exclusivos da conta ou do console.

---

## 22. Referências técnicas

- [FFX HD Cross-platform Save Converter, repositório oficial](https://github.com/mrhappyasthma/Final-Fantasy-X-HD-Cross-platform-Save-Converter)
- [Wiki do conversor: extração de saves do PS2](https://github.com/mrhappyasthma/Final-Fantasy-X-HD-Cross-platform-Save-Converter/wiki/Playstation-2-(PS2)-Guide)
- [Wiki do conversor: saves do PS3](https://github.com/mrhappyasthma/Final-Fantasy-X-HD-Cross-platform-Save-Converter/wiki/Playstation-3-(PS3)-Guide)
- [Wiki do conversor: problemas conhecidos](https://github.com/mrhappyasthma/Final-Fantasy-X-HD-Cross-platform-Save-Converter/wiki/Known-Issues)
- [Apollo Save Tool, repositório oficial](https://github.com/bucanero/apollo-ps3)
- [wLaunchELF, repositório oficial](https://github.com/ps2homebrew/wLaunchELF)
- [Apollo save-decrypters e checksum fixers](https://github.com/bucanero/save-decrypters)

### Limitações das fontes

O autor do conversor classifica PS2 → PS3 como implementado, mas não como verificado por ele próprio na tabela do projeto. A wiki de PS3 também deixa incompleta a seção de recriptografia de saves vindos de outra plataforma. Por isso este guia dá destaque ao fluxo que foi efetivamente concluído no caso real: save-base legítimo do próprio PS3, troca apenas do `SAVES` e `Encrypt Decrypted Files` no Bruteforce Save Data.

---

## 23. Conclusão

A conversão é possível, mas envolve três formatos e duas camadas de integridade diferentes. O erro mais comum é acreditar que gerar `SAVES` encerra o processo. Não encerra: ele ainda precisa ser incorporado a uma pasta legítima de PS3 e recriptografado.

A rota mais segura é conservadora:

- preservar o save original do PS2;
- extrair apenas os dados corretos;
- deixar o conversor cuidar da transformação interna do FFX;
- deixar um save-base real fornecer a identidade e a estrutura do PS3;
- deixar o Bruteforce recriptografar somente os arquivos descriptografados;
- testar por `Load`;
- salvar novamente dentro do próprio jogo.

Se a prévia estiver estranha, mas o jogo carregar e o progresso interno estiver correto, a conversão deu certo.
