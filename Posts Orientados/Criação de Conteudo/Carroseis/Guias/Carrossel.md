# Guia de Carrossel — Sistema de Presença

---

## 🧩 Introdução objetiva

Carrossel não é só design.  
É um circuito de atenção: cada card tem função. Sem transição cognitiva entre esses pontos, o leitor sai — e o post “morre” cedo.  
Este guia documenta o ajuste prático que testamos: usar o **slide 3 como ponte de contexto** para reduzir quedas entre empatia e entrega de valor. Objetivo: clareza operacional, não motivação.

---

## ⚙️ Fundamentos técnicos

**Princípio central**  
A atenção é sequencial e acumulativa. O leitor decide, slide a slide, se vale a pena continuar. Se a transição entre “me importo” (empatia) e “por que isso importa” (contexto) falha, a curva de retenção cai abruptamente.

**Métrica chave**

- **Retenção de cards** = % de pessoas que chegam ao último card.  
    (Usamos essa métrica porque é direta e mensurável nos Insights de post.)
    

**Relações úteis (heurísticas)**

- `Qualidade do conteúdo ≈ Retenção × (Saves_weighted + Comments_weighted)`  
    (heurística para priorizar retenção sobre curtidas; pesos dependem do objetivo)
    
- `Engajamento útil = Comentários × Tempo médio de visualização`  
    (comentários mostram intenção, tempo mostra profundidade)
    

**Benchmark interno (transparente)**

- Observação própria — base: 12 carrosséis (Out/2025).
    
    - Sem slide 3 de contexto → retenção média ≈ **41%**.
        
    - Com slide 3 de contexto → retenção média ≈ **68%**.
        
    - Nota: amostra pequena; tendência consistente. Próximo passo: validar com n>30.
        

---

## ⚠️ Erros comuns (e impacto)

1. **Pular a ponte (Empatia → Valor direto)**
    
    - _Causa:_ pressa para "entregar".
        
    - _Impacto:_ queda imediata na retenção (observado ~-38% em média).
        
2. **Slide 3 denso ou confuso**
    
    - _Causa:_ tentativa de explicar tudo num card.
        
    - _Impacto:_ leitor não processa a transição e abandona.
        
3. **Capa chamativa mas sem promessa cognitiva**
    
    - _Causa:_ foco só em design/curiosidade.
        
    - _Impacto:_ muitas visitas iniciais, pouca retenção e poucos salvamentos.
        
4. **Medir pelo sinal errado (curtidas como proxy de qualidade)**
    
    - _Causa:_ confusão entre atenção rápida e utilidade.
        
    - _Impacto:_ decisões de otimização equivocadas; perseguir likes ao invés de retenção.
        
5. **Não ensinar como medir**
    
    - _Causa:_ pressupor que todo mundo sabe usar Insights.
        
    - _Impacto:_ usuários não replicam o teste nem validam melhorias.
        

---

## 🔧 Framework racional — “PONTE” (5 passos)

**Objetivo:** transformar um carrossel de 5–7 cards em um circuito de atenção mensurável.

### Passos

1. **Capa (Slide 1)** — 1 linha empática + 1 afirmação técnica curta.
    
    - Ex.: “Já percebeu que seu carrossel começa bem e morre cedo? 90% dos drops no slide 3.”
        
2. **Empatia (Slide 2)** — validar dor / problema do público.
    
    - Ex.: “Você para aqui porque...”
        
3. **Ponte de contexto (Slide 3)** — **1 frase** que explica _por que_ continuar: finalidade + transição.
    
    - Regras: 1 frase | 1 conexão lógica | 1 mini-indicador (ex.: número, exemplo curto).
        
4. **Valor (Slides 4–5)** — exemplo prático / dado / checklist aplicável.
    
    - Um insight por card.
        
5. **Encerramento + CTA (último slide)** — síntese + micro-ação mensurável e de baixo risco.
    
    - Ex.: “Salve e compare retenção entre dois carrosséis.”
        

### Métricas de controle (o que monitorar)

- `Retenção (%)` — principal.
    
- `Salvamentos` — sinal de utilidade.
    
- `Comentários qualitativos` — intenção/curiosidade.
    
- `Curtidas` — sinal secundário (vanity).
    
- `Tempo médio de visualização` (se disponível).
    

### Pseudofórmula de sucesso

`Score_Carrossel = Retencao * (0.6) + (Saves_norm * 0.3) + (Comments_norm * 0.1)`

(Score usado internamente para priorizar; normalize cada métrica entre 0–1)

---

## 🧪 Exemplo aplicado (passo a passo com números hipotéticos)

**Contexto:** você tem dois carrosséis recentes:

- Post A (sem slide 3 contextual): retenção = 38%, saves = 12, comments = 6
    
- Post B (com slide 3 contextual): retenção = 62%, saves = 26, comments = 12
    

**Interpretação:** Post B tem ganho claro em retenção (+24pp) e mais que dobrou salvamentos. Isso indica maior utilidade e maior probabilidade de alcance sustentável (algoritmo cria preferência por conteúdo que mantém atenção).

**Como documentar (método):**

1. Antes do teste: registrar baseline (retention %, saves, comments) — data, horário, tipo de conteúdo.
    
2. Aplicar ajuste (transformar slide 3).
    
3. Publicar e coletar dados 72h e 7 dias.
    
4. Comparar variação absoluta e relativa.
    
5. Decidir: adotar padrão, ajustar microcopy, testar com outro tema.
    

**Versão leve (humor técnico inteligente opcional):**

> "Se seu carrossel fosse um funil, o slide 3 é a válvula de retenção — não feche a válvula com excesso de texto."

---

## 🧾 Checklist prático (aplicar hoje — ≤ 10 minutos)

-  Abra seu último carrossel e anote `Retenção (%)`, `Saves`, `Comments`.
    
-  Escreva **1** frase que explica por que o leitor deve continuar (para o slide 3). Tempo: 2 minutos.
    
-  Substitua o slide 3 pelo novo texto (1 frase) e publique.
    
-  Compare retenção depois de 72h e 7 dias.
    
-  Se retenção subir ≥15pp, padronize o formato (documente no banco de templates).
    

---

## 🌐 GLOSSÁRIO SP

- **Retenção (de cards)** = % de usuários que chegam ao último card do carrossel.
    
- **Slide de contexto** = card que conecta empatia ao valor com uma transição clara (1 frase).
    
- **Salvamento (save)** = ação do usuário de guardar o post; indicador de utilidade.
    
- **Curtida (like)** = sinal de atenção rápida; não substitui retenção.
    
- **Micro-ação** = tarefa executável em ≤2 minutos que gera evidência mensurável.
    

---

## Mini-conclusão

Carrossel é engenharia de atenção. O ajuste mais eficaz que testamos: transformar o **slide 3** em uma ponte de contexto clara (1 frase), reduzindo quedas entre empatia e entrega de valor. 

Meça pelo que importa: **retenção**, não é só curtidas.

Guia de Carrossel — @SistemadePresenca