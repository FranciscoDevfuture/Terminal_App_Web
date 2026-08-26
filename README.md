# Terminal — Assistentes (Ronda + Ocorrências)

## Atualização: câmera não abria + edição após relatório finalizado

**Câmera não abria (nos dois apps):** o `<input type="file" capture="environment">`
puro em HTML não funciona de forma confiável dentro de um WebView empacotado
em APK — falta a permissão nativa e o tratamento de intent que só existem
usando o plugin oficial de câmera. Troquei pelo `@capacitor/camera` (plugin
nativo de verdade) nos dois apps, sempre com **câmera traseira**. Fora do
app (navegador comum), continua funcionando como antes.

**Bônus — Ocorrências também tinha o mesmo problema de CDN que já resolvemos
na Ronda várias vezes:** o salvamento/compartilhamento de PDF carregava
`@capacitor/filesystem` e `@capacitor/share` via `import` de um CDN
(`esm.sh`) em tempo de execução — mesma fragilidade de rede que já
corrigimos antes. Troquei para usar os plugins nativos já embutidos no
projeto (`window.Capacitor.Plugins`), sem depender de internet.

**Editar relatório depois de finalizado (Ronda):** antes, o botão ✏️ Editar
travava assim que a ronda era finalizada. Agora continua funcionando depois
do relatório pronto — edita qualquer campo ou ocorrência e o relatório é
regenerado na hora com os dados atualizados. (O Agente de Ocorrências já
tinha esse recurso, via o botão de correção com IA na tela do relatório.)

---

Os dois apps unidos num só, sem misturar o código de cada um (o que
arriscaria conflito de nomes de elementos/variáveis entre eles). A
estrutura é simples: uma tela inicial oferece as duas opções, e cada
app abre como sua própria página dentro do mesmo projeto/APK.

## Estrutura

```
www/
  index.html        ← tela inicial (escolher Ronda ou Ocorrências)
  ronda.html         ← Assistente de Ronda (o app que já existia)
  ocorrencias.html    ← Agente de Ocorrências (o app que já existia)
  manifest.json, ícones
```

Cada app ganhou um link "🏠 Início" no topo pra voltar pra tela inicial.

## O que foi ajustado em cada arquivo

- **ronda.html**: nada na lógica — só troquei o `<script src="https://cdnjs...">`
  do html2pdf.js por uma cópia embutida no próprio arquivo (mesmo motivo de
  sempre: não depender de internet/CDN dentro do WebView do app).
- **ocorrencias.html**: mesma troca, mas com o jsPDF (a biblioteca de PDF que
  esse app usa) — também embutido agora.
- Ambos ganharam o link "🏠 Início" no cabeçalho/tela inicial.

Como cada app mora no seu próprio arquivo/documento (não estão misturados
num HTML só), não existe risco de uma variável ou ID de um colidir com o
outro — cada um roda isolado, como se fosse uma aba separada.

O histórico de cada app fica salvo separadamente (chaves diferentes no
armazenamento local do navegador/app: `ronda_history` para a Ronda,
`ocorrencias`/`config` para o Agente de Ocorrências) — não se misturam.

## Como gerar o APK

```
npm install
npx cap sync android
```
Depois abra a pasta `android/` no Android Studio e Build → Build APK(s),
como de costume.

## Personalização

- **Nome do app / appId**: `capacitor.config.json` (atualmente
  `com.terminal.assistentes` / "Terminal Assistentes").
- **Tela inicial**: `www/index.html` — os dois cartões (ícone, título,
  descrição) e o link de destino de cada um.
- **Ícone**: os PNGs em `android/app/src/main/res/mipmap-*` (mesmo ícone da
  Ronda, reaproveitado para o app combinado — troque se quiser um ícone
  próprio para o app unificado).
