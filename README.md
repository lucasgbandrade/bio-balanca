# Bio Balança

App web que lê o peso de balanças de bioimpedância Bluetooth direto do navegador, sem instalar nada, e calcula estimativas de composição corporal, como as do app **OKOK International**.

É um único arquivo `index.html`: sem servidor, sem conta e sem coleta de dados.

## Funcionalidades

- **Leitura por Bluetooth (Web Bluetooth)** de balanças:
  - **OKOK / Chipsea (formato C0)**, que transmitem o peso pelo sinal Bluetooth e não aceitam conexão. Testado com o produto `0x0A01`.
  - **Padrão Bluetooth SIG**: Weight Scale (`0x181D`) e Body Composition (`0x181B`).
  - Outros formatos, com calibração automática (você digita o peso do display e o app encontra o valor nos dados).
- **Busca por sinal**: lista os aparelhos próximos pela força do sinal e reconhece a balança OKOK sozinha.
- **Composição corporal estimada**: IMC, gordura (% e kg), massa magra, massa muscular, músculo esquelético, água, proteína, massa óssea, gordura visceral, metabolismo basal e idade metabólica.
- **Comparação imediata** com a última medição salva (peso, gordura, músculo e IMC).
- **Histórico** com gráfico de peso e gordura.
- **Perfil** (sexo, idade e altura) usado nos cálculos.
- **Aba Avançado**: log dos dados recebidos, calibração, configuração manual da impedância e modo demonstração.
- Interface escura, pensada para celular.

## Compatibilidade

| Plataforma | Navegador | Funciona? |
| --- | --- | --- |
| Android | Chrome ou Edge | ✅ |
| Windows, macOS, Linux | Chrome ou Edge | ✅ |
| iPhone / iPad | Bluefy (App Store) | ✅ |
| iPhone / iPad | Safari | ❌ (não tem Web Bluetooth) |
| Qualquer | Firefox | ❌ |

O Web Bluetooth só funciona em páginas **https** (ou em arquivo local).

### Balanças OKOK: ativar a busca por sinal

As balanças OKOK só transmitem o peso pelo sinal Bluetooth, e o Chrome precisa de um recurso experimental para ler esse sinal:

1. Abra `chrome://flags`.
2. Procure **Experimental Web Platform features** e marque **Enabled**.
3. Toque em **Relaunch**.

## Como usar

1. Abra o app no Chrome.
2. Preencha sexo, idade e altura na aba **Perfil**.
3. Feche o app OKOK, se ele estiver aberto, para não disputar a balança.
4. Na tela **Medir**, toque em **Buscar por sinal**. Se a sua balança aceitar conexão, use **Conectar balança**.
5. Suba na balança. O peso aparece em tempo real e trava no resultado final.
6. Toque em **Salvar** para guardar a medição no histórico.

## Protocolo OKOK / Chipsea (formato C0)

A balança não aceita conexão: os dados vêm nos dados de fabricante (manufacturer data) do sinal Bluetooth.

- **Company ID:** byte baixo `0xC0`; o byte alto é um contador que muda a cada pacote.
- **13 bytes de dados:**

| Bytes | Conteúdo |
| --- | --- |
| 0–1 | Peso, big-endian, ÷100 (ex.: `1F 5E` = 80,30 kg) |
| 2–3 | Impedância, big-endian, ÷10 |
| 4–5 | ID do produto (ex.: `0A 01`) |
| 6 | Atributos: bit 0 = medição final (`0x24` medindo, `0x25` final); bits 1–2 = casas decimais; bits 3–4 = unidade (0 = kg, 1 = jin, 2 = lb) |
| 7–12 | Endereço (MAC) da balança |

Exemplo de pacote final:

```
c0 cd 1f 5e 17 70 0a 01 25 34 5c f3 b2 ce 40
```

> **Atenção:** nas balanças testadas, a impedância é **sempre 600,0 Ω** (`17 70`), mesmo sem ninguém em cima, ou seja, ela não é medida de verdade. Por isso a composição corporal, tanto no app OKOK quanto aqui, depende só de peso, altura, idade e sexo.

## Cálculos

- **Balanças OKOK:** fórmulas do app OKOK (CsAlgoBuilder, para balanças de 4 eletrodos), levantadas por engenharia reversa.
- **Outras balanças com impedância real:** massa magra pela equação BIA de Sun et al. (2003), metabolismo basal por Katch-McArdle.
- **Sem impedância:** IMC e metabolismo basal por Mifflin-St Jeor.

Os resultados são **estimativas** e não substituem avaliação médica ou nutricional.

## Privacidade

Perfil, histórico e configurações ficam apenas no `localStorage` do navegador. Nada é enviado para servidores. Os dados ficam separados por endereço: abrir o app em outro domínio começa do zero.

## Publicar

O projeto é estático e pode ser publicado em qualquer hospedagem com https.

**Vercel:** importe este repositório em [vercel.com](https://vercel.com) e clique em **Deploy**, sem nenhuma configuração. Cada novo commit publica uma nova versão automaticamente.

**Pela linha de comando:**

```bash
npx vercel --prod
```

**Localmente:** abra o `index.html` no Chrome ou rode um servidor simples:

```bash
npx serve .
```

## Instalar como app no celular

O Bio Balança é um PWA: depois de publicado em https, pode ser instalado e abre em tela cheia, com ícone próprio e funcionando sem internet.

- **Android (Chrome):** toque no botão **📲 Instalar como app** na tela inicial, ou no menu **⋮ → Instalar app**.
- **iPhone:** no Safari, toque em **Compartilhar → Adicionar à Tela de Início**. Para usar o Bluetooth, abra o app pelo navegador Bluefy.

Ao publicar uma nova versão do `index.html`, aumente o número em `CACHE` no `sw.js` (ex.: `bio-balanca-v2`) para os celulares baixarem a atualização.

## Estrutura

```
.
├── index.html              # app completo (HTML, CSS e JavaScript)
├── manifest.json           # nome, cores e ícones do app instalado
├── sw.js                   # service worker: cache e funcionamento offline
├── icon-192.png            # ícone do app
├── icon-512.png            # ícone do app (alta resolução)
├── icon-maskable-512.png   # ícone adaptável do Android
├── apple-touch-icon.png    # ícone do iPhone
└── README.md
```

## Referências

- [openScale](https://github.com/oliexdev/openScale): suporte a balanças OKOK/Chipsea.
- [hub-saude](https://github.com/pedrohsipriano/hub-saude): engenharia reversa do formato C0 e das fórmulas do app OKOK.
- [Web Bluetooth Scanning API](https://webbluetoothcg.github.io/web-bluetooth/scanning.html).

## Aviso

Projeto independente, sem relação com a OKOK International ou a Chipsea. Use por sua conta e risco.
