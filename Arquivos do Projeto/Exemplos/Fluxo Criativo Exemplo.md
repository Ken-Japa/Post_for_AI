# Introdução
Esse documento mostra a lógica geral de construção do produto do Fluxo Criativo e fornece um exemplo de resultado.Servirá como uma referência base do produto que temos pront e referência.
Nota: o produto já conta com landing page com CTAs, formulário pronto automatiado (ao ser preenchido gera o json de input, cadastra cliente no google sheets, envia um email de notificação).

# Lógica de construção
Através de um formulário ou entrevista, prenche-se o json de input. Esse json serve para o script conseguir enviar o prompt para a IA (o prompt tem uma prop importante que é o tipo de campanha que ajudar a personalizar a mensagem). Existem duas formas de fluxo: enviar para uma IA individual ou uma mais completa: envia a Gemini e Cohere (com objetivo de angariar diferentes ideias de posts) e depois envia um resumo dessas duas resposta para a Mistral que escolhe as melhores ideias e elabora sua resposta. Depois de um desses dois processos (que eu posso escolher manualmente) gero o PDF e HTML que é um resultado 95% feito (posso revisar manualmente ao final), praticamente pronto para enviar ao cliente. 
Nota: o exemplo do resultado é o resultado bruto sem edição humana.
## Json de entrada:
O formato do json de entrada é o seguinte:

"{

  "nome_do_cliente": "",

  "subnicho": "",

  "informacoes_de_contato": "",

  "publico_alvo": "",

  "tom_de_voz": "",

  "estilo_de_comunicacao": "",

  "vocabulario_da_marca": [""],

  "exemplos_de_nicho": [""],

  "objetivos_de_marketing": "",

  "canais_de_distribuicao": [""],

  "topicos_principais": [""],

  "palavras_chave": [""],

  "chamada_para_acao": "",

  "restricoes_e_diretrizes": "",

  "informacoes_adicionais": "",

  "referencias_de_concorrentes": [""],

  "referencias_de_estilo_e_formato": [""],

  "tipo_de_conteudo": "",

  "tipo_de_campanha": "",

  "posts_anteriores": [""],

  "conteudos_semanais": [

    {

      "chamada_para_acao_individual": "",

      "objetivo_do_conteudo_individual": ""

    }

  ]

}"

## Construção de prompt
Construo o prompt para envio para IA com esse script:
"import json

from .campaign_narrative_generator import generate_campaign_narrative

from ...metric_suggester import suggest_metrics

  

def build_prompt(client_profile: dict, niche_guidelines: dict, content_type: str, weekly_themes: list[str], weekly_goal: str, campaign_type: str, strategic_analysis: dict = None) -> str:

    """

    Constrói o prompt completo para a API do Gemini, combinando o perfil do cliente, diretrizes de nicho, contexto semanal e instruções de formato.

  

    Args:

        client_profile (dict): Dicionário contendo o perfil do cliente (subnicho, tom_de_voz, publico_alvo, objetivos_gerais).

        niche_guidelines (dict): Dicionário contendo diretrizes específicas do nicho.

        content_type (str): O tipo de conteúdo a ser gerado (ex: 'instagram_post').

        weekly_themes (list[str]): Uma lista de temas a serem abordados na semana.

        weekly_goal (str): O objetivo principal do conteúdo para a semana.

            campaign_type (str): O tipo de campanha (ex: 'autoridade').

            strategic_analysis (dict): O resultado da análise estratégica do briefing.

  

    Returns:

        str: O prompt completo formatado para a API do Gemini.

    """

  

    # Coleta de dados do perfil do cliente e diretrizes do nicho, tratando valores vazios

    nome_do_cliente = client_profile.get("nome_do_cliente", "")

    subnicho = client_profile.get("subnicho", "")

    tom_de_voz = client_profile.get("tom_de_voz", "")

    estilo_de_comunicacao = client_profile.get("estilo_de_comunicacao", "")

    vocabulario_da_marca = client_profile.get("vocabulario_da_marca", [])

    publico_alvo = client_profile.get("publico_alvo", "")

    objetivos_de_marketing = client_profile.get("objetivos_de_marketing", "")

    exemplos_de_nicho = client_profile.get("exemplos_de_nicho", [])

    canais_de_distribuicao = client_profile.get("canais_de_distribuicao", [])

    topicos_principais = client_profile.get("topicos_principais", [])

    palavras_chave = client_profile.get("palavras_chave", [])

    chamada_para_acao = client_profile.get("chamada_para_acao", "")

    restricoes_e_diretrizes = client_profile.get("restricoes_e_diretrizes", "")

    informacoes_adicionais = client_profile.get("informacoes_adicionais", "")

    referencias_de_concorrentes = client_profile.get("referencias_de_concorrentes", [])

    referencias_de_estilo_e_formato = client_profile.get("referencias_de_estilo_e_formato", [])

    posts_anteriores = client_profile.get("posts_anteriores", [])

  

    # Sugerir métricas com base no tipo de campanha e objetivos de marketing

    sugerir_metricas = suggest_metrics(campaign_type, objetivos_de_marketing)

  

    # Adicionar diretrizes de nicho, se existirem

    if niche_guidelines:

        for key, value in niche_guidelines.items():

            if isinstance(value, list):

                informacoes_adicionais += f" {key}: {', '.join(value)}."

            else:

                informacoes_adicionais += f" {key}: {value}."

  

    # System Message: Define o papel da IA

    system_message = f"Você é um copywriter especializado em mídias sociais."

    if subnicho:

        system_message += f" O nicho é {subnicho}."

    if niche_guidelines:

        if 'subnicho' in niche_guidelines and niche_guidelines['subnicho']:

            system_message += f" para {niche_guidelines['subnicho']}."

        if 'exemplos_de_nicho' in niche_guidelines and niche_guidelines['exemplos_de_nicho']:

            exemplos_str = ', '.join(niche_guidelines['exemplos_de_nicho'])

            system_message += f" Atuando sob as diretrizes de nicho: {exemplos_str}."

    system_message += f" Compreenda as nuances e particularidades deste segmento para gerar conteúdo.Seu objetivo é criar conteúdo altamente engajador, viral e relevante para o público-alvo do cliente."

  

    # User Message: Combina todas as informações para criar uma instrução detalhada

    user_message_parts = []

    if canais_de_distribuicao:

        canais_str = ', '.join(canais_de_distribuicao)

        user_message_parts.append(f"Com base no perfil do cliente e nas diretrizes do nicho, crie conteúdo para os seguintes canais: {canais_str}.")

    elif content_type:

        user_message_parts.append(f"Com base no perfil do cliente e nas diretrizes do nicho, crie conteúdo para {content_type}.")

  

    user_message_parts.append("Evite soar como uma IA, busque um estilo de texto humanizado, evite clichês de copy genérica.")

  

    user_message_parts.append("\n**Perfil do Cliente:**")

    if nome_do_cliente:

        user_message_parts.append(f"- Nome do Cliente: {nome_do_cliente}")

    if tom_de_voz:

        user_message_parts.append(f"- Tom de Voz: {tom_de_voz}")

    if estilo_de_comunicacao and estilo_de_comunicacao.strip():

        user_message_parts.append(f"- Busque criar o texto como se fosse: {estilo_de_comunicacao}")

    if vocabulario_da_marca and len(vocabulario_da_marca) > 0:

        user_message_parts.append(f"- O cliente costuma usar o vocabulário e ou frases como: {', '.join(vocabulario_da_marca)}. Busque adaptar seu texto ao estilo do cliente de se comunicar.")

    if publico_alvo:

        user_message_parts.append(f"- Público-Alvo: {publico_alvo}")

  

    # Análise Estratégica Inicial

    user_message_parts.append("\n**Análise Estratégica Inicial:**")

    if referencias_de_concorrentes:

        user_message_parts.append(f"- Análise de Concorrentes/Referências: Considere o estilo e as estratégias de conteúdo dos seguintes concorrentes/referências: {', '.join(referencias_de_concorrentes)}. Identifique oportunidades e diferenciais.")

    if informacoes_adicionais:

        user_message_parts.append(f"- Informações Adicionais e Diretrizes Específicas: {informacoes_adicionais}. Incorpore essas informações para refinar a estratégia de conteúdo.")

  

    user_message_parts.append("\n**Informações Estratégicas do Briefing:**")

  

    if objetivos_de_marketing:

        user_message_parts.append(f"- Objetivos de Marketing: {objetivos_de_marketing}")

    if canais_de_distribuicao:

        user_message_parts.append(f"- Canais de Distribuição: {', '.join(canais_de_distribuicao)}")

    if topicos_principais:

        user_message_parts.append(f"- Tópicos Principais: {', '.join(topicos_principais)}")

    if palavras_chave:

        user_message_parts.append(f"- Palavras-Chave: {', '.join(palavras_chave)}")

    if chamada_para_acao:

        user_message_parts.append(f"- Chamada para Ação (CTA): {chamada_para_acao}")

    if restricoes_e_diretrizes:

        user_message_parts.append(f"- Restrições e Diretrizes: {restricoes_e_diretrizes}")

    if posts_anteriores:

        user_message_parts.append("\n**Posts Anteriores:**")

        user_message_parts.append("Considere os seguintes posts já publicados e evite repetir temas ou abordagens de forma idêntica. Busque originalidade e complementariedade.")

        for post in posts_anteriores:

            user_message_parts.append(f"- Tema: {post.get('tema', 'Não especificado')}")

    if referencias_de_concorrentes:

        user_message_parts.append(f"- Concorrentes/Referências: {', '.join(referencias_de_concorrentes)}")

    if referencias_de_estilo_e_formato:

        user_message_parts.append(f"- Referências de Estilo e Formato: {', '.join(referencias_de_estilo_e_formato)}")

  

    if weekly_themes or weekly_goal:

        user_message_parts.append("\n**Contexto Semanal:**")

        if weekly_themes:

            user_message_parts.append(f"- Temas da Semana: {', '.join(weekly_themes)}")

        if weekly_goal:

            user_message_parts.append(f"- Objetivo Semanal: {weekly_goal}")

  

    user_message_parts.append("\n**Instruções de Formato:**")

    user_message_parts.append("Crie uma campanha semanal *coesa e com narrativa progressiva*, composta por 5 ideias de posts. Para cada post, inclua:")

    user_message_parts.append(" Os 5 Posts serão postados na seguinte ordem: sexta, sábado, domingo, segunda e quarta. Baseado no nicho, objetivo e nos dias da semana, retorne também um horário recomendado para postagem.")

    user_message_parts.append(" Busque se possivel inserir stroytelling ou casos ficticios no texto. Busque também inserir dados concretos, números ou estatísticas pertinentes mesmo que sejam aproximados no texto se fizer sentido.")

    user_message_parts.extend(generate_campaign_narrative(campaign_type))

    user_message_parts.append("- `titulo`: Um título conciso para o post.")

    user_message_parts.append("- `tema`: Resumo do tema abordado no post.")

    user_message_parts.append("- `legenda_principal`: A legenda principal do post.")

    user_message_parts.append("- `variacoes_legenda`: Uma lista de 2-3 variações da legenda principal.")

    user_message_parts.append("- `hashtags`: Uma lista de hashtags relevantes.")

    user_message_parts.append("- `sugestao_formato`: Sugestão de formato (ex: \"Carrossel de imagens\", \"Vídeo curto\", \"Infográfico\").")

    user_message_parts.append("- `post_strategy_rationale`: Uma justificativa estratégica detalhada para este post, explicando como ele se encaixa na narrativa semanal e contribui para os objetivos gerais.")

    user_message_parts.append("- `micro_briefing`: Um breve resumo do objetivo e foco principal do post, para contextualização.")

    user_message_parts.append("- `micro_roteiro`: Se `sugestao_formato` for \"Vídeo curto\" ou \"Reel\", inclua um micro-roteiro detalhando cenas, falas/textos e chamadas para ação.")

    user_message_parts.append("- `carrossel_slides`: Se `sugestao_formato` for \"Carrossel de imagens\", inclua uma lista de objetos, onde cada objeto representa um slide do carrossel, contendo `titulo_slide`, `texto_slide` e `sugestao_visual_slide`.")

    user_message_parts.append("- `visual_prompt_suggestion`: Uma descrição detalhada da imagem ou vídeo principal para o post, incluindo estilo, cores, elementos e atmosfera, para ser usada por uma IA de geração de imagens. Evite usar texto no prompt para imagem.")

    user_message_parts.append("- `text_in_image`: Caso tenha alguma sugestão de texto para incluir na imagem. Use esse campo")

    user_message_parts.append("- `visual_description_portuguese`: Uma descrição em português da imagem ou vídeo principal, para ser exibida no briefing.")

    user_message_parts.append("- `cta_individual`: Uma chamada para ação específica para este post.")

    user_message_parts.append("- `interacao`: Sugerir uma pergunta para a audiência ou uma forma de incentivar comentários, aumentando o engajamento além da simples publicação..")

    user_message_parts.append("- `response_script`: Array de objetos com respostas prontas para os comentários mais prováveis. Cada objeto deve conter: `comentario_generico` (Comentário Genérico mais provável), `resposta_sugerida` (resposta sugerida para o comentário mais provável), `comentario_negativo` (Comentário Negativo mais provável), `resposta_negativo` (resposta para o comentario negativo mais provável). Inclua 2 tipos de comentários prováveis.")

    user_message_parts.append("- `ab_test_suggestions`: Uma string para cada post, sugirindo 1-2 testes A/B com formato: 'Elemento a testar (ex: título) → Variação A vs Variação B → Métrica de sucesso'.")

    user_message_parts.append("- `indicador_principal`: A métrica mais importante para este post.")

    user_message_parts.append("- `optimization_triggers`: Retorne uma string com base no `indicador_principal`, defina: 1) Threshold de alerta , 2) Ação corretiva, 3) Threshold de sucesso.")

    user_message_parts.append("\n Gere **APENAS UM JSON VÁLIDO**, sem texto adicional, comentários ou explicações. Não inclua marcações de code block (como ```json). Retorne o conteúdo em formato JSON, com a estrutura abaixo:")

    user_message_parts.append("{{")

    user_message_parts.append("    \"weekly_strategy_summary\": \"Resumo da estratégia semanal para a campanha.\",")

    user_message_parts.append("    \"future_strategy\": {\"proximos_passos\": \"Sugestões gerais para depois da campanha.\", \"posts_nutricao\": [{\"tema\": \"Tema do post de nutrição\", \"formato\": \"Formato do post (ex: Carrossel, Vídeo)\", \"objetivo\": \"Objetivo do post de nutrição\"}], \"remarketing\": [{\"estrategia\": \"Estratégia de remarketing\", \"canal\": \"Canal de remarketing (ex: Email, Anúncios)\"}], \"long_term\": {\"comunidade\": \"Estratégias para comunidade\", \"parcerias\": \"Estratégias de parcerias\"}}, ")

    user_message_parts.append("    \"market_references\": \"Uma lista de 3 objetos JSON, cada um contendo: 'Nome/Handle' (string), 'Diferenciais' (string), 'Oportunidades' (string), 'Posicionamento do Cliente' (string). Exemplo: [{'Nome/Handle': 'PerfilExemplo', 'Diferenciais': 'Conteúdo visual de alta qualidade', 'Oportunidades': 'Maior engajamento em postagens diárias', 'Posicionamento do Cliente': 'Enfatizar expertise técnica'}].\",")

    user_message_parts.append("    \"posts\": [")

    user_message_parts.append("        {{")

    user_message_parts.append("            \"titulo\": \"Título do Post 1\",")

    user_message_parts.append("            \"tema\": \"Resumo do tema abordado no post 1\",")

    user_message_parts.append("            \"legenda_principal\": \"Legenda principal do post 1.\",")

    user_message_parts.append("            \"variacoes_legenda\": [")

    user_message_parts.append("                \"Variação 1 da legenda 1.\",")

    user_message_parts.append("                \"Variação 2 da legenda 1.\"")

    user_message_parts.append("            ],")

    user_message_parts.append("            \"hashtags\": [\"#Hashtag1\", \"#Hashtag2\"],")

    user_message_parts.append("            \"horario_de_postagem\": \"Horário sugerido para postagem\",")

    user_message_parts.append("            \"sugestao_formato\": \"Sugestão de formato para o post (ex: 'Carrossel de imagens', 'Vídeo', 'Reel', 'Imagem única', 'Infográfico'). Pode adicionar explicações.\",")

    user_message_parts.append("            \"carrossel_slides\": [], // Sempre inclua este campo como um array vazio se o formato não for 'Carrossel'.")

    user_message_parts.append("            \"micro_roteiro\": [], // Sempre inclua este campo como um array vazio se o formato não for 'Vídeo' ou 'Reel'.")

    user_message_parts.append("            \"post_strategy_rationale\": \"Justificativa estratégica para este post.\",")

    user_message_parts.append("            \"micro_briefing\": \"Breve resumo do objetivo do post.\",")

    user_message_parts.append("            \"visual_prompt_suggestion\": \"Descrição detalhada para IA de geração de imagens. Evite usar texto no prompt para imagem\",")

    user_message_parts.append("            \"text_in_image\": \"Caso tenha sugestão de texto para ser incluído na imagem. Use esse campo\",")

    user_message_parts.append("            \"visual_description_portuguese\": \"Descrição em português da imagem ou vídeo principal.\",")

    user_message_parts.append("            \"cta_individual\": \"Chamada para ação específica para este post.\",")

    user_message_parts.append("            \"ab_test_suggestions\": \"Uma string com sugestões para testes A/B neste post.\",")

    user_message_parts.append("            \"indicador_principal\": \"A métrica mais importante para este post.\",")

    user_message_parts.append("            \"optimization_triggers\": \"Retorne uma string com base no `indicador_principal`, defina: 1) Threshold de alerta , 2) Ação corretiva, 3) Threshold de sucesso.\",")

    user_message_parts.append("            \"interacao\": \"Formas de como aumentar a interação com este post.\",")

    user_message_parts.append("            \"response_script\": [")

    user_message_parts.append("                {")

    user_message_parts.append("                    \"comentario_generico\": Comentário genérico mais provável para este post")

    user_message_parts.append("                    \"resposta_sugerida\": \"Resposta sugerida para o comentário genérico\",")

    user_message_parts.append("                    \"comentario_negativo\": \"Comentário negativo mais provável para este post\",")

    user_message_parts.append("                    \"resposta_negativo\": \"Resposta para o comentário negativo mais provável\"")

    user_message_parts.append("                },")

    user_message_parts.append("                {")

    user_message_parts.append("                    \"comentario_generico\": Segundo comentário genérico mais provável para este post")

    user_message_parts.append("                    \"resposta_sugerida\": \"Resposta sugerida para o segundo comentário genérico\",")

    user_message_parts.append("                    \"comentario_negativo\": \"Segundo comentário negativo mais provável para este post\",")

    user_message_parts.append("                    \"resposta_negativo\": \"Resposta para o segundo comentário negativo mais provável\"")

    user_message_parts.append("                }")

    user_message_parts.append("            ],")

    user_message_parts.append("            \"micro_roteiro\": [")

    user_message_parts.append("                {")

    user_message_parts.append("                    \"cena\": 1,")

    user_message_parts.append("                    \"descricao\": \"Pessoa sorrindo\",")

    user_message_parts.append("                    \"texto_tela\": \"Problema resolvido!\",")

    user_message_parts.append("                    \"fala\": \"Você sabia que...?\"")

    user_message_parts.append("                }")

    user_message_parts.append("            ],")

    user_message_parts.append("            \"carrossel_slides\": [")

    user_message_parts.append("                {")

    user_message_parts.append("                    \"titulo_slide\": \"Título do Slide 1\",")

    user_message_parts.append("                    \"texto_slide\": \"Texto do slide 1\",")

    user_message_parts.append("                    \"sugestao_visual_slide\": \"Visual para slide 1\"")

    user_message_parts.append("                }")

    user_message_parts.append("            ]")

    user_message_parts.append("        }}")

    user_message_parts.append("    ],")

    user_message_parts.append("    \"metricas_de_sucesso_sugeridas\": " + json.dumps(sugerir_metricas, indent=4, ensure_ascii=False).replace("\n", "\n    ") + ",")

    user_message_parts.append("}}")

  

    user_message = "\n".join(user_message_parts)

  

    return f"{system_message}\n\n{user_message}""

### Função auxiliar construção de prompt
#### Tipo de campanha:
"def generate_campaign_narrative(campaign_type: str) -> list[str]:

    """

    Gera a narrativa da campanha com base no tipo de campanha fornecido.

  

    Args:

        campaign_type (str): O tipo de campanha (e.g., "lancamento", "autoridade").

  

    Returns:

        list[str]: Uma lista de strings representando a narrativa da campanha.

    """

    narratives = {

        "lancamento": [

            "A campanha deve seguir uma narrativa progressiva ao longo dos 5 posts, abordando:",

            "- Post 1: Apresentação do Problema/Desafio do público-alvo.",

            "- Post 2: Introdução da Solução (produto/serviço do cliente).",

            "- Post 3: Detalhamento dos Benefícios e Vantagens da solução.",

            "- Post 4: Prova Social/Exemplos de Sucesso/Depoimentos.",

            "- Post 5: Chamada para Ação (CTA) clara e direta.",

        ],

        "autoridade": [

            "A campanha deve seguir uma narrativa progressiva ao longo dos 5 posts, abordando:",

            "- Post 1: Dica ou Insight da Semana: Conteúdo educativo com valor imediato.",

            "- Post 2: Bastidores: Mostrar como a solução é construída (processo, equipe, metodologia).",

            "- Post 3: Estudo de Caso: Provar resultados ou demonstrar expertise com um exemplo real.",

            "- Post 4: FAQ / Quebra de Objeções: Responder dúvidas comuns e reforçar autoridade.",

            "- Post 5: Evento ou CTA Premium: Convidar para um webinar, e-book, consultoria, etc.",

        ],

        "engajamento": [

            "A campanha deve seguir uma narrativa progressiva ao longo dos 5 posts, abordando:",

            "- Post 1: Enquete ou Pergunta: Criar uma conversa com o público.",

            "- Post 2: Peça de Opinião: Pedir que comentem ideias, dores ou experiências.",

            "- Post 3: Desafio ou Participação: Propor que executem algo (ex: “Desafio 7 dias”).",

            "- Post 4: Live ou Sessão de Q&A: Interação direta em tempo real.",

            "- Post 5: Resumo/Compilado: Mostrar as melhores respostas, resultados ou insights.",

        ],

        "conversao": [

            "A campanha deve seguir uma narrativa progressiva ao longo dos 5 posts, abordando:",

            "- Post 1: Atenção / Gancho Forte: Chamar atenção com um insight, dado ou pergunta direta.",

            "- Post 2: Problema: Reforçar a dor ou oportunidade perdida.",

            "- Post 3: Oferta: Apresentar a solução paga e seus diferenciais.",

            "- Post 4: Urgência / Escassez: Mostrar prazo, bônus ou vantagens limitadas.",

            "- Post 5: CTA Final: Levar diretamente ao cadastro, compra ou formulário.",

        ],

        "retencao": [

            "A campanha deve seguir uma narrativa progressiva ao longo dos 5 posts, abordando:",

            "- Post 1: Agradecimento: Mostrar reconhecimento ao usuário/cliente atual.",

            "- Post 2: Atualizações: Apresentar novidades e melhorias.",

            "- Post 3: Conteúdo Exclusivo: Dicas avançadas, tutoriais, bônus.",

            "- Post 4: Feedback: Pedir opiniões e sugestões.",

            "- Post 5: Comunidade: Convidar para grupo fechado, evento ou programa VIP.",

        ],

        "criativa": [

            "A lógica aqui não é seguir um arco narrativo de 5 posts conectados, mas sim gerar ideias independentes, originais e variadas, pensadas no estilo do criador.",

            "Lista de ideias → Formato → Objetivo → Estrutura resumida.",

        ],

    }

    return narratives.get(campaign_type.lower(), narratives["lancamento"])"
#### Auxiliar para métricas
"import json

import os

  

def suggest_metrics(campaign_type: str, objetivos_de_marketing: str) -> dict:

    """

    Sugere métricas de sucesso com base no tipo de campanha e objetivos de marketing.

  

    Args:

        campaign_type (str): O tipo de campanha (ex: "lancamento", "autoridade", "engajamento").

        objetivos_de_marketing (str): Uma descrição dos objetivos de marketing da campanha.

  

    Returns:

        dict: Um dicionário contendo as métricas de sucesso sugeridas, formatado para inclusão no JSON final.

    """

    script_dir = os.path.dirname(__file__)

    metricas_map_path = os.path.join(script_dir, "metricas_map.json")

  

    try:

        with open(metricas_map_path, 'r', encoding='utf-8') as f:

            metricas_map = json.load(f)

    except FileNotFoundError:

        print(f"Erro: O arquivo {metricas_map_path} não foi encontrado.")

        return {}

    except json.JSONDecodeError:

        print(f"Erro: O arquivo {metricas_map_path} não é um JSON válido.")

        return {}

  

    suggested_metrics = {

        "objetivo_principal": objetivos_de_marketing, # Usando a descrição completa como objetivo principal

        "indicadores_chave": [],

        "metricas_secundarias": []

    }

  

    if campaign_type in metricas_map:

        campaign_data = metricas_map[campaign_type]

        suggested_metrics["indicadores_chave"] = campaign_data.get("metricas_principais", [])

        suggested_metrics["metricas_secundarias"] = campaign_data.get("metricas_secundarias", [])

    else:

        print(f"Aviso: Tipo de campanha '{campaign_type}' não encontrado no metricas_map.json.")

  

    return suggested_metrics

  

if __name__ == '__main__':

    # Exemplo de uso

    campaign_type_example = "lancamento"

    objetivos_de_marketing_example = "Lançar a plataforma e gerar cadastros no trial gratuito"

    metrics = suggest_metrics(campaign_type_example, objetivos_de_marketing_example)

    print(json.dumps(metrics, indent=2, ensure_ascii=False))

  

    campaign_type_example = "autoridade"

    objetivos_de_marketing_example = "Construir confiança e reputação no mercado"

    metrics = suggest_metrics(campaign_type_example, objetivos_de_marketing_example)

    print(json.dumps(metrics, indent=2, ensure_ascii=False))

  

    campaign_type_example = "nao_existe"

    objetivos_de_marketing_example = "Um objetivo qualquer"

    metrics = suggest_metrics(campaign_type_example, objetivos_de_marketing_example)

    print(json.dumps(metrics, indent=2, ensure_ascii=False))"

##### Json de métrica de mapeamento

"{

  "lancamento": {

    "objetivo_principal": "Gerar atenção e cadastros/testes do produto",

    "metricas_principais": [

      "Taxa de cliques no link da bio (CTR)",

      "Número de conversões (cadastros, downloads, trials)",

      "Taxa de conversão da landing page (%)"

    ],

    "metricas_secundarias": [

      "Impressões totais",

      "Alcance dos posts",

      "Visualizações de vídeo (especialmente Post 2 ou 4)",

      "Engajamento no CTA final (comentários, cliques no botão)"

    ],

    "interpretacao": "Quanto maior o CTR e as conversões, melhor a eficiência da narrativa de lançamento."

  },

  

  "autoridade": {

    "objetivo_principal": "Construir confiança, reputação e percepção de expertise",

    "metricas_principais": [

      "Compartilhamentos por post",

      "Salvamentos por post",

      "Tempo médio de leitura (em artigos ou blog)",

      "Cliques em conteúdos educativos (e-book, whitepaper)"

    ],

    "metricas_secundarias": [

      "Novos seguidores orgânicos",

      "Número de comentários com dúvidas ou perguntas",

      "Menções externas ou backlinks"

    ],

    "interpretacao": "Altos compartilhamentos e salvamentos indicam que o conteúdo é visto como valioso e confiável."

  },

  

  "engajamento": {

    "objetivo_principal": "Aumentar interação e fortalecer a comunidade",

    "metricas_principais": [

      "Taxa de engajamento (%) [(likes + comentários + compartilhamentos) / alcance]",

      "Número de comentários por post",

      "Respostas a stories ou enquetes",

      "Participações em desafios ou interações diretas"

    ],

    "metricas_secundarias": [

      "Cliques em stickers",

      "Visualizações de stories ou lives",

      "Número de conteúdos gerados pelo usuário (UGC)"

    ],

    "interpretacao": "Engajamento alto mostra que o público está ativo e emocionalmente envolvido."

  },

  

  "conversao": {

    "objetivo_principal": "Gerar vendas ou leads qualificados",

    "metricas_principais": [

      "Taxa de conversão (%)",

      "Valor total de vendas ou receita gerada",

      "Custo por lead (CPL)",

      "Retorno sobre investimento (ROI)"

    ],

    "metricas_secundarias": [

      "CTR em botões de compra",

      "Taxa de abandono no checkout",

      "Tempo médio até a conversão"

    ],

    "interpretacao": "Se as conversões não crescem, revise a clareza da oferta e o alinhamento com o público."

  },

  

  "retencao": {

    "objetivo_principal": "Aumentar retenção e engajamento de usuários atuais",

    "metricas_principais": [

      "Taxa de reengajamento (usuários ativos novamente)",

      "Taxa de churn (cancelamentos)",

      "Participação em programas de fidelidade ou comunidade"

    ],

    "metricas_secundarias": [

      "Taxa de abertura de e-mails de clientes",

      "Cliques em atualizações de produto",

      "Feedbacks positivos recebidos"

    ],

    "interpretacao": "Altas taxas de reengajamento indicam que os usuários continuam encontrando valor no produto."

  },

  

  "criativa": {

    "objetivo_principal": "Explorar formatos, crescer base e testar ideias",

    "metricas_principais": [

      "Crescimento de seguidores por semana",

      "Taxa de engajamento média",

      "Alcance orgânico por post",

      "Taxa de compartilhamento"

    ],

    "metricas_secundarias": [

      "Tempo médio de visualização em Reels",

      "CTR em experimentos de conteúdo",

      "Feedback qualitativo nos comentários"

    ],

    "interpretacao": "Essas métricas ajudam a identificar o que mais ressoa com o público e guiam futuras estratégias de conteúdo."

  }

}"

# Exemplos do resultado

## Json utilizado para gerar o resultado

"{

  "nome_do_cliente": "Fluxo Criativo",

  "subnicho": "Plataforma de geração de conteúdo estratégico para redes sociais",

  "informacoes_de_contato": "kiira.shinta@gmail.com",

  "publico_alvo": "Solopreneurs BR (coaches, consultores, etc.). Foco em autoridade vs lançamento.",

  "tom_de_voz": "inovador, didático e inspirador",

  "estilo_de_comunicacao": "uma conversa entre um consultor de marketing e um empreendedor/influencer curioso, explicando com clareza e empolgação como o serviço pode otimizar sua presença digital e trazer oportunidades para seu negócio.",

  "vocabulario_da_marca": [

    "automação criativa",

    "conteúdo estratégico",

    "facilidade",

    "escala de resultados",

    "posts personalizados",

    "criação inteligente"

  ],

  "exemplos_de_nicho": [

    "aplicativos de automação de marketing",

    "ferramentas para social media"

  ],

  "objetivos_de_marketing": "gerar confiança na ferramenta e atrair usuários para experimentarem o serviço.",

  "canais_de_distribuicao": [

    "Instagram",

    "LinkedIn",

    "Facebook",

    "Landing page"

  ],

  "topicos_principais": [

    "como acelerar criação de conteúdo",

    "dicas de marketing digital",

    "casos reais de transformação com automação",

    "tendências em marketing"

  ],

  "palavras_chave": [

    "marketing automatizado",

    "ferramenta de social media"

  ],

  "chamada_para_acao": "Experimente o Fluxo Criativo e descubra como gerar conteúdo de alto impacto em minutos!",

  "restricoes_e_diretrizes": "Evitar jargões técnicos de programação; manter o foco no benefício real para o usuário; não prometer resultados irreais; comunicar com clareza e empatia.",

  "informacoes_adicionais": "O Fluxo Criativo é uma plataforma de geração de conteúdo inteligente que combina estratégia de marketing com tecnologia de ponta. Permite criar postagens, legendas e campanhas completas com base em um briefing simples, gerando ideias originais e alinhadas à voz da marca.",

  "referencias_de_concorrentes": [

    "@copy.ai",

    "@jasperai",

    "Buffer",

    "Hootsuite"

  ],

  "referencias_de_estilo_e_formato": [

    "carrosséis educativos sobre IA e marketing",

    "Instagram",

    "Linkedin"

  ],

  "tipo_de_conteudo": "conteúdo educativo e institucional com foco em conversão e awareness",

  "tipo_de_campanha": "conversao",

  "conteudos_semanais": [

    {

      "chamada_para_acao_individual": "",

      "objetivo_do_conteudo_individual": ""

    }

  ]

}"


## Prompt enviado para IA com base no json

"
--- Prompt Log (content_generation) ---
Cliente: Fluxo Criativo
Data/Hora: 2025-10-17 18:38:44
-------------------------------------

Você é um copywriter especializado em mídias sociais. O nicho é Plataforma de geração de conteúdo estratégico para redes sociais. para Plataforma de geração de conteúdo estratégico para redes sociais. Atuando sob as diretrizes de nicho: aplicativos de automação de marketing, ferramentas para social media. Compreenda as nuances e particularidades deste segmento para gerar conteúdo.Seu objetivo é criar conteúdo altamente engajador, viral e relevante para o público-alvo do cliente.

Com base no perfil do cliente e nas diretrizes do nicho, crie conteúdo para os seguintes canais: Instagram, LinkedIn, Facebook, Landing page.
Evite soar como uma IA, busque um estilo de texto humanizado, evite clichês de copy genérica.

**Perfil do Cliente:**
- Nome do Cliente: Fluxo Criativo
- Tom de Voz: inovador, didático e inspirador
- Busque criar o texto como se fosse: uma conversa entre um consultor de marketing e um empreendedor/influencer curioso, explicando com clareza e empolgação como o serviço pode otimizar sua presença digital e trazer oportunidades para seu negócio.
- O cliente costuma usar o vocabulário e ou frases como: automação criativa, conteúdo estratégico, facilidade, escala de resultados, posts personalizados, criação inteligente. Busque adaptar seu texto ao estilo do cliente de se comunicar.
- Público-Alvo: Solopreneurs BR (coaches, consultores, etc.). Foco em autoridade vs lançamento.

**Análise Estratégica Inicial:**
- Informações Adicionais e Diretrizes Específicas: O Fluxo Criativo é uma plataforma de geração de conteúdo inteligente que combina estratégia de marketing com tecnologia de ponta. Permite criar postagens, legendas e campanhas completas com base em um briefing simples, gerando ideias originais e alinhadas à voz da marca. subnicho: Plataforma de geração de conteúdo estratégico para redes sociais. exemplos_de_nicho: aplicativos de automação de marketing, ferramentas para social media. analise_de_concorrentes_referencias: None.. Incorpore essas informações para refinar a estratégia de conteúdo.

**Informações Estratégicas do Briefing:**
- Objetivos de Marketing: gerar confiança na ferramenta e atrair usuários para experimentarem o serviço.
- Canais de Distribuição: Instagram, LinkedIn, Facebook, Landing page
- Tópicos Principais: como acelerar criação de conteúdo, dicas de marketing digital, casos reais de transformação com automação, tendências em marketing
- Palavras-Chave: marketing automatizado, ferramenta de social media
- Chamada para Ação (CTA): Experimente o Fluxo Criativo e descubra como gerar conteúdo de alto impacto em minutos!
- Restrições e Diretrizes: Evitar jargões técnicos de programação; manter o foco no benefício real para o usuário; não prometer resultados irreais; comunicar com clareza e empatia.
- Referências de Estilo e Formato: carrosséis educativos sobre IA e marketing, Instagram, Linkedin

**Contexto Semanal:**
- Temas da Semana: 
- Objetivo Semanal: gerar confiança na ferramenta e atrair usuários para experimentarem o serviço.

**Instruções de Formato:**
Crie uma campanha semanal *coesa e com narrativa progressiva*, composta por 5 ideias de posts. Para cada post, inclua:
 Os 5 Posts serão postados na seguinte ordem: sexta, sábado, domingo, segunda e quarta. Baseado no nicho, objetivo e nos dias da semana, retorne também um horário recomendado para postagem.
 Busque se possivel inserir stroytelling ou casos ficticios no texto. Busque também inserir dados concretos, números ou estatísticas pertinentes mesmo que sejam aproximados no texto se fizer sentido.
A campanha deve seguir uma narrativa progressiva ao longo dos 5 posts, abordando:
- Post 1: Atenção / Gancho Forte: Chamar atenção com um insight, dado ou pergunta direta.
- Post 2: Problema: Reforçar a dor ou oportunidade perdida.
- Post 3: Oferta: Apresentar a solução paga e seus diferenciais.
- Post 4: Urgência / Escassez: Mostrar prazo, bônus ou vantagens limitadas.
- Post 5: CTA Final: Levar diretamente ao cadastro, compra ou formulário.
- `titulo`: Um título conciso para o post.
- `tema`: Resumo do tema abordado no post.
- `legenda_principal`: A legenda principal do post.
- `variacoes_legenda`: Uma lista de 2-3 variações da legenda principal.
- `hashtags`: Uma lista de hashtags relevantes.
- `sugestao_formato`: Sugestão de formato (ex: "Carrossel de imagens", "Vídeo curto", "Infográfico").
- `post_strategy_rationale`: Uma justificativa estratégica detalhada para este post, explicando como ele se encaixa na narrativa semanal e contribui para os objetivos gerais.
- `micro_briefing`: Um breve resumo do objetivo e foco principal do post, para contextualização.
- `micro_roteiro`: Se `sugestao_formato` for "Vídeo curto" ou "Reel", inclua um micro-roteiro detalhando cenas, falas/textos e chamadas para ação.
- `carrossel_slides`: Se `sugestao_formato` for "Carrossel de imagens", inclua uma lista de objetos, onde cada objeto representa um slide do carrossel, contendo `titulo_slide`, `texto_slide` e `sugestao_visual_slide`.
- `visual_prompt_suggestion`: Uma descrição detalhada da imagem ou vídeo principal para o post, incluindo estilo, cores, elementos e atmosfera, para ser usada por uma IA de geração de imagens. Evite usar texto no prompt para imagem.
- `text_in_image`: Caso tenha alguma sugestão de texto para incluir na imagem. Use esse campo
- `visual_description_portuguese`: Uma descrição em português da imagem ou vídeo principal, para ser exibida no briefing.
- `cta_individual`: Uma chamada para ação específica para este post.
- `interacao`: Sugerir uma pergunta para a audiência ou uma forma de incentivar comentários, aumentando o engajamento além da simples publicação..
- `response_script`: Array de objetos com respostas prontas para os comentários mais prováveis. Cada objeto deve conter: `comentario_generico` (Comentário Genérico mais provável), `resposta_sugerida` (resposta sugerida para o comentário mais provável), `comentario_negativo` (Comentário Negativo mais provável), `resposta_negativo` (resposta para o comentario negativo mais provável). Inclua 2 tipos de comentários prováveis.
- `ab_test_suggestions`: Uma string para cada post, sugirindo 1-2 testes A/B com formato: 'Elemento a testar (ex: título) → Variação A vs Variação B → Métrica de sucesso'.
- `indicador_principal`: A métrica mais importante para este post.
- `optimization_triggers`: Retorne uma string com base no `indicador_principal`, defina: 1) Threshold de alerta , 2) Ação corretiva, 3) Threshold de sucesso.

 Gere **APENAS UM JSON VÁLIDO**, sem texto adicional, comentários ou explicações. Não inclua marcações de code block (como json). Retorne o conteúdo em formato JSON, com a estrutura abaixo:
{{
    "weekly_strategy_summary": "Resumo da estratégia semanal para a campanha.",
    "future_strategy": {"proximos_passos": "Sugestões gerais para depois da campanha.", "posts_nutricao": [{"tema": "Tema do post de nutrição", "formato": "Formato do post (ex: Carrossel, Vídeo)", "objetivo": "Objetivo do post de nutrição"}], "remarketing": [{"estrategia": "Estratégia de remarketing", "canal": "Canal de remarketing (ex: Email, Anúncios)"}], "long_term": {"comunidade": "Estratégias para comunidade", "parcerias": "Estratégias de parcerias"}}, 
    "market_references": "Uma lista de 3 objetos JSON, cada um contendo: 'Nome/Handle' (string), 'Diferenciais' (string), 'Oportunidades' (string), 'Posicionamento do Cliente' (string). Exemplo: [{'Nome/Handle': 'PerfilExemplo', 'Diferenciais': 'Conteúdo visual de alta qualidade', 'Oportunidades': 'Maior engajamento em postagens diárias', 'Posicionamento do Cliente': 'Enfatizar expertise técnica'}].",
    "posts": [
        {{
            "titulo": "Título do Post 1",
            "tema": "Resumo do tema abordado no post 1",
            "legenda_principal": "Legenda principal do post 1.",
            "variacoes_legenda": [
                "Variação 1 da legenda 1.",
                "Variação 2 da legenda 1."
            ],
            "hashtags": ["#Hashtag1", "#Hashtag2"],
            "horario_de_postagem": "Horário sugerido para postagem",
            "sugestao_formato": "Sugestão de formato para o post (ex: 'Carrossel de imagens', 'Vídeo', 'Reel', 'Imagem única', 'Infográfico'). Pode adicionar explicações.",
            "carrossel_slides": [], // Sempre inclua este campo como um array vazio se o formato não for 'Carrossel'.
            "micro_roteiro": [], // Sempre inclua este campo como um array vazio se o formato não for 'Vídeo' ou 'Reel'.
            "post_strategy_rationale": "Justificativa estratégica para este post.",
            "micro_briefing": "Breve resumo do objetivo do post.",
            "visual_prompt_suggestion": "Descrição detalhada para IA de geração de imagens. Evite usar texto no prompt para imagem",
            "text_in_image": "Caso tenha sugestão de texto para ser incluído na imagem. Use esse campo",
            "visual_description_portuguese": "Descrição em português da imagem ou vídeo principal.",
            "cta_individual": "Chamada para ação específica para este post.",
            "ab_test_suggestions": "Uma string com sugestões para testes A/B neste post.",
            "indicador_principal": "A métrica mais importante para este post.",
            "optimization_triggers": "Retorne uma string com base no `indicador_principal`, defina: 1) Threshold de alerta , 2) Ação corretiva, 3) Threshold de sucesso.",
            "interacao": "Formas de como aumentar a interação com este post.",
            "response_script": [
                {
                    "comentario_generico": Comentário genérico mais provável para este post
                    "resposta_sugerida": "Resposta sugerida para o comentário genérico",
                    "comentario_negativo": "Comentário negativo mais provável para este post",
                    "resposta_negativo": "Resposta para o comentário negativo mais provável"
                },
                {
                    "comentario_generico": Segundo comentário genérico mais provável para este post
                    "resposta_sugerida": "Resposta sugerida para o segundo comentário genérico",
                    "comentario_negativo": "Segundo comentário negativo mais provável para este post",
                    "resposta_negativo": "Resposta para o segundo comentário negativo mais provável"
                }
            ],
            "micro_roteiro": [
                {
                    "cena": 1,
                    "descricao": "Pessoa sorrindo",
                    "texto_tela": "Problema resolvido!",
                    "fala": "Você sabia que...?"
                }
            ],
            "carrossel_slides": [
                {
                    "titulo_slide": "Título do Slide 1",
                    "texto_slide": "Texto do slide 1",
                    "sugestao_visual_slide": "Visual para slide 1"
                }
            ]
        }}
    ],
    "metricas_de_sucesso_sugeridas": {
        "objetivo_principal": "gerar confiança na ferramenta e atrair usuários para experimentarem o serviço.",
        "indicadores_chave": [
            "Taxa de conversão (%)",
            "Valor total de vendas ou receita gerada",
            "Custo por lead (CPL)",
            "Retorno sobre investimento (ROI)"
        ],
        "metricas_secundarias": [
            "CTR em botões de compra",
            "Taxa de abandono no checkout",
            "Tempo médio até a conversão"
        ]
    },
}}
"
## Resultado gerado em HTML

"
<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Roteiro de Publicações para Fluxo Criativo</title>
    <link href="https://fonts.googleapis.com/css2?family=Roboto:wght@300;400;700&display=swap" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css">
    <style>
        body { font-family: 'Roboto', sans-serif; line-height: 1.6; color: #333; margin: 0; padding: 0; background-color: #f4f4f4; }
        .container { width: 80%; margin: 20px auto; background: #fff; padding: 30px; border-radius: 8px; box-shadow: 0 0 10px rgba(0,0,0,0.1); }
        /* Capa - Mudar para a cor primária do PDF */
        .cover {
            text-align: center;
            padding: 50px 0;
            background: linear-gradient(to bottom, #1A237E, #3949AB);
            color: #fff;
            border-radius: 8px 8px 0 0;
            margin-bottom: 30px;
        }
        .cover h1 { margin: 0; font-size: 2.5em; }
        .cover p { margin: 0; font-size: 1.2em; margin-top: 10px; }
        /* Títulos de Seção */
        h1, h2, h3 {
            font-weight: 700;
            letter-spacing: 0.5px;
        }
        h2 {
            color: #1A237E; /* Azul Escuro */
            border-bottom: 3px solid #5C6BC0; /* Azul Médio */
            padding-bottom: 10px;
            margin-top: 40px;
            font-size: 1.8em;
        }
        /* Bloco de Post */
        .post-section {
            background-color: #F5F5F5; /* Cinza muito claro */
            border-left: 6px solid #1A237E; /* Linha de destaque */
            padding: 25px;
            margin-bottom: 30px;
            border-radius: 8px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.05); /* Sombra sutil */
        }
        /* Subtítulos dentro do Post */
        .post-section h3 {
            color: #1A237E; /* Azul Escuro */
            border-bottom: 1px solid #E0E0E0;
            padding-bottom: 5px;
            margin-top: 0;
            margin-bottom: 15px;
        }
        .post-section .subtitulo-grupo1 strong:first-child { color: #1A237E !important; } /* Azul Escuro */
        .post-section .subtitulo-grupo2 strong:first-child { color: #5C6BC0 !important; } /* Azul Médio */
        .post-section .subtitulo-grupo3 strong:first-child { color: #2E7D32 !important; } /* Verde */
        .post-section .subtitulo-grupo4 strong:first-child { color: #555555 !important; } /* Cinza Escuro */
        /* Destaque para os rótulos (Tema, Legenda, etc.) – agora específico para labels */
        .post-section p > strong:first-child {
            
            font-weight: 700;
            text-transform: uppercase;
            font-size: 1.1em;
        }
        /* Reset para strong internos (no texto Markdown) */
        .post-section p strong:not(:first-child) {
            color: inherit; /* Sem cor extra */
            text-transform: none; /* Sem uppercase */
            font-size: inherit; /* Tamanho normal */
        }
        .checklist { list-style-type: none; padding: 0; }
        .checklist li { background: #f0f0f0; margin-bottom: 5px; padding: 10px; border-radius: 3px; }
        .checklist li:before { content: "\f058"; font-family: "Font Awesome 6 Free"; color: #2E7D32; font-weight: bold; margin-right: 8px; }
        em {
            font-style: italic;
            color: #5C6BC0;
        }
        .calendar-table { width: 100%; border-collapse: separate; border-spacing: 2px; margin-top: 20px; }
        .calendar-table th, .calendar-table td { border: 2px solid #333; padding: 12px; text-align: left; }
        .calendar-table th { background-color: #3F51B5; color: #fff; position: relative; } /* Cabeçalho mais vibrante */
        .calendar-table th:before { font-family: "Font Awesome 6 Free"; margin-right: 8px; } /* Ícone de calendário em th */
        .calendar-table td:first-child { font-weight: bold; color: #1A237E; } /* Bold em dias */
        .calendar-table tr:nth-child(even) { background-color: #E8EAF6; } /* Azul claro alternado */
        .calendar-table tr:hover { background-color: #C5CAE9; transition: background 0.3s; } /* Hover para destaque */
        .footer {
            width: 100%;
            margin-top: 50px;
            font-size: 0.9em;
            color: #777;
            border-top: 1px solid #ddd;
            padding-top: 20px;
        }
        .footer p {
            display: block;
            text-align: right;
        }
        .checklist li { padding: 12px; margin-bottom: 8px; display: flex; align-items: center; }
        .checklist ul { margin-left: 20px; list-style-type: disc; } /* Sub-listas com bullets */
        ul.competitor-list li { border-bottom: 1px solid #E0E0E0; padding-bottom: 10px; margin-bottom: 10px; }
        ul.competitor-list li:before { content: "\f091"; font-family: "Font Awesome 6 Free"; color: #FF5722; margin-right: 8px; } /* Ícone de trophy para refs */
        details { margin-bottom: 15px; }
        summary { cursor: pointer; font-weight: bold; color: #5C6BC0; }
        .post-section p strong:before { font-family: "Font Awesome 6 Free"; margin-right: 6px; }
        .post-section p strong[data-icon="theme"]:before { content: "\f249"; }
        .post-section p strong[data-icon="cta"]:before { content: "\f0a1"; }
        .post-section p { word-break: break-word; max-width: 100%; }
        @media print {
            .container { width: 100%; box-shadow: none; }
            .cover { page-break-after: always; }
            .post-section { page-break-inside: avoid; }
        }
        @media (max-width: 768px) {
            .container { width: 95%; padding: 15px; }
            .post-section { padding: 15px; }
            .calendar-table { font-size: 0.9em; }
            .calendar-table th, .calendar-table td { padding: 8px; }
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="cover">
            <h1>Briefing de Conteúdo Profissional</h1>
            <p>Para: Fluxo Criativo</p>
            <p>Data: 17/10/2025</p>
        </div>

        <h2>Visão Geral do Conteúdo</h2>
        <p>Este documento apresenta o conteúdo sugerido para as redes sociais do cliente, com base nas diretrizes fornecidas e no perfil do público-alvo.</p>

        
    <div class="quick-view-section" style="
        background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
        padding: 40px;
        margin: 30px 0;
        border-radius: 12px;
        color: white;
        box-shadow: 0 10px 30px rgba(0,0,0,0.2);
    ">
        <h2 style="color: white; border: none; margin-top: 0; font-size: 2em;">
            ⚡ Quick View - Versão Rápida
        </h2>
        <p style="opacity: 0.9; font-size: 1.1em; margin-bottom: 30px;">
            Um resumo dos posts dessa semana. Role para baixo para ver a estratégia completa.
        </p>
    
        <div style="
            background: white;
            color: #333;
            padding: 25px;
            margin: 20px 20px;
            margin-bottom: 30px;
            border-radius: 8px;
            box-shadow: 0 4px 12px rgba(0,0,0,0.15);
        ">
            <div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 15px;">
                <h3 style="margin: 0; color: #667eea; font-size: 1.3em;">
                    Post 1: Sua semana de trabalho tem horas suficientes?
                </h3>
                <a href="#post-1" style="color: #667eea; text-decoration: underline; font-size: 0.95em; margin-left: 10px;">Ver detalhes completos</a>
            </div>
            
            <p style="margin: 10px 0; color: #666; font-size: 0.95em;">
                📅 <strong>Sexta-feira, 18:00</strong>
            </p>
            
            <!-- Legenda copyable -->
           <div style="
             background: #f0f0f0;
            padding: 12px 16px;
            border-radius: 8px;  /* Menos arredondado = melhor pra texto longo */
            font-size: 0.85em;
            font-weight: bold;
            color: #666;
            margin: 10px 0;
            line-height: 1.5;  /* Espaçamento entre linhas */
            display: inline-block;  /* Só ocupa o espaço necessário */
            max-width: 100%;  /* Mas respeita container */
        ">
                    Carrossel de imagens

            </div>
            
            <div style="margin: 20px 20px;">
                <label style="
                    display: block;
                    font-weight: bold;
                    margin-bottom: 8px;
                    color: #667eea;
                    font-size: 0.9em;
                    text-transform: uppercase;
                    letter-spacing: 0.5px;
                ">
                    📝 Legenda:
                </label>
                <textarea readonly onclick="this.select()" style="
                    width: 100%;
                    min-height: 120px;
                    padding: 15px;
                    border: 2px solid #e0e0e0;
                    border-radius: 6px;
                    font-family: inherit;
                    font-size: 0.95em;
                    line-height: 1.6;
                    resize: vertical;
                    cursor: pointer;
                    transition: border-color 0.3s;
                " onfocus="this.style.borderColor='#667eea'">Você, solopreneur, já parou pra pensar que pode estar investindo quase um dia de trabalho por semana só na criação de conteúdo?

 Se você pudesse ganhar 8 horas de volta na sua semana, o que faria com elas? Atenderia mais um cliente? Começaria aquele curso novo? É exatamente esse o tempo que muitos solopreneurs perdem criando conteúdo. A questão é: esse tempo está te trazendo o retorno esperado? Vamos conversar sobre isso. 
Imagine ter um dia inteiro a mais na sua agenda toda semana. É o que mui...</textarea>
            </div>
            
            <!-- Hashtags copyable -->
            <div style="margin: 20px 20px;">
                <label style="
                    display: block;
                    font-weight: bold;
                    margin-bottom: 8px;
                    color: #667eea;
                    font-size: 0.9em;
                    text-transform: uppercase;
                    letter-spacing: 0.5px;
                ">
                    #️⃣ Hashtags:
                </label>
                <input 
                    type="text" 
                    readonly 
                    onclick="this.select()"
                    value="#MarketingDeConteudo #Produtividade #Solopreneur #EmpreendedorismoDigital #GestaoDeTempo"
                    style="
                        width: 100%;
                        padding: 12px 15px;
                        border: 2px solid #e0e0e0;
                        border-radius: 6px;
                        font-family: inherit;
                        cursor: pointer;
                        font-size: 0.9em;
                    "
                    onfocus="this.style.borderColor='#667eea'"
                />
            </div>
            
            <!-- CTA -->
            <div style="
                background: #f8f9ff;
                padding: 15px;
                border-left: 4px solid #667eea;
                border-radius: 4px;
                margin-top: 15px;
            ">
                <strong style="color: #667eea; font-size: 1.05em;">💬 CTA:</strong> 
                <span style="color: #333;">Comente abaixo: quantas horas por semana você dedica à criação de conteúdo?</span>
            </div>
            
            <div style="
                background: #f8f9ff;
                padding: 15px;
                border-left: 4px solid #667eea;
                border-radius: 4px;
                margin-top: 15px;
            ">
            <strong style="color: #667eea; font-size: 1.05em;">Indicador Principal:</strong> 
                <span style="color: #333;">Alcance e Comentários</span>
            </div>
            
        </div>
        
        <div style="
            background: white;
            color: #333;
            padding: 25px;
            margin: 20px 20px;
            margin-bottom: 30px;
            border-radius: 8px;
            box-shadow: 0 4px 12px rgba(0,0,0,0.15);
        ">
            <div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 15px;">
                <h3 style="margin: 0; color: #667eea; font-size: 1.3em;">
                    Post 2: A Roda do Hamster de Conteúdo: Você está nela?
                </h3>
                <a href="#post-2" style="color: #667eea; text-decoration: underline; font-size: 0.95em; margin-left: 10px;">Ver detalhes completos</a>
            </div>
            
            <p style="margin: 10px 0; color: #666; font-size: 0.95em;">
                📅 <strong>Sábado, 11:00</strong>
            </p>
            
            <!-- Legenda copyable -->
           <div style="
             background: #f0f0f0;
            padding: 12px 16px;
            border-radius: 8px;  /* Menos arredondado = melhor pra texto longo */
            font-size: 0.85em;
            font-weight: bold;
            color: #666;
            margin: 10px 0;
            line-height: 1.5;  /* Espaçamento entre linhas */
            display: inline-block;  /* Só ocupa o espaço necessário */
            max-width: 100%;  /* Mas respeita container */
        ">
                    Vídeo curto (Reel)

            </div>
            
            <div style="margin: 20px 20px;">
                <label style="
                    display: block;
                    font-weight: bold;
                    margin-bottom: 8px;
                    color: #667eea;
                    font-size: 0.9em;
                    text-transform: uppercase;
                    letter-spacing: 0.5px;
                ">
                    📝 Legenda:
                </label>
                <textarea readonly onclick="this.select()" style="
                    width: 100%;
                    min-height: 120px;
                    padding: 15px;
                    border: 2px solid #e0e0e0;
                    border-radius: 6px;
                    font-family: inherit;
                    font-size: 0.95em;
                    line-height: 1.6;
                    resize: vertical;
                    cursor: pointer;
                    transition: border-color 0.3s;
                " onfocus="this.style.borderColor='#667eea'">Você conhece a Laura? Uma coach de carreira brilhante, cheia de insights para compartilhar.

 Postar todo dia. Pouco engajamento. Sentir que está falando sozinho(a). Repetir. Essa é a 'Roda do Hamster de Conteúdo', e muitos solopreneurs estão presos nela. A verdade? Consistência sem estratégia é apenas esforço desperdiçado. Você se identifica com essa situação? 
Vamos falar sobre o ciclo vicioso do conteúdo: 1) Pressão para postar. 2) Horas gastas para criar algo. 3) Resultados que não pagam o e...</textarea>
            </div>
            
            <!-- Hashtags copyable -->
            <div style="margin: 20px 20px;">
                <label style="
                    display: block;
                    font-weight: bold;
                    margin-bottom: 8px;
                    color: #667eea;
                    font-size: 0.9em;
                    text-transform: uppercase;
                    letter-spacing: 0.5px;
                ">
                    #️⃣ Hashtags:
                </label>
                <input 
                    type="text" 
                    readonly 
                    onclick="this.select()"
                    value="#MarketingDigitalBrasil #ConteudoEstrategico #BloqueioCriativo #Empreendedora #CoachingDeCarreira"
                    style="
                        width: 100%;
                        padding: 12px 15px;
                        border: 2px solid #e0e0e0;
                        border-radius: 6px;
                        font-family: inherit;
                        cursor: pointer;
                        font-size: 0.9em;
                    "
                    onfocus="this.style.borderColor='#667eea'"
                />
            </div>
            
            <!-- CTA -->
            <div style="
                background: #f8f9ff;
                padding: 15px;
                border-left: 4px solid #667eea;
                border-radius: 4px;
                margin-top: 15px;
            ">
                <strong style="color: #667eea; font-size: 1.05em;">💬 CTA:</strong> 
                <span style="color: #333;">Se você se identificou, comente 'EU' aqui embaixo. 👇</span>
            </div>
            
            <div style="
                background: #f8f9ff;
                padding: 15px;
                border-left: 4px solid #667eea;
                border-radius: 4px;
                margin-top: 15px;
            ">
            <strong style="color: #667eea; font-size: 1.05em;">Indicador Principal:</strong> 
                <span style="color: #333;">Taxa de Engajamento (comentários + salvamentos)</span>
            </div>
            
        </div>
        
        <div style="
            background: white;
            color: #333;
            padding: 25px;
            margin: 20px 20px;
            margin-bottom: 30px;
            border-radius: 8px;
            box-shadow: 0 4px 12px rgba(0,0,0,0.15);
        ">
            <div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 15px;">
                <h3 style="margin: 0; color: #667eea; font-size: 1.3em;">
                    Post 3: E se você tivesse um estrategista de conteúdo 24/7?
                </h3>
                <a href="#post-3" style="color: #667eea; text-decoration: underline; font-size: 0.95em; margin-left: 10px;">Ver detalhes completos</a>
            </div>
            
            <p style="margin: 10px 0; color: #666; font-size: 0.95em;">
                📅 <strong>Domingo, 20:00</strong>
            </p>
            
            <!-- Legenda copyable -->
           <div style="
             background: #f0f0f0;
            padding: 12px 16px;
            border-radius: 8px;  /* Menos arredondado = melhor pra texto longo */
            font-size: 0.85em;
            font-weight: bold;
            color: #666;
            margin: 10px 0;
            line-height: 1.5;  /* Espaçamento entre linhas */
            display: inline-block;  /* Só ocupa o espaço necessário */
            max-width: 100%;  /* Mas respeita container */
        ">
                    Carrossel de imagens (educativo/demonstrativo)

            </div>
            
            <div style="margin: 20px 20px;">
                <label style="
                    display: block;
                    font-weight: bold;
                    margin-bottom: 8px;
                    color: #667eea;
                    font-size: 0.9em;
                    text-transform: uppercase;
                    letter-spacing: 0.5px;
                ">
                    📝 Legenda:
                </label>
                <textarea readonly onclick="this.select()" style="
                    width: 100%;
                    min-height: 120px;
                    padding: 15px;
                    border: 2px solid #e0e0e0;
                    border-radius: 6px;
                    font-family: inherit;
                    font-size: 0.95em;
                    line-height: 1.6;
                    resize: vertical;
                    cursor: pointer;
                    transition: border-color 0.3s;
                " onfocus="this.style.borderColor='#667eea'">Lembra da frustração da Laura e da 'Roda do Hamster'? Agora, imagine este cenário:

 A paralisia da tela em branco acabou. Apresentamos o Fluxo Criativo: sua nova plataforma de criação de conteúdo estratégico. Funciona assim: você nos dá o seu conhecimento, e nós o transformamos em posts personalizados que fortalecem sua autoridade. É a união perfeita da sua expertise com a nossa tecnologia. Simples, rápido e inteligente. 
E se criar o conteúdo da semana levasse o tempo de um café? Com o Fluxo C...</textarea>
            </div>
            
            <!-- Hashtags copyable -->
            <div style="margin: 20px 20px;">
                <label style="
                    display: block;
                    font-weight: bold;
                    margin-bottom: 8px;
                    color: #667eea;
                    font-size: 0.9em;
                    text-transform: uppercase;
                    letter-spacing: 0.5px;
                ">
                    #️⃣ Hashtags:
                </label>
                <input 
                    type="text" 
                    readonly 
                    onclick="this.select()"
                    value="#FerramentaDeMarketing #AutomacaoCriativa #MarketingAutomatizado #ConteudoInteligente #FluxoCriativo"
                    style="
                        width: 100%;
                        padding: 12px 15px;
                        border: 2px solid #e0e0e0;
                        border-radius: 6px;
                        font-family: inherit;
                        cursor: pointer;
                        font-size: 0.9em;
                    "
                    onfocus="this.style.borderColor='#667eea'"
                />
            </div>
            
            <!-- CTA -->
            <div style="
                background: #f8f9ff;
                padding: 15px;
                border-left: 4px solid #667eea;
                border-radius: 4px;
                margin-top: 15px;
            ">
                <strong style="color: #667eea; font-size: 1.05em;">💬 CTA:</strong> 
                <span style="color: #333;">Clique no link da bio para descobrir como a criação inteligente pode transformar sua rotina.</span>
            </div>
            
            <div style="
                background: #f8f9ff;
                padding: 15px;
                border-left: 4px solid #667eea;
                border-radius: 4px;
                margin-top: 15px;
            ">
            <strong style="color: #667eea; font-size: 1.05em;">Indicador Principal:</strong> 
                <span style="color: #333;">Cliques no Link da Bio</span>
            </div>
            
        </div>
        
        <div style="
            background: white;
            color: #333;
            padding: 25px;
            margin: 20px 20px;
            margin-bottom: 30px;
            border-radius: 8px;
            box-shadow: 0 4px 12px rgba(0,0,0,0.15);
        ">
            <div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 15px;">
                <h3 style="margin: 0; color: #667eea; font-size: 1.3em;">
                    Post 4: Um presente para quem age rápido.
                </h3>
                <a href="#post-4" style="color: #667eea; text-decoration: underline; font-size: 0.95em; margin-left: 10px;">Ver detalhes completos</a>
            </div>
            
            <p style="margin: 10px 0; color: #666; font-size: 0.95em;">
                📅 <strong>Segunda-feira, 09:00</strong>
            </p>
            
            <!-- Legenda copyable -->
           <div style="
             background: #f0f0f0;
            padding: 12px 16px;
            border-radius: 8px;  /* Menos arredondado = melhor pra texto longo */
            font-size: 0.85em;
            font-weight: bold;
            color: #666;
            margin: 10px 0;
            line-height: 1.5;  /* Espaçamento entre linhas */
            display: inline-block;  /* Só ocupa o espaço necessário */
            max-width: 100%;  /* Mas respeita container */
        ">
                    Imagem única com texto forte

            </div>
            
            <div style="margin: 20px 20px;">
                <label style="
                    display: block;
                    font-weight: bold;
                    margin-bottom: 8px;
                    color: #667eea;
                    font-size: 0.9em;
                    text-transform: uppercase;
                    letter-spacing: 0.5px;
                ">
                    📝 Legenda:
                </label>
                <textarea readonly onclick="this.select()" style="
                    width: 100%;
                    min-height: 120px;
                    padding: 15px;
                    border: 2px solid #e0e0e0;
                    border-radius: 6px;
                    font-family: inherit;
                    font-size: 0.95em;
                    line-height: 1.6;
                    resize: vertical;
                    cursor: pointer;
                    transition: border-color 0.3s;
                " onfocus="this.style.borderColor='#667eea'">Nós vimos o quanto vocês se identificaram com o desafio de criar conteúdo. E para ajudar os solopreneurs que estão realmente comprometidos em dar o próximo passo, preparamos algo especial.

 Você quer apenas testar uma ferramenta ou quer um sistema completo para decolar sua autoridade? Para os 100 primeiros a se cadastrarem no Fluxo Criativo, não vamos entregar apenas o acesso, mas também nosso Kit de Autoridade Acelerada. É o nosso empurrãozinho para você começar com tudo. Vagas limitadas. Gara...</textarea>
            </div>
            
            <!-- Hashtags copyable -->
            <div style="margin: 20px 20px;">
                <label style="
                    display: block;
                    font-weight: bold;
                    margin-bottom: 8px;
                    color: #667eea;
                    font-size: 0.9em;
                    text-transform: uppercase;
                    letter-spacing: 0.5px;
                ">
                    #️⃣ Hashtags:
                </label>
                <input 
                    type="text" 
                    readonly 
                    onclick="this.select()"
                    value="#Oportunidade #BonusExclusivo #MarketingParaCoaches #VagasLimitadas #CrescimentoAcelerado"
                    style="
                        width: 100%;
                        padding: 12px 15px;
                        border: 2px solid #e0e0e0;
                        border-radius: 6px;
                        font-family: inherit;
                        cursor: pointer;
                        font-size: 0.9em;
                    "
                    onfocus="this.style.borderColor='#667eea'"
                />
            </div>
            
            <!-- CTA -->
            <div style="
                background: #f8f9ff;
                padding: 15px;
                border-left: 4px solid #667eea;
                border-radius: 4px;
                margin-top: 15px;
            ">
                <strong style="color: #667eea; font-size: 1.05em;">💬 CTA:</strong> 
                <span style="color: #333;">Seja um dos 100 primeiros! Clique no link da bio e garanta seu teste + bônus agora!</span>
            </div>
            
            <div style="
                background: #f8f9ff;
                padding: 15px;
                border-left: 4px solid #667eea;
                border-radius: 4px;
                margin-top: 15px;
            ">
            <strong style="color: #667eea; font-size: 1.05em;">Indicador Principal:</strong> 
                <span style="color: #333;">Taxa de Conversão na Landing Page (inícios de teste)</span>
            </div>
            
        </div>
        
        <div style="
            background: white;
            color: #333;
            padding: 25px;
            margin: 20px 20px;
            margin-bottom: 30px;
            border-radius: 8px;
            box-shadow: 0 4px 12px rgba(0,0,0,0.15);
        ">
            <div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 15px;">
                <h3 style="margin: 0; color: #667eea; font-size: 1.3em;">
                    Post 5: Sua escolha: continuar correndo ou começar a voar?
                </h3>
                <a href="#post-5" style="color: #667eea; text-decoration: underline; font-size: 0.95em; margin-left: 10px;">Ver detalhes completos</a>
            </div>
            
            <p style="margin: 10px 0; color: #666; font-size: 0.95em;">
                📅 <strong>Quarta-feira, 12:00</strong>
            </p>
            
            <!-- Legenda copyable -->
           <div style="
             background: #f0f0f0;
            padding: 12px 16px;
            border-radius: 8px;  /* Menos arredondado = melhor pra texto longo */
            font-size: 0.85em;
            font-weight: bold;
            color: #666;
            margin: 10px 0;
            line-height: 1.5;  /* Espaçamento entre linhas */
            display: inline-block;  /* Só ocupa o espaço necessário */
            max-width: 100%;  /* Mas respeita container */
        ">
                    Vídeo curto (Reel) ou Imagem com Citação Forte

            </div>
            
            <div style="margin: 20px 20px;">
                <label style="
                    display: block;
                    font-weight: bold;
                    margin-bottom: 8px;
                    color: #667eea;
                    font-size: 0.9em;
                    text-transform: uppercase;
                    letter-spacing: 0.5px;
                ">
                    📝 Legenda:
                </label>
                <textarea readonly onclick="this.select()" style="
                    width: 100%;
                    min-height: 120px;
                    padding: 15px;
                    border: 2px solid #e0e0e0;
                    border-radius: 6px;
                    font-family: inherit;
                    font-size: 0.95em;
                    line-height: 1.6;
                    resize: vertical;
                    cursor: pointer;
                    transition: border-color 0.3s;
                " onfocus="this.style.borderColor='#667eea'">Nesta semana, falamos sobre as horas perdidas, a frustração de postar sem estratégia e a sensação de estar na 'Roda do Hamster'.

 Você pode passar o resto do mês pensando no que postar, ou pode usar os próximos 15 minutos para se cadastrar e ter ideias para o mês inteiro. A escolha entre esforço e estratégia está a um clique de distância. O link na bio é o seu primeiro passo para a automação criativa. Aja agora. 
Recapitulando: mais tempo na sua agenda, fim do bloqueio criativo, posts que const...</textarea>
            </div>
            
            <!-- Hashtags copyable -->
            <div style="margin: 20px 20px;">
                <label style="
                    display: block;
                    font-weight: bold;
                    margin-bottom: 8px;
                    color: #667eea;
                    font-size: 0.9em;
                    text-transform: uppercase;
                    letter-spacing: 0.5px;
                ">
                    #️⃣ Hashtags:
                </label>
                <input 
                    type="text" 
                    readonly 
                    onclick="this.select()"
                    value="#Decisao #TransformacaoDigital #FuturoDoMarketing #SejaEstrategico #CTA"
                    style="
                        width: 100%;
                        padding: 12px 15px;
                        border: 2px solid #e0e0e0;
                        border-radius: 6px;
                        font-family: inherit;
                        cursor: pointer;
                        font-size: 0.9em;
                    "
                    onfocus="this.style.borderColor='#667eea'"
                />
            </div>
            
            <!-- CTA -->
            <div style="
                background: #f8f9ff;
                padding: 15px;
                border-left: 4px solid #667eea;
                border-radius: 4px;
                margin-top: 15px;
            ">
                <strong style="color: #667eea; font-size: 1.05em;">💬 CTA:</strong> 
                <span style="color: #333;">Sua escala de resultados começa agora. Clique no link da bio para destravar seu potencial.</span>
            </div>
            
            <div style="
                background: #f8f9ff;
                padding: 15px;
                border-left: 4px solid #667eea;
                border-radius: 4px;
                margin-top: 15px;
            ">
            <strong style="color: #667eea; font-size: 1.05em;">Indicador Principal:</strong> 
                <span style="color: #333;">Cliques no Link da Bio e Taxa de Conversão Final</span>
            </div>
            
        </div>
        
        <div style="
            text-align: center;
            margin-top: 30px;
            padding-top: 20px;
            border-top: 1px solid rgba(255,255,255,0.3);
        ">
            <p style="font-size: 1.2em; margin-bottom: 10px; font-weight: bold;">
                ⬇️ Role para baixo para ver:
            </p>
            <div style="
                display: flex;
                justify-content: center;
                gap: 20px;
                flex-wrap: wrap;
                font-size: 0.95em;
                opacity: 0.9;
            ">
                <span>✨ Estratégia completa</span>
                <span>🎨 Roteiros de Reels/Carrosséis</span>
                <span>📊 Métricas e A/B tests</span>
                <span>💬 Scripts de resposta</span>
            </div>
        </div>
    </div>
    
                <h2>Sumário Executivo</h2>
        <p>Esta campanha semanal de 5 posts segue uma narrativa progressiva para construir confiança e impulsionar a experimentação da plataforma Fluxo Criativo. Começamos capturando a atenção com uma dor latente (Post 1), aprofundamos o problema e geramos empatia (Post 2), apresentamos a Fluxo Criativo como a solução estratégica (Post 3), criamos urgência com um bônus exclusivo (Post 4) e finalizamos com uma chamada para ação direta e clara (Post 5). O objetivo é guiar o solopreneur desde o reconhecimento do seu desafio de conteúdo até a decisão de testar nossa ferramenta de automação criativa.</p>
        <h3>Sugestões para Depois da Campanha:</h3>
        <p>Após esta campanha de conversão, o foco deve ser na nutrição dos novos leads e usuários. A estratégia deve se dividir em educar sobre o uso avançado da plataforma, compartilhar casos de sucesso para reforçar a prova social e iniciar ações de remarketing para quem não converteu.</p>
        <h4>Posts de Nutrição:</h4>
        <ul>
            <li><strong>Como criar seu calendário de 30 dias de conteúdo em 1 hora com o Fluxo Criativo</strong> - Vídeo Tutorial (Reel/YouTube Shorts) - Demonstrar a eficiência e a facilidade da ferramenta, reduzindo a curva de aprendizado e aumentando a retenção de usuários.</li>
            <li><strong>Case de Sucesso: Como a Coach [Nome Fictício] dobrou seu engajamento usando posts personalizados do Fluxo Criativo</strong> - Carrossel com Depoimento - Gerar prova social, mostrando resultados reais e tangíveis que inspiram confiança em potenciais clientes.</li>
        </ul>
        <h4>Estratégias de Remarketing:</h4>
        <ul>
            <li><strong>Anúncios segmentados para visitantes da landing page que não iniciaram o teste. Oferecer um template de conteúdo exclusivo ('5 Posts para Gerar Autoridade') como incentivo adicional para o cadastro.</strong> - Canal: Instagram/Facebook Ads</li>
        </ul>
        <h3>Análise de Concorrentes e Referências de Sucesso:</h3>
        <ul class="competitor-list">
            <li><strong>Nome/Handle:</strong> mLabs / Metricool<br>

                <strong>Diferenciais:</strong> Foco em agendamento, relatórios e gestão de múltiplas contas. Ferramentas consolidadas no mercado.<br>

                <strong>Oportunidades:</strong> Eles gerenciam o 'depois' (agendar e medir). A nossa oportunidade está no 'antes': a criação estratégica do conteúdo. Eles são a logística, nós somos a inteligência criativa.<br>

                <strong>Posicionamento do Cliente:</strong> O Fluxo Criativo não é apenas um agendador, é o seu estrategista de conteúdo pessoal. Focamos na qualidade e inteligência da criação, não apenas na distribuição.</li>

            <li><strong>Nome/Handle:</strong> Jasper / Copy.ai<br>

                <strong>Diferenciais:</strong> Geradores de texto por IA generalistas, com múltiplos templates para blogs, anúncios, etc.<br>

                <strong>Oportunidades:</strong> São como um canivete suíço, bons em tudo, mas mestres em nada. Nossa oportunidade é a especialização. Somos a ferramenta de IA treinada especificamente para o conteúdo de redes sociais de quem vende conhecimento.<br>

                <strong>Posicionamento do Cliente:</strong> Enquanto outros geram textos genéricos, o Fluxo Criativo entrega uma 'automação criativa' com posts personalizados, alinhados à sua voz e aos seus objetivos de negócio para construir autoridade.</li>

            <li><strong>Nome/Handle:</strong> Canva<br>

                <strong>Diferenciais:</strong> Líder absoluto em design simplificado, com foco total no apelo visual.<br>

                <strong>Oportunidades:</strong> O Canva resolve o design, mas não o texto estratégico. Muitos usuários abrem o Canva e travam sem saber O QUE escrever. Nós resolvemos essa paralisia.<br>

                <strong>Posicionamento do Cliente:</strong> O Fluxo Criativo é o parceiro perfeito para o seu Canva. Nós te damos a estratégia e o texto que convertem, você finaliza com o design que encanta. É a união da criação inteligente com a beleza visual.</li>

        </ul>

                <h2>Calendário de Publicação</h2>
        <table class="calendar-table">
            <thead>
                <tr><th>📅 Dia</th><th>Data</th><th>Horário</th><th>Post</th></tr>
            </thead>
            <tbody>
                <tr><td>Sexta-feira</td><td>17/10</td><td>18:00</td><td>Sua semana de trabalho tem horas suficientes?</td></tr>
                <tr><td>Sábado</td><td>18/10</td><td>11:00</td><td>A Roda do Hamster de Conteúdo: Você está nela?</td></tr>
                <tr><td>Domingo</td><td>19/10</td><td>20:00</td><td>E se você tivesse um estrategista de conteúdo 24/7?</td></tr>
                <tr><td>Segunda-feira</td><td>20/10</td><td>09:00</td><td>Um presente para quem age rápido.</td></tr>
                <tr><td>Quarta-feira</td><td>22/10</td><td>12:00</td><td>Sua escolha: continuar correndo ou começar a voar?</td></tr>
            </tbody>
        </table>


        <h2>Posts Sugeridos</h2>
        
            <div class="post-section" id="post-1">
                <h3>Post #1: Sua semana de trabalho tem horas suficientes?</h3>
               <p class="subtitulo-grupo1"><strong>Tema:</strong> Atenção / Gancho Forte: Quantificar o tempo gasto na criação de conteúdo.</p>
                <p class="subtitulo-grupo1"><strong>Justificativa Estratégica:</strong> Este primeiro post visa capturar a atenção do público-alvo ao tocar em uma dor universal e quantificável: a falta de tempo. Ao apresentar um dado estatístico e fazer uma pergunta direta, criamos um gancho forte que valida a experiência do solopreneur e o convida para a conversa. O objetivo é gerar identificação imediata e preparar o terreno para apresentar o problema de forma mais aprofundada no post seguinte.</p>
                <p class="subtitulo-grupo1"><strong>Micro Briefing:</strong> Gerar consciência sobre o tempo excessivo gasto com a criação de conteúdo e iniciar uma conversa sobre produtividade para solopreneurs.</p>
                <p class="subtitulo-grupo2"><strong>✍️ Legenda Principal:</strong> Você, solopreneur, já parou pra pensar que pode estar investindo quase um dia de trabalho por semana só na criação de conteúdo?</p>
                <details>
                    <summary style="text-decoration: underline; font-style: italic; color: #0000EE;">Clique para expandir variações</summary>
                    <p><strong>📝 Variações:</strong></p>
                    <ul>
                            <li>Se você pudesse ganhar 8 horas de volta na sua semana, o que faria com elas? Atenderia mais um cliente? Começaria aquele curso novo? É exatamente esse o tempo que muitos solopreneurs perdem criando conteúdo. A questão é: esse tempo está te trazendo o retorno esperado? Vamos conversar sobre isso.</li>
                    <li>Imagine ter um dia inteiro a mais na sua agenda toda semana. É o que muitos empreendedores gastam tentando criar o post perfeito. Mas será que a solução é trabalhar mais, ou trabalhar de forma mais inteligente? Reflita sobre isso e me diga: o que mais te consome tempo na hora de criar conteúdo?</li>

                    </ul>
                </details>
                <p class="subtitulo-grupo2"><strong># Hashtags:</strong> #MarketingDeConteudo #Produtividade #Solopreneur #EmpreendedorismoDigital #GestaoDeTempo</p>
                <p class="subtitulo-grupo2"><strong>Indicador Principal:</strong> Alcance e Comentários</p>
                <p class="subtitulo-grupo3"><strong>📢 Chamada para Ação:</strong> Comente abaixo: quantas horas por semana você dedica à criação de conteúdo?</p>
                <p class="subtitulo-grupo3"><strong>💡 Sugestões de Interação/Engajamento:</strong> Pergunte diretamente: 'Qual é a tarefa que mais consome seu tempo na criação de conteúdo: ter ideias, escrever as legendas ou criar as imagens?'</p>
        
                <details>
                    <summary style="text-decoration: underline; font-style: italic; color: #0000EE;">Clique para expandir roteiro de respostas</summary>
                    <p><strong>🗨️ Roteiro de Respostas:</strong></p>
                    <ul>
                                <li><strong>Comentário Genérico:</strong> Nossa, eu gasto muito mais que isso! É exaustivo.<br>
                        <strong>Resposta Sugerida:</strong> Totalmente! A gente entende essa realidade. É por isso que acreditamos que a tecnologia pode ser nossa aliada para otimizar esse processo. Fique de olho nos próximos posts que vamos falar mais sobre isso! 😉</li>
                    <li><strong>Comentário Negativo:</strong> Mais uma ferramenta pra fazer a gente gastar dinheiro.<br>
                        <strong>Resposta para Negativo:</strong> Entendemos seu ponto de vista. Na verdade, o objetivo é o contrário: fazer você economizar tempo, que é o seu ativo mais valioso, e gerar resultados que paguem o investimento e ainda tragam lucro. A ideia é ser um investimento, não um custo.</li>
                    <li><strong>Comentário Genérico:</strong> Minha maior dificuldade é ter ideias do que postar.<br>
                        <strong>Resposta Sugerida:</strong> Essa é a dor de muitos! O bloqueio criativo é real. E se você tivesse uma fonte infinita de ideias estratégicas, alinhadas com seu negócio? 🤔 Continue acompanhando a gente por aqui!</li>
                    <li><strong>Comentário Negativo:</strong> Isso de automação tira a autenticidade.<br>
                        <strong>Resposta para Negativo:</strong> Ótima observação! Acreditamos em uma 'automação criativa'. A ferramenta te dá a estratégia e a estrutura, mas a sua voz e personalidade são sempre o toque final. A ideia é potencializar sua autenticidade, não substituí-la.</li>

                    </ul>
                </details>
            
                <p class="subtitulo-grupo4"><strong>Sugestões Visuais:</strong> Imagem dividida. De um lado, um empreendedor com expressão de cansaço na frente do computador. Do outro, um calendário com um dia inteiro riscado com o texto 'Criação de Conteúdo'.</p>
                <p class="subtitulo-grupo4"><strong>Texto na Imagem/Vídeo:</strong> Você realmente tem 8 horas por semana para 'perder'?</p>
                <p class="subtitulo-grupo4"><strong>Formato Sugerido:</strong> Carrossel de imagens</p>
        
        
                <h4>🎞️ Slides do Carrossel:</h4>
                <ol>
                                <li><strong>Capa</strong>: Você gasta mais de 8 horas por semana criando conteúdo? (Visual: Um relógio grande com o ponteiro marcando 8 horas, sobreposto a uma mesa de trabalho de um empreendedor. Cores vibrantes, mas com um tom de alerta.)</li>
                    <li><strong>O Dilema do Solopreneur</strong>: Como coach ou consultor, seu tempo vale ouro. Cada hora criando posts é uma hora a menos mentorando, vendendo ou planejando o futuro do seu negócio. (Visual: Uma balança, com 'Criar Conteúdo' de um lado e 'Atender Clientes' do outro, visivelmente desequilibrada.)</li>
                    <li><strong>A Grande Pergunta...</strong>: Todo esse esforço está se convertendo em autoridade e vendas? Ou é apenas para 'marcar presença' online? (Visual: Uma grande interrogação (?) sobre um gráfico de engajamento baixo (poucas curtidas, comentários).)</li>
                    <li><strong>Você não está sozinho(a).</strong>: A dificuldade de criar conteúdo estratégico e consistente é o desafio Nº 1 de 60% dos empreendedores digitais. Mas existe uma forma mais inteligente de fazer isso. (Visual: Um ícone de 'check' verde ao lado da frase 'Trabalhe de forma inteligente, não mais difícil'.)</li>
                    <li><strong>CTA de Interação</strong>: Comente aqui: Qual sua maior dificuldade na hora de criar conteúdo para as redes sociais? (Visual: Um balão de diálogo com a pergunta em destaque, incentivando a resposta.)</li>

                </ol>
            
                <p class="subtitulo-grupo4"><strong>Prompt para IA Geradora de Imagens:</strong> A minimalist infographic style. A split screen, on the left, a frustrated person looking at a laptop with social media icons floating around. On the right, a calendar with one full day crossed out, labeled 'Content Creation'. Use a palette of teal, orange, and off-white. The atmosphere should be clean and slightly stressful.</p>
                <p class="subtitulo-grupo4"><strong>🧪 Testes A/B:</strong> Texto da imagem → 'Você gasta 8h/semana com conteúdo?' vs 'Recupere 1 dia de trabalho por semana.' → Métrica de sucesso: Taxa de cliques no perfil.</p>
                <p class="subtitulo-grupo4"><strong>⚠️ Como Corrigir a Rota:</strong> 1) Alerta: Alcance < 10% da média ou < 5 comentários em 24h. 2) Ação: Impulsionar o post para audiência semelhante ou compartilhar nos stories com uma enquete. 3) Sucesso: Alcance > 20% da média e mais de 15 comentários qualitativos.</p>
                <p class="subtitulo-grupo4"><strong>🚀 Checklist de Publicação:</strong></p>
                <ul class="checklist">
                    <li>Revisar texto e gramática.</li>
                    <li>Verificar a qualidade da imagem/vídeo.</li>
                    <li>Confirmar o agendamento.</li>
                    <li>Responder a comentários e mensagens.</li>
                </ul>
            </div>
        
            <div class="post-section" id="post-2">
                <h3>Post #2: A Roda do Hamster de Conteúdo: Você está nela?</h3>
               <p class="subtitulo-grupo1"><strong>Tema:</strong> Problema: Aprofundar a dor do esforço sem resultado.</p>
                <p class="subtitulo-grupo1"><strong>Justificativa Estratégica:</strong> Após capturar a atenção, este post aprofunda a dor usando o storytelling. A personagem 'Laura' cria uma conexão emocional e empática. O conceito de 'Roda do Hamster de Conteúdo' é uma metáfora poderosa que resume o sentimento de frustração do público. O objetivo é fazer com que a audiência pense 'Essa é exatamente a minha situação', aumentando a necessidade pela solução que será apresentada no próximo post.</p>
                <p class="subtitulo-grupo1"><strong>Micro Briefing:</strong> Criar empatia e aprofundar a dor da criação de conteúdo ineficaz através de uma narrativa (storytelling).</p>
                <p class="subtitulo-grupo2"><strong>✍️ Legenda Principal:</strong> Você conhece a Laura? Uma coach de carreira brilhante, cheia de insights para compartilhar.</p>
                <details>
                    <summary style="text-decoration: underline; font-style: italic; color: #0000EE;">Clique para expandir variações</summary>
                    <p><strong>📝 Variações:</strong></p>
                    <ul>
                            <li>Postar todo dia. Pouco engajamento. Sentir que está falando sozinho(a). Repetir. Essa é a 'Roda do Hamster de Conteúdo', e muitos solopreneurs estão presos nela. A verdade? Consistência sem estratégia é apenas esforço desperdiçado. Você se identifica com essa situação?</li>
                    <li>Vamos falar sobre o ciclo vicioso do conteúdo: 1) Pressão para postar. 2) Horas gastas para criar algo. 3) Resultados que não pagam o esforço. 4) Frustração. 5) Repete. Se você está nesse ciclo, saiba que a culpa não é sua. É do método. Está na hora de trocar o esforço bruto pela criação inteligente.</li>

                    </ul>
                </details>
                <p class="subtitulo-grupo2"><strong># Hashtags:</strong> #MarketingDigitalBrasil #ConteudoEstrategico #BloqueioCriativo #Empreendedora #CoachingDeCarreira</p>
                <p class="subtitulo-grupo2"><strong>Indicador Principal:</strong> Taxa de Engajamento (comentários + salvamentos)</p>
                <p class="subtitulo-grupo3"><strong>📢 Chamada para Ação:</strong> Se você se identificou, comente 'EU' aqui embaixo. 👇</p>
                <p class="subtitulo-grupo3"><strong>💡 Sugestões de Interação/Engajamento:</strong> Peça para as pessoas marcarem um(a) amigo(a) empreendedor(a) que também precisa sair dessa 'roda do hamster'.</p>
        
                <details>
                    <summary style="text-decoration: underline; font-style: italic; color: #0000EE;">Clique para expandir roteiro de respostas</summary>
                    <p><strong>🗨️ Roteiro de Respostas:</strong></p>
                    <ul>
                                <li><strong>Comentário Genérico:</strong> EU! Totalmente eu. É muito frustrante.<br>
                        <strong>Resposta Sugerida:</strong> A gente sabe como é. Mas a boa notícia é que você não precisa continuar assim. A solução está em trocar o esforço pela estratégia. Fica com a gente que o próximo passo vem aí!</li>
                    <li><strong>Comentário Negativo:</strong> Ah, mas se fosse fácil todo mundo fazia.<br>
                        <strong>Resposta para Negativo:</strong> Concordamos, não é 'fácil' no sentido de não exigir nada. Mas pode ser muito mais 'simples' e 'inteligente'. A ideia não é eliminar o trabalho, mas direcionar sua energia para o que realmente gera resultado.</li>
                    <li><strong>Comentário Genérico:</strong> Essa Laura sou eu todinha!<br>
                        <strong>Resposta Sugerida:</strong> É a história de muitas de nós! O importante é reconhecer que está nesse ciclo para poder sair dele. O primeiro passo você já deu. 😉</li>
                    <li><strong>Comentário Negativo:</strong> Isso não funciona pro meu nicho.<br>
                        <strong>Resposta para Negativo:</strong> Essa é uma dúvida comum! A beleza da estratégia é que ela se adapta. O método por trás da criação de conteúdo de autoridade funciona para qualquer nicho que venda conhecimento, de coaching a consultoria financeira. O que muda é o 'tempero' que você coloca.</li>

                    </ul>
                </details>
            
                <p class="subtitulo-grupo4"><strong>Sugestões Visuais:</strong> Vídeo curto mostrando a jornada frustrante de uma empreendedora criando conteúdo, culminando na metáfora de uma roda de hamster e terminando com um olhar de esperança.</p>
                
                <p class="subtitulo-grupo4"><strong>Formato Sugerido:</strong> Vídeo curto (Reel)</p>
        
        
                <h4>🎬 Micro Roteiro (Vídeo):</h4>
                <ol>
                                <li><strong>Cena:</strong> 1 - <strong>Descrição Visual:</strong> Close-up no rosto de uma pessoa (ator/atriz) com expressão de cansaço e frustração, olhando para a tela de um notebook. - <strong>Fala:</strong> </li>
                    <li><strong>Cena:</strong> 2 - <strong>Descrição Visual:</strong> Corte rápido mostrando a pessoa digitando freneticamente, depois olhando para o relógio na parede, que gira rápido. - <strong>Fala:</strong> </li>
                    <li><strong>Cena:</strong> 3 - <strong>Descrição Visual:</strong> A pessoa posta no celular. A tela do celular mostra a postagem com pouquíssimas curtidas (ex: 3 likes). A pessoa suspira, desapontada. - <strong>Fala:</strong> </li>
                    <li><strong>Cena:</strong> 4 - <strong>Descrição Visual:</strong> A imagem se transforma em uma animação de um hamster correndo em uma roda, com ícones de redes sociais ao redor. - <strong>Fala:</strong> </li>
                    <li><strong>Cena:</strong> 5 - <strong>Descrição Visual:</strong> A pessoa para de correr, olha para a câmera com um pequeno sorriso de esperança. - <strong>Fala:</strong> </li>

                </ol>
            
                <p class="subtitulo-grupo4"><strong>Prompt para IA Geradora de Imagens:</strong> A hyper-realistic video. A woman in her 30s, sitting in a cozy home office, looking tired and uninspired at her laptop. Quick cuts: her typing, deleting, sighing. A shot of her phone screen showing a post with only 2 likes. Transition to a stylized animation of a hamster wheel with social media icons. The final shot is her looking up with a glimmer of hope. The lighting starts dim and gets brighter at the end.</p>
                <p class="subtitulo-grupo4"><strong>🧪 Testes A/B:</strong> Gancho do vídeo → Iniciar com a cena da frustração vs Iniciar com a animação da roda do hamster → Métrica de sucesso: Taxa de retenção do vídeo (watch time).</p>
                <p class="subtitulo-grupo4"><strong>⚠️ Como Corrigir a Rota:</strong> 1) Alerta: Taxa de engajamento < 2% em 24h. 2) Ação: Compartilhar o Reel nos stories com a caixa de perguntas 'Qual sua maior frustração com conteúdo?'. 3) Sucesso: Mais de 50 comentários com 'EU' e alta taxa de salvamentos (>1% dos alcançados).</p>
                <p class="subtitulo-grupo4"><strong>🚀 Checklist de Publicação:</strong></p>
                <ul class="checklist">
                    <li>Revisar texto e gramática.</li>
                    <li>Verificar a qualidade da imagem/vídeo.</li>
                    <li>Confirmar o agendamento.</li>
                    <li>Responder a comentários e mensagens.</li>
                </ul>
            </div>
        
            <div class="post-section" id="post-3">
                <h3>Post #3: E se você tivesse um estrategista de conteúdo 24/7?</h3>
               <p class="subtitulo-grupo1"><strong>Tema:</strong> Oferta: Apresentar a Fluxo Criativo como a solução.</p>
                <p class="subtitulo-grupo1"><strong>Justificativa Estratégica:</strong> Este é o post de revelação. Após construir a consciência do problema e a empatia, apresentamos a Fluxo Criativo como a solução lógica e desejável. O formato de carrossel é ideal para explicar o 'como funciona' de forma didática e visual. O foco é nos benefícios (economia de tempo, escala de resultados) e nos diferenciais ('automação criativa', 'criação inteligente'), usando o vocabulário do cliente para gerar reconhecimento e confiança.</p>
                <p class="subtitulo-grupo1"><strong>Micro Briefing:</strong> Apresentar a Fluxo Criativo, explicando seu funcionamento e principais benefícios de forma clara e inspiradora.</p>
                <p class="subtitulo-grupo2"><strong>✍️ Legenda Principal:</strong> Lembra da frustração da Laura e da 'Roda do Hamster'? Agora, imagine este cenário:</p>
                <details>
                    <summary style="text-decoration: underline; font-style: italic; color: #0000EE;">Clique para expandir variações</summary>
                    <p><strong>📝 Variações:</strong></p>
                    <ul>
                            <li>A paralisia da tela em branco acabou. Apresentamos o Fluxo Criativo: sua nova plataforma de criação de conteúdo estratégico. Funciona assim: você nos dá o seu conhecimento, e nós o transformamos em posts personalizados que fortalecem sua autoridade. É a união perfeita da sua expertise com a nossa tecnologia. Simples, rápido e inteligente.</li>
                    <li>E se criar o conteúdo da semana levasse o tempo de um café? Com o Fluxo Criativo, isso é possível. Nossa plataforma não é só um gerador de texto. É um assistente estratégico que entende seu público e seus objetivos, criando posts que realmente funcionam. Diga adeus ao 'adivinhar' o que postar e olá para a criação de conteúdo com propósito.</li>

                    </ul>
                </details>
                <p class="subtitulo-grupo2"><strong># Hashtags:</strong> #FerramentaDeMarketing #AutomacaoCriativa #MarketingAutomatizado #ConteudoInteligente #FluxoCriativo</p>
                <p class="subtitulo-grupo2"><strong>Indicador Principal:</strong> Cliques no Link da Bio</p>
                <p class="subtitulo-grupo3"><strong>📢 Chamada para Ação:</strong> Clique no link da bio para descobrir como a criação inteligente pode transformar sua rotina.</p>
                <p class="subtitulo-grupo3"><strong>💡 Sugestões de Interação/Engajamento:</strong> Qual funcionalidade você mais gostaria de experimentar primeiro? A geração de legendas, os roteiros de Reels ou as ideias para carrosséis?</p>
        
                <details>
                    <summary style="text-decoration: underline; font-style: italic; color: #0000EE;">Clique para expandir roteiro de respostas</summary>
                    <p><strong>🗨️ Roteiro de Respostas:</strong></p>
                    <ul>
                                <li><strong>Comentário Genérico:</strong> Uau, parece incrível! Quanto custa?<br>
                        <strong>Resposta Sugerida:</strong> Que bom que gostou! Temos alguns planos que se adaptam a cada momento do seu negócio. O melhor é que você pode experimentar e ver o poder da ferramenta na prática! Clica no link da bio pra conferir tudo. 😉</li>
                    <li><strong>Comentário Negativo:</strong> Tenho medo de ficar tudo muito robótico e sem a minha cara.<br>
                        <strong>Resposta para Negativo:</strong> Essa é a nossa maior preocupação também! Por isso, o Fluxo Criativo foi desenvolvido para ser um ponto de partida estratégico. Ele te dá 80% do caminho, com textos e ideias de alta qualidade, e você entra com seus 20% de personalidade e toque final. É o melhor dos dois mundos!</li>
                    <li><strong>Comentário Genérico:</strong> Isso funciona para o nicho de [ex: terapia holística]?<br>
                        <strong>Resposta Sugerida:</strong> Com certeza! A plataforma é ideal para qualquer profissional que baseia seu negócio em conhecimento e autoridade. Você insere os temas e a linguagem do seu nicho, e a IA cria o conteúdo estratégico para você. Vai adorar!</li>
                    <li><strong>Comentário Negativo:</strong> Já testei outras IAs e não gostei.<br>
                        <strong>Resposta para Negativo:</strong> Entendemos o ceticismo. Muitas IAs são genéricas. O nosso diferencial é o foco: somos especialistas em conteúdo para redes sociais com o objetivo de construir autoridade para solopreneurs. Te convidamos a fazer um teste para ver a diferença na prática. O link está na bio!</li>

                    </ul>
                </details>
            
                <p class="subtitulo-grupo4"><strong>Sugestões Visuais:</strong> Carrossel com design moderno e limpo, explicando o passo a passo de como a plataforma Fluxo Criativo funciona, usando ícones e mockups.</p>
                <p class="subtitulo-grupo4"><strong>Texto na Imagem/Vídeo:</strong> Sua máquina de conteúdo estratégico chegou.</p>
                <p class="subtitulo-grupo4"><strong>Formato Sugerido:</strong> Carrossel de imagens (educativo/demonstrativo)</p>
        
        
                <h4>🎞️ Slides do Carrossel:</h4>
                <ol>
                                <li><strong>Capa</strong>: Saia da Roda do Hamster. Entre no Fluxo Criativo. (Visual: Uma imagem dividida: de um lado, a roda de hamster em preto e branco. Do outro, um foguete decolando em cores vibrantes, com o logo do Fluxo Criativo.)</li>
                    <li><strong>Passo 1: Sua Ideia Genial</strong>: Tudo começa com você. Insira um tema, uma dor do seu cliente ou um insight que você teve. (Visual: Um ícone de uma lâmpada brilhante e um campo de texto simples com a frase 'Como superar a procrastinação'.)</li>
                    <li><strong>Passo 2: A Mágica Acontece</strong>: Nossa IA estratégica analisa seu objetivo e transforma sua ideia em múltiplos formatos de conteúdo. (Visual: Uma animação gráfica mostrando a 'lâmpada' entrando em uma 'caixa' com o logo do Fluxo Criativo e várias setas saindo dela.)</li>
                    <li><strong>Passo 3: Posts Prontos em Minutos</strong>: Receba legendas, roteiros para Reels, ideias para carrosséis... tudo personalizado para a sua marca. (Visual: Mockups de tela de celular mostrando diferentes tipos de posts (um carrossel, um Reel) já prontos.)</li>
                    <li><strong>O Resultado: Mais Tempo e Autoridade</strong>: Recupere seu tempo para focar no que importa: seus clientes. E veja sua autoridade decolar com conteúdo de alto impacto. (Visual: Uma pessoa sorrindo, relaxada, com um gráfico de crescimento de engajamento ao fundo.)</li>
                    <li><strong>CTA</strong>: Curioso(a) para ver como funciona? O link para conhecer mais está na nossa bio! (Visual: Um botão grande e chamativo com o texto 'Quero Conhecer' e uma seta apontando para a bio.)</li>

                </ol>
            
                <p class="subtitulo-grupo4"><strong>Prompt para IA Geradora de Imagens:</strong> A series of clean, modern infographic-style carousel slides. Use a consistent color palette of deep blue, bright magenta, and white. Each slide uses bold typography and simple icons to illustrate the process: a lightbulb for an idea, a brain with gears for the AI, and phone mockups for the results. The overall feeling is innovative, efficient, and empowering.</p>
                <p class="subtitulo-grupo4"><strong>🧪 Testes A/B:</strong> Título do Carrossel → 'E se você tivesse um estrategista 24/7?' vs 'Transforme 1 ideia em 10 posts em 5 minutos.' → Métrica de sucesso: Taxa de conclusão do carrossel (swipe-through rate).</p>
                <p class="subtitulo-grupo4"><strong>⚠️ Como Corrigir a Rota:</strong> 1) Alerta: CTR < 1% nas primeiras 12h. 2) Ação: Adicionar um sticker de link nos stories apontando para a bio e explicando o carrossel. 3) Sucesso: CTR > 3% e um aumento notável no tráfego para a landing page.</p>
                <p class="subtitulo-grupo4"><strong>🚀 Checklist de Publicação:</strong></p>
                <ul class="checklist">
                    <li>Revisar texto e gramática.</li>
                    <li>Verificar a qualidade da imagem/vídeo.</li>
                    <li>Confirmar o agendamento.</li>
                    <li>Responder a comentários e mensagens.</li>
                </ul>
            </div>
        
            <div class="post-section" id="post-4">
                <h3>Post #4: Um presente para quem age rápido.</h3>
               <p class="subtitulo-grupo1"><strong>Tema:</strong> Urgência / Escassez: Incentivar a ação imediata com um bônus.</p>
                <p class="subtitulo-grupo1"><strong>Justificativa Estratégica:</strong> Este post introduz o elemento de urgência. Após apresentar a solução, damos um motivo forte para o usuário agir AGORA, e não depois. O bônus 'Kit de Autoridade Acelerada' é altamente relevante para o público-alvo e agrega um valor tangível à oferta do teste. Limitar a 100 pessoas cria uma escassez real (ou percebida) que combate a procrastinação e acelera a decisão de clicar e se cadastrar.</p>
                <p class="subtitulo-grupo1"><strong>Micro Briefing:</strong> Criar um senso de urgência e aumentar o valor percebido da oferta de teste através de um bônus exclusivo e limitado.</p>
                <p class="subtitulo-grupo2"><strong>✍️ Legenda Principal:</strong> Nós vimos o quanto vocês se identificaram com o desafio de criar conteúdo. E para ajudar os solopreneurs que estão realmente comprometidos em dar o próximo passo, preparamos algo especial.</p>
                <details>
                    <summary style="text-decoration: underline; font-style: italic; color: #0000EE;">Clique para expandir variações</summary>
                    <p><strong>📝 Variações:</strong></p>
                    <ul>
                            <li>Você quer apenas testar uma ferramenta ou quer um sistema completo para decolar sua autoridade? Para os 100 primeiros a se cadastrarem no Fluxo Criativo, não vamos entregar apenas o acesso, mas também nosso Kit de Autoridade Acelerada. É o nosso empurrãozinho para você começar com tudo. Vagas limitadas. Garanta o seu.</li>
                    <li>Imagine começar a usar o Fluxo Criativo e já ter um arsenal de conteúdo validado nas mãos. É isso que nosso bônus exclusivo oferece. Mas é só para os 100 primeiros. Se você está esperando o momento perfeito para otimizar sua criação de conteúdo, o momento é este.</li>

                    </ul>
                </details>
                <p class="subtitulo-grupo2"><strong># Hashtags:</strong> #Oportunidade #BonusExclusivo #MarketingParaCoaches #VagasLimitadas #CrescimentoAcelerado</p>
                <p class="subtitulo-grupo2"><strong>Indicador Principal:</strong> Taxa de Conversão na Landing Page (inícios de teste)</p>
                <p class="subtitulo-grupo3"><strong>📢 Chamada para Ação:</strong> Seja um dos 100 primeiros! Clique no link da bio e garanta seu teste + bônus agora!</p>
                <p class="subtitulo-grupo3"><strong>💡 Sugestões de Interação/Engajamento:</strong> Marque um amigo(a) que não pode perder essa oportunidade de acelerar os resultados!</p>
        
                <details>
                    <summary style="text-decoration: underline; font-style: italic; color: #0000EE;">Clique para expandir roteiro de respostas</summary>
                    <p><strong>🗨️ Roteiro de Respostas:</strong></p>
                    <ul>
                                <li><strong>Comentário Genérico:</strong> Já quero! Como faço pra garantir o meu?<br>
                        <strong>Resposta Sugerida:</strong> Que demais! É só correr no link da nossa bio, iniciar seu teste e o acesso ao bônus será enviado para você. Mas não demora, são só para os 100 primeiros! 😉</li>
                    <li><strong>Comentário Negativo:</strong> Isso é só marketing pra gente se cadastrar logo.<br>
                        <strong>Resposta para Negativo:</strong> É uma forma de incentivar quem está mais decidido, sim! E também uma forma de agradecermos a confiança dos primeiros que acreditam no nosso projeto. O bônus é um material de altíssimo valor que realmente vai te ajudar a ter resultados mais rápidos.</li>
                    <li><strong>Comentário Genérico:</strong> O teste é pago?<br>
                        <strong>Resposta Sugerida:</strong> Você pode experimentar a plataforma para ver todo o potencial dela! Todos os detalhes sobre o período de teste e os planos estão explicadinhos na página do link da nossa bio. Vai lá conferir!</li>
                    <li><strong>Comentário Negativo:</strong> Esse bônus vai estar disponível depois?<br>
                        <strong>Resposta para Negativo:</strong> Este kit específico foi criado como um presente de agradecimento para os 100 primeiros a iniciarem o teste nesta ação. Não temos previsão de oferecê-lo novamente, então a hora de garantir é agora!</li>

                    </ul>
                </details>
            
                <p class="subtitulo-grupo4"><strong>Sugestões Visuais:</strong> Imagem com visual premium de uma caixa de presente estilizada ao lado de mockups de um guia digital e templates, com um texto forte sobreposto anunciando o bônus exclusivo.</p>
                <p class="subtitulo-grupo4"><strong>Texto na Imagem/Vídeo:</strong> BÔNUS EXCLUSIVO: Para os 100 primeiros. Não perca.</p>
                <p class="subtitulo-grupo4"><strong>Formato Sugerido:</strong> Imagem única com texto forte</p>
        
        
                <p class="subtitulo-grupo4"><strong>Prompt para IA Geradora de Imagens:</strong> A striking and elegant image. A sleek, modern gift box in dark blue with magenta ribbons, subtly glowing. Next to it, mockups of a digital guide and content templates. The background is clean and slightly out of focus. The overall mood is exclusive, valuable, and urgent. Minimalist and premium feel.</p>
                <p class="subtitulo-grupo4"><strong>🧪 Testes A/B:</strong> Texto na imagem → 'Bônus Exclusivo: Para os 100 primeiros' vs 'Seu Kit de Autoridade Acelerada te espera' → Métrica de sucesso: Taxa de cliques (CTR).</p>
                <p class="subtitulo-grupo4"><strong>⚠️ Como Corrigir a Rota:</strong> 1) Alerta: Menos de 20 conversões nas primeiras 24h. 2) Ação: Rodar um anúncio de tráfego para este post, segmentando para a audiência que mais se engajou com os posts anteriores. 3) Sucesso: Atingir a meta de 100 conversões ou um aumento de 200% na taxa de conversão diária.</p>
                <p class="subtitulo-grupo4"><strong>🚀 Checklist de Publicação:</strong></p>
                <ul class="checklist">
                    <li>Revisar texto e gramática.</li>
                    <li>Verificar a qualidade da imagem/vídeo.</li>
                    <li>Confirmar o agendamento.</li>
                    <li>Responder a comentários e mensagens.</li>
                </ul>
            </div>
        
            <div class="post-section" id="post-5">
                <h3>Post #5: Sua escolha: continuar correndo ou começar a voar?</h3>
               <p class="subtitulo-grupo1"><strong>Tema:</strong> CTA Final: Reforçar a transformação e levar à ação.</p>
                <p class="subtitulo-grupo1"><strong>Justificativa Estratégica:</strong> Este post é o fechamento da campanha. Ele recapitula a jornada da dor à solução e posiciona a decisão de se cadastrar como uma escolha clara entre o 'velho jeito' e o 'novo jeito'. A linguagem é direta, empoderadora e focada na ação. O objetivo não é mais educar, mas sim converter, levando o usuário qualificado e nutrido ao longo da semana a tomar a ação final de clicar no link e iniciar o teste.</p>
                <p class="subtitulo-grupo1"><strong>Micro Briefing:</strong> Concluir a narrativa da campanha, reforçar a proposta de valor e direcionar todo o foco para a conversão final através de uma CTA forte.</p>
                <p class="subtitulo-grupo2"><strong>✍️ Legenda Principal:</strong> Nesta semana, falamos sobre as horas perdidas, a frustração de postar sem estratégia e a sensação de estar na 'Roda do Hamster'.</p>
                <details>
                    <summary style="text-decoration: underline; font-style: italic; color: #0000EE;">Clique para expandir variações</summary>
                    <p><strong>📝 Variações:</strong></p>
                    <ul>
                            <li>Você pode passar o resto do mês pensando no que postar, ou pode usar os próximos 15 minutos para se cadastrar e ter ideias para o mês inteiro. A escolha entre esforço e estratégia está a um clique de distância. O link na bio é o seu primeiro passo para a automação criativa. Aja agora.</li>
                    <li>Recapitulando: mais tempo na sua agenda, fim do bloqueio criativo, posts que constroem autoridade. Essa é a promessa do Fluxo Criativo. Se isso soa como a transformação que seu negócio precisa, não espere mais. O momento de agir é agora. Clique no link da bio e comece sua jornada.</li>

                    </ul>
                </details>
                <p class="subtitulo-grupo2"><strong># Hashtags:</strong> #Decisao #TransformacaoDigital #FuturoDoMarketing #SejaEstrategico #CTA</p>
                <p class="subtitulo-grupo2"><strong>Indicador Principal:</strong> Cliques no Link da Bio e Taxa de Conversão Final</p>
                <p class="subtitulo-grupo3"><strong>📢 Chamada para Ação:</strong> Sua escala de resultados começa agora. Clique no link da bio para destravar seu potencial.</p>
                <p class="subtitulo-grupo3"><strong>💡 Sugestões de Interação/Engajamento:</strong> Não há pergunta de interação neste post, o foco é 100% no CTA. A interação esperada é o clique.</p>
        
                <details>
                    <summary style="text-decoration: underline; font-style: italic; color: #0000EE;">Clique para expandir roteiro de respostas</summary>
                    <p><strong>🗨️ Roteiro de Respostas:</strong></p>
                    <ul>
                                <li><strong>Comentário Genérico:</strong> Me cadastrei! Ansiosa pra começar!<br>
                        <strong>Resposta Sugerida:</strong> Que notícia incrível! 🎉 Seja muito bem-vindo(a) ao Fluxo Criativo! Temos certeza que você vai adorar. Qualquer dúvida, nosso suporte está à disposição!</li>
                    <li><strong>Comentário Negativo:</strong> Ainda estou na dúvida se é pra mim.<br>
                        <strong>Resposta para Negativo:</strong> A melhor forma de tirar a dúvida é vendo na prática! O teste serve exatamente pra isso: para você explorar a ferramenta, ver o tipo de conteúdo que ela gera para o seu negócio e decidir com segurança. Se não for o que você espera, não há compromisso. Que tal dar uma espiada? 😉</li>
                    <li><strong>Comentário Genérico:</strong> Até quando vai o bônus?<br>
                        <strong>Resposta Sugerida:</strong> O bônus é para os 100 primeiros que se cadastrarem, então é por ordem de chegada! Não podemos garantir até quando estará disponível, por isso recomendamos garantir o seu o quanto antes. O link está na bio!</li>
                    <li><strong>Comentário Negativo:</strong> Não tenho dinheiro pra investir nisso agora.<br>
                        <strong>Resposta para Negativo:</strong> Entendemos perfeitamente a questão do investimento. Pense no valor do seu tempo. Se a ferramenta te economizar 8h/semana, quanto isso vale? Esse tempo pode ser usado para conseguir um novo cliente que, sozinho, já pagaria o investimento por meses. Comece pelo teste e veja o potencial de retorno. 😊</li>

                    </ul>
                </details>
            
                <p class="subtitulo-grupo4"><strong>Sugestões Visuais:</strong> Vídeo de alto impacto que contrasta o 'antes' (frustração, roda de hamster) com o 'depois' (calma, resultados, satisfação) com o Fluxo Criativo, terminando com uma chamada para ação direta.</p>
                <p class="subtitulo-grupo4"><strong>Texto na Imagem/Vídeo:</strong> A decisão é sua. O link está na bio.</p>
                <p class="subtitulo-grupo4"><strong>Formato Sugerido:</strong> Vídeo curto (Reel) ou Imagem com Citação Forte</p>
        
        
                <h4>🎬 Micro Roteiro (Vídeo):</h4>
                <ol>
                                <li><strong>Cena:</strong> 1 - <strong>Descrição Visual:</strong> Cena rápida da roda de hamster do post 2, em preto e branco. - <strong>Fala:</strong> </li>
                    <li><strong>Cena:</strong> 2 - <strong>Descrição Visual:</strong> Transição rápida para uma cena colorida e vibrante de uma pessoa (a mesma dos outros vídeos) trabalhando de forma calma e focada, sorrindo. - <strong>Fala:</strong> </li>
                    <li><strong>Cena:</strong> 3 - <strong>Descrição Visual:</strong> Close na tela do notebook mostrando a interface do Fluxo Criativo, com vários posts gerados. - <strong>Fala:</strong> </li>
                    <li><strong>Cena:</strong> 4 - <strong>Descrição Visual:</strong> A pessoa fecha o notebook, pega uma xícara de café e olha pela janela, com ar de satisfação e dever cumprido. - <strong>Fala:</strong> </li>
                    <li><strong>Cena:</strong> 5 - <strong>Descrição Visual:</strong> Tela final com o logo do Fluxo Criativo e uma chamada para ação clara. - <strong>Fala:</strong> </li>

                </ol>
            
                <p class="subtitulo-grupo4"><strong>Prompt para IA Geradora de Imagens:</strong> A short, impactful video. Starts with a desaturated clip of the hamster wheel. Quick transition to a vibrant shot of a solopreneur smiling, closing their laptop with satisfaction. A super-fast screen recording of the Fluxo Criativo dashboard in action. Ends with a powerful shot of the person looking confidently at the camera. Upbeat, motivational background music. Text overlays are bold and clear.</p>
                <p class="subtitulo-grupo4"><strong>🧪 Testes A/B:</strong> Formato → Vídeo Reel vs Imagem Estática com Citação Forte → Métrica de sucesso: Taxa de Conversão (Cliques que resultam em cadastro).</p>
                <p class="subtitulo-grupo4"><strong>⚠️ Como Corrigir a Rota:</strong> 1) Alerta: CTR < 2% e taxa de conversão da campanha abaixo da meta. 2) Ação: Criar um Story urgente no fim do dia: 'Últimas horas para garantir seu bônus! Restam X vagas'. 3) Sucesso: Pico de cliques e conversões, atingindo a meta semanal de novos usuários.</p>
                <p class="subtitulo-grupo4"><strong>🚀 Checklist de Publicação:</strong></p>
                <ul class="checklist">
                    <li>Revisar texto e gramática.</li>
                    <li>Verificar a qualidade da imagem/vídeo.</li>
                    <li>Confirmar o agendamento.</li>
                    <li>Responder a comentários e mensagens.</li>
                </ul>
            </div>
        
        
                <h2>Checklist de Publicação</h2>
        <ul class="checklist">
            <li><strong>15/10</strong></li>
            <ul>
                <li>Preparar Post #1: Sua semana de trabalho tem horas suficientes?</li>
            </ul>
            <li><strong>16/10</strong></li>
            <ul>
                <li>Preparar Post #2: A Roda do Hamster de Conteúdo: Você está nela?</li>
            </ul>
            <li><strong>17/10</strong></li>
            <ul>
                <li><strong>Postar Post #1: Sua semana de trabalho tem horas suficientes?</strong></li>
                <li>Preparar Post #3: E se você tivesse um estrategista de conteúdo 24/7?</li>
            </ul>
            <li><strong>18/10</strong></li>
            <ul>
                <li><strong>Postar Post #2: A Roda do Hamster de Conteúdo: Você está nela?</strong></li>
                <li>Preparar Post #4: Um presente para quem age rápido.</li>
            </ul>
            <li><strong>19/10</strong></li>
            <ul>
                <li>Responder comentários Post #1: Sua semana de trabalho tem horas suficientes?</li>
                <li><strong>Postar Post #3: E se você tivesse um estrategista de conteúdo 24/7?</strong></li>
            </ul>
            <li><strong>20/10</strong></li>
            <ul>
                <li>Responder comentários Post #2: A Roda do Hamster de Conteúdo: Você está nela?</li>
                <li><strong>Postar Post #4: Um presente para quem age rápido.</strong></li>
                <li>Preparar Post #5: Sua escolha: continuar correndo ou começar a voar?</li>
            </ul>
            <li><strong>21/10</strong></li>
            <ul>
                <li>Responder 2ª vez comentários Post #1: Sua semana de trabalho tem horas suficientes?</li>
                <li>Responder comentários Post #3: E se você tivesse um estrategista de conteúdo 24/7?</li>
            </ul>
            <li><strong>22/10</strong></li>
            <ul>
                <li>Responder 2ª vez comentários Post #2: A Roda do Hamster de Conteúdo: Você está nela?</li>
                <li>Responder comentários Post #4: Um presente para quem age rápido.</li>
                <li><strong>Postar Post #5: Sua escolha: continuar correndo ou começar a voar?</strong></li>
            </ul>
            <li><strong>23/10</strong></li>
            <ul>
                <li>Responder 2ª vez comentários Post #3: E se você tivesse um estrategista de conteúdo 24/7?</li>
            </ul>
            <li><strong>24/10</strong></li>
            <ul>
                <li>Responder 2ª vez comentários Post #4: Um presente para quem age rápido.</li>
                <li>Responder comentários Post #5: Sua escolha: continuar correndo ou começar a voar?</li>
            </ul>
            <li><strong>26/10</strong></li>
            <ul>
                <li>Responder 2ª vez comentários Post #5: Sua escolha: continuar correndo ou começar a voar?</li>
            </ul>
        </ul>


            <div class="footer">
                <p>&copy; 2025 Conteúdo gerado por Fluxo Criativo. Todos os direitos reservados.</p>
            </div>
        </div>
    </body>
</html>
    "