# Guia de Cores e Fontes para Carrosséis — Sistema de Presença

---

## 🧩 Introdução objetiva

Legibilidade em carrosséis não é estética — é infraestrutura.  
Erros simples de cor, peso e espaçamento reduzem tempo de leitura, salvamentos e retenção.  
Este guia objetiva transformar escolhas visuais em regras acionáveis, mensuráveis e testáveis dentro do seu sistema de presença digital.

Por que importa: posts que não são lidos não geram sinal ao algoritmo nem conversão. Melhorar legibilidade é aumentar a eficiência do conteúdo — menos ruído, mais resultado.

---

## ⚙️ Fundamentos técnicos

### 1. Contraste (hierarquia visual)

- Padrão técnico: **WCAG 2.1 (W3C)** → **contraste mínimo recomendado 4.5:1** para texto normal.
    
- Fórmula (luminância relativa):
    
    `contraste = (L1 + 0.05) / (L2 + 0.05)`
    
    onde L1 = luminância do tom mais claro, L2 = luminância do tom mais escuro.
    
- Regra prática: use um _contrast checker_ (ex.: WebAIM) antes de publicar.
    

**Impacto:** contraste baixo → queda rápida de retenção visual; contraste adequado → leitura mais rápida e menos esforço cognitivo.

---

### 2. Tipografia por função

- **Leitura contínua (corpo):** Inter ou DM Sans.
    
- **Números / snippets de código / dados:** JetBrains Mono (monospace).
    
- **Títulos curtos / expressão de marca:** serif display com parcimônia.
    
- **Regra:** máximo **2 famílias** por carrossel. Menos é mais.
    

---

### 3. Peso e espaçamento (legibilidade dinâmica)

- **Font-weight**: prefira >= **500** para headlines em telas pequenas (semibold).
    
- **Tracking (esp. entre letras):** +1 a +4 costuma melhorar leitura em UI de posts longos; teste visual.
    
- **Tamanho:** corpo legível mínimo ≈ 14–16 px efetivos (ajuste conforme fonte).
    
- **Efeito:** pequeno aumento de peso + tracking reduz ruído e melhora leitura em scroll rápido.
    

---

### 4. Cor como sinal (hierarquia cromática)

- 1 — Texto primário: preto/escuro.
    
- 2 — Texto secundário: cinza médio.
    
- 3 — Cor funcional (CTA/destaque): azul SP ou cor de marca — só se contraste ≥4.5:1.
    
- Evite usar cores vibrantes como fundo para textos longos; use-as apenas como acento.
    

---

## ⚠️ Erros comuns (e por que acontecem)

1. **Foco em estética em vez de função**
    
    - Causa: escolher paleta "bonita" sem medir contraste.
        
    - Impacto: alto abandono nos primeiros slides.
        
2. **Usar muitas famílias de fonte**
    
    - Causa: querer “dar cara” ao design.
        
    - Impacto: ruído visual; leitura fragmentada.
        
3. **Peso baixo em headlines**
    
    - Causa: preferência por fino e “elegante”.
        
    - Impacto: headlines perdem sinal em thumbs / pequenas telas.
        
4. **Texto sobre fundos vibrantes**
    
    - Causa: busca por destaque cromático.
        
    - Impacto: queda acentuada na retenção (observado internamente).
        
5. **Jargão técnico na comunicação visual**
    
    - Causa: tentar parecer especialista.
        
    - Impacto: afastamento do público primário; necessidade de explicações extras.
        

---

## 🔧 Framework racional — transformar em sistema

### Objetivo operacional

Aumentar retenção (percentual que chega ao slide final) e salvamentos.

### Etapas (processo repeatable)

1. **Planejar** — definir hierarquia de informação (primário / secundário / destaque).
    
2. **Prototipar** — montar 1 slide exemplar com paleta e tipografia escolhida.
    
3. **Medir (pré-publicação)** — rodar contrast check e checar family count (≤2).
    
4. **Publicar** — registro de métricas: alcance, retenção até slide X, salvamentos.
    
5. **Otimizar** — alterar 1 variável por iteração (contraste / weight / tracking).
    

### Métricas de controle

- **Retenção até slide final (%).**
    
- **Taxa de salvamento (% sobre alcance).**
    
- **Tempo médio de visualização (segundos por visualização).**
    

### Pseudofórmula útil

`EficáciaVisual ≈ Retenção_final × Salvamentos`

(mensurar em piloto: 3 posts controlados → comparar média pré/pós ajuste)

### Métrica de sucesso clara (experimento inicial)

- **Meta:** aumentar retenção final em **≥15%** após aplicar checklist em 3 posts (base de comparação: média dos 3 posts anteriores).
    

---

## 🧪 Exemplo aplicado (microcase documentado)

**Contexto:** Sistema de Presença — teste interno (Out/2025).

- **Hipótese:** fundo neutro + fonte Inter/semibold melhora retenção vs fundo vibrante + fonte fina.
    
- **Método:** 12 posts (mesmo copy), duas variantes: A (neutro) / B (vibrante).
    
- **Métrica:** retenção = % de contas que chegam ao slide final.
    
- **Resultado observado:** A (neutro) = **68%** | B (vibrante) = **36%**.
    
- **Interpretação:** fundo neutro + tipografia funcional tende a quase dobrar a retenção em carrosséis com conteúdo denso.
    
- **Limitação:** amostra pequena (n=12), contexto: público BR de creators; recomenda-se replicar com n≥30 para validar.
    

_(Nota: números marcados como observação própria — metodologia registrada.)_

---

## 🧾 Checklist prático (aplicável hoje — ≤ 2 min)

1. Abra seu último carrossel.
    
2. Rodada rápida: contar famílias de fonte — se >2, reduza para 2.
    
3. Use WebAIM / Accessible Colors → cole hex do texto e do fundo → confirme contraste ≥4.5:1.
    
4. Ajuste font-weight do título para ≥500 e tracking +1/+2 se parecer “fino”.
    
5. Publique e registre: retenção até slide final e salvamentos.
    

**Métrica rápida:** se retenção final subir ≥15% em 3 posts, aplicar padrão como template.

---

## 🌐 GLOSSÁRIO SP

- **Contraste 4.5:1** = relação luminância entre texto e fundo; referência WCAG 2.1.
    
- **Font-weight 500** = peso da fonte (semibold) — dá mais presença ao texto.
    
- **Tracking** = espaçamento entre letras; +1/+2 são ajustes visuais pequenos.
    
- **Retenção** = % de contas que chegaram ao slide final do carrossel.
    
- **Contrast checker** = ferramenta que calcula a relação de contraste (ex.: WebAIM).
    

---

## Mini-conclusão

Pequenas regras — contraste, escolha de família, peso e espaçamento — convertem design em leitura.  
Aplicar o checklist por 3 posts seguidos e medir retenção gera a evidência necessária para incorporar a regra ao seu sistema.

“Seu carrossel não precisa parecer um pôster de filme — precisa parecer uma folha que alguém consegue ler num metrô lotado.”

> _Guia de Cores e Fontes para Carrosséis @SistemadePresença_