# Como apresentar o dashboard EnergyGreen

Roteiro para demonstrar a página web ao vivo, usando os CSVs da pasta [exemplos/](../exemplos/). Cada arquivo foi feito para mostrar uma funcionalidade ou um tratamento de erro diferente.

> **Aviso importante:** os CSVs de `exemplos/` são **sintéticos** (gerados por `scripts/gerar_csvs_exemplo.py`, semente 42). Eles servem para demonstrar a interface, **não para validar o modelo**. As métricas de qualidade do trabalho (MAE, RMSE, WAPE, R²) vêm do conjunto de teste real da UCI, em `models/metrics.json`. Diga isso à banca antes de alguém perguntar.

Duração sugerida: **8 a 10 minutos**.

---

## 1. Preparação (faça antes de começar)

1. Ative o ambiente e suba o servidor:
   ```powershell
   .\.venv\Scripts\Activate.ps1
   uvicorn app.main:app
   ```
2. Abra `http://localhost:8000` e confirme que a página carrega sem o erro "Modelos não treinados". Se aparecer, rode `python -m src.train`.
3. Deixe a pasta `exemplos/` aberta no Explorer, pronta para arrastar arquivos.
4. Se o `Usar dados de exemplo` for mostrar o conjunto real, confirme antes que `python -m src.data` já foi executado (o botão depende de `data/`).
5. Teste o fluxo inteiro uma vez antes da apresentação. A série fica **em memória**: se o servidor reiniciar, é preciso carregar o CSV de novo.
6. Feche notificações e deixe o navegador em tela cheia (F11). Se a sala for clara, o tema claro projeta melhor.

---

## 2. Visão geral dos arquivos de exemplo

| Arquivo | Conteúdo | Use para mostrar |
|---|---|---|
| `01_casa_tipica_30dias.csv` | 30 dias, horário, todas as colunas | **Fluxo principal**: máquina do tempo, previsão, custo |
| `02_so_consumo_21dias.csv` | 21 dias, só `Global_active_power` | Aviso de **medições estimadas** (colunas ausentes) |
| `03_formato_uci_por_minuto_14dias.csv` | 14 dias, **por minuto**, formato bruto da UCI (`;`, `dd/mm/aaaa`, `?`) | Que o sistema aceita o formato original e agrega por hora |
| `04_com_lacunas_28dias.csv` | 28 dias com uma queda de 5 h e outra de 2 dias | **Preenchimento de lacunas** |
| `05_picos_e_desperdicio_30dias.csv` | 30 dias com picos fortes à noite e consumo fantasma alto | **Alertas** de pico e desperdício |
| `06_invalido_historico_curto_5dias.csv` | 5 dias (120 h) | **Recusa** com mensagem em português |

Ordem recomendada na apresentação: **01 → 05 → 04 → 02 → 03 → 06**. Começa pelo caminho feliz, mostra o recurso mais impressionante (alertas), depois fecha com robustez.

---

## 3. Roteiro passo a passo

### Abertura (30 s)

> "O EnergyGreen prevê o consumo de energia de uma casa nas próximas 24 horas usando redes LSTM e GRU, e transforma essa previsão em decisões: quanto vai custar, quando há pico e onde há desperdício."

Mostre a tela inicial vazia. Ela tem um estado vazio de propósito: nada aparece até haver dados.

### Passo 1: Carregar dados com o `01_casa_tipica_30dias.csv` (1 min)

1. Arraste o arquivo para a área de upload.
2. Aponte o resumo que aparece: período, número de horas, horas preenchidas, média e pico em kW.
3. Observe os **cartões de resumo**: consumo previsto das próximas 24 h (kWh), pico previsto e horário, custo estimado e variação contra as 24 h anteriores.

Fala sugerida: "Em um segundo a série entrou, foi limpa pelo mesmo código do treino e já gerou uma previsão."

### Passo 2: Máquina do tempo (2 min), o destaque

O modelo precisa de **168 h (7 dias)** de histórico. Por isso, nos 30 dias do arquivo 01, só dá para escolher momentos **a partir do 8º dia**.

1. Mova o controle deslizante (ou use o seletor de data) para um dia no meio do arquivo.
2. Mostre o gráfico: as 168 h de entrada e a previsão das 24 h seguintes, com o baseline (repete o dia anterior) sempre visível.
3. Clique em **"Revelar o real"**. O que de fato aconteceu é sobreposto, e aparecem MAE, RMSE e WAPE **daquele dia**.
4. Use **◀ ▶** para andar um dia, depois **"Reproduzir"** para avançar dia a dia como uma animação.
5. Alterne **LSTM ↔ GRU** e compare com o baseline.

Fala sugerida: "É como voltar no tempo e perguntar ao modelo: se eu estivesse naquele dia, o que você preveria? Depois conferimos."

Pontos a destacar:
- Ao chegar ao **último dia** da série, a referência fica além do fim: não existe "real", então o botão revelar não se aplica e o sistema trata isso. É exatamente o caso de uso em produção (prever o futuro de verdade).
- Alguns dias o modelo vai errar mais que o baseline. Diga isso com naturalidade: na avaliação real, o LSTM foi melhor que o baseline em 139 de 208 dias avaliados (67%), não em todos.

### Passo 3: Simulador de custo e tarifa (1,5 min)

1. Altere o preço do kWh e a tarifa de ponta. O custo por hora (barras, com a ponta destacada) e o total recalculam ao vivo.
2. Ajuste a faixa de horário de ponta.
3. Mova o controle **"deslocar X% do consumo da ponta"** e mostre a **economia em R$**.

Fala sugerida: "Aqui o usuário testa uma decisão: se eu passar a usar a máquina de lavar fora do horário de ponta, quanto economizo?"

Esses cálculos usam só a previsão já na tela, então respondem instantaneamente, sem rodar o modelo de novo.

### Passo 4: Alertas de pico e desperdício com o `05_picos_e_desperdicio_30dias.csv` (2 min)

1. Carregue o arquivo 05. Ele tem, a cada 4 dias, um pico forte entre 18h e 21h (simulando chuveiro + forno + ar-condicionado), e um **consumo fantasma** constante, inclusive de madrugada.
2. Navegue até um dia de pico. Arraste a **linha de limite (kW)** no gráfico: a lista de horas acima do limite e o excedente se atualiza.
3. Revele o real: as **anomalias** aparecem marcadas, com sugestão em texto, por exemplo "Consumo muito acima do esperado às 14h, verifique equipamentos ligados."
4. Aponte o consumo de madrugada: no arquivo 01 ele cai bastante; no 05 permanece alto. É o desperdício (standby).

Fala sugerida: "Compare a madrugada do arquivo 01 com a do 05. A casa dorme, mas o consumo não cai: equipamentos ligados à toa."

### Passo 5: Robustez (2 min)

Mostre que o sistema não quebra com dados imperfeitos:

- **`04_com_lacunas_28dias.csv`**: o resumo informa quantas horas foram preenchidas. Lacunas curtas são interpoladas; as longas (a de 2 dias) usam a mesma hora da semana anterior.
- **`02_so_consumo_21dias.csv`**: só tem a coluna de consumo. O sistema aceita e mostra o aviso de que as demais medições foram **estimadas a partir do treino**.
- **`03_formato_uci_por_minuto_14dias.csv`**: formato bruto original (separador `;`, datas `dd/mm/aaaa`, `?` nas falhas, uma linha por minuto). É agregado por hora automaticamente.
- **`06_invalido_historico_curto_5dias.csv`**: tem só 120 h, menos que as 168 h necessárias. Deve ser **recusado com mensagem clara em português**. Mostrar um erro bem tratado vale ponto.

### Passo 6: Tabela e desempenho do modelo (1 min)

1. **Tabela de valores previstos:** hora, previsto, baseline, real, erro e custo. Ordene uma coluna e clique em exportar CSV.
2. **Desempenho do modelo:** métricas de teste dos três modelos, erro por horizonte e curvas de treino. Aqui entram os **números reais** do trabalho (seção abaixo).

---

## 4. Números reais para citar (conjunto de teste da UCI)

De `models/metrics.json`, em dados que nenhum modelo viu no treino:

| Modelo | MAE (kW) | RMSE (kW) | WAPE | R² |
|---|---|---|---|---|
| Baseline (dia anterior) | 0,497 | 0,744 | 52,6% | −0,099 |
| GRU | 0,434 | 0,594 | 45,9% | 0,300 |
| **LSTM** | **0,426** | **0,592** | **45,1%** | **0,305** |

Cite sempre a fonte: o conjunto de teste real, não os CSVs sintéticos.

---

## 5. Perguntas prováveis da banca

**"Por que o erro ainda é alto (WAPE ~45%)?"**
O consumo de uma casa só é muito ruidoso: depende de hábitos, visitas, viagens. O R² de ~0,30 mostra que a rede captura a rotina, mas não o imprevisível. O ganho sobre o baseline é real, porém moderado, e o trabalho reporta isso sem maquiar.

**"O modelo acerta os picos?"**
Não bem. Ele os **subestima**: nas horas acima do percentil 90 (~1,92 kW) o viés do LSTM é cerca de −1,0 kW. É típico de treinar com MSE em dados ruidosos. Por isso o recurso de alertas permite ao usuário definir seu próprio limite e comparar com o real.

**"Os CSVs de exemplo provam que o modelo funciona?"**
Não, e dizemos isso de forma explícita: são sintéticos, só para demonstrar a interface e o tratamento de entrada. A validação está no conjunto de teste real.

**"Por que LSTM e GRU, e por que 168 horas?"**
São redes feitas para sequências; 168 h cobre uma semana inteira, capturando a rotina diária e a semanal. Veja [REDE_NEURAL.md](REDE_NEURAL.md).

**"E se o CSV tiver colunas faltando ou lacunas?"**
Mostrado ao vivo nos arquivos 02 e 04.

**"Por que o modelo prevê as 24 h de uma vez?"**
Previsão multi-passo direta evita acumular erro, que acontece quando a previsão de uma hora vira entrada da próxima.

---

## 6. Plano B (se algo falhar)

| Problema | O que fazer |
|---|---|
| Página mostra "Modelos não treinados" | Rodar `python -m src.train` (LSTM ~2 min, GRU ~8 min em CPU) antes da apresentação; não dá para resolver ao vivo |
| Upload recusado inesperadamente | Mostre a mensagem de erro (é parte da demo), e use `Usar dados de exemplo` |
| Servidor reiniciou e a tela ficou vazia | A série fica em memória; recarregue o CSV |
| Sem internet | Plotly.js vem de CDN, então os gráficos **não carregam sem internet**. Teste a rede do local ou use o celular como roteador |
| Tudo falhou | Use os prints em `reports/figures/` (`dashboard_completo.png` etc.), o PDF em `reports/apresentacao/` ou o vídeo em `reports/video/` |

---

## 7. Checklist final

- [ ] Servidor no ar e página carregando
- [ ] Internet testada (CDN do Plotly)
- [ ] Pasta `exemplos/` aberta, arquivos 01 a 06 visíveis
- [ ] Fluxo ensaiado uma vez, na ordem 01 → 05 → 04 → 02 → 03 → 06
- [ ] Números da seção 4 decorados (LSTM vence o baseline; R² ~0,30; picos subestimados)
- [ ] Plano B (prints, PDF e vídeo) à mão
