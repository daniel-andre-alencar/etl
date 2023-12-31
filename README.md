# etl
import pandas as pd
import requests
import json
import openai
import random  # Importe o módulo random para escolher frases aleatórias

# Configure a sua chave de API do OpenAI
openai.api_key = "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"

# URL da API SDW2023 (substitua pela URL real)
sdw2023_api_url = 'https://exemplo.com/api'

# Passo 1: Ler o CSV e obter IDs de usuário
df = pd.read_csv('SDW2023.csv')
user_ids = df['UserID'].tolist()

# Frases adicionais para uso no modelo de linguagem GPT-4
frases_adicionais = [
    "Lembre-se sempre de diversificar seus investimentos.",
    "Investir com sabedoria é o segredo do sucesso financeiro.",
    "Aqui estão algumas dicas para investir com inteligência: [Dicas de investimento].",
    "O mercado financeiro está em constante evolução, fique atualizado.",
    "Invista hoje para colher os frutos no futuro.",
]

# Passo 2: Definir uma função para obter dados do usuário de uma API
def obter_dados_do_usuario(id):
    response = requests.get(f'{sdw2023_api_url}/usuarios/{id}')
    return response.json() if response.status_code == 200 else None

# Passo 3: Buscar dados do usuário e filtrar usuários válidos
usuarios = [usuario for id in user_ids if (usuario := obter_dados_do_usuario(id)) is not None]

# Passo 4: Definir uma função para gerar notícias de IA
def gerar_noticias_de_ia(usuario):
    # Escolha uma das frases adicionais aleatoriamente
    frase_adicional = random.choice(frases_adicionais)

    completamento = openai.ChatCompletion.create(
        model="gpt-4",
        messages=[
            {
                "role": "system",
                "content": "Você é um especialista em marketing bancário."
            },
            {
                "role": "user",
                "content": f"Crie uma mensagem para {usuario['nome']} sobre a importância dos investimentos (máximo de 100 caracteres)"
            },
            {
                "role": "assistant",
                "content": frase_adicional  # Adicione a frase adicional aqui
            }
        ]
    )
    return completamento.choices[0].message.content.strip('\"')

# Passo 5: Gerar notícias para cada usuário e anexar aos dados do usuário
for usuario in usuarios:
    noticia = gerar_noticias_de_ia(usuario)
    print(noticia)
    if 'noticias' not in usuario:
        usuario['noticias'] = []
    usuario['noticias'].append({
        "icone": "https://digitalinnovationone.github.io/santander-dev-week-2023-api/icons/credit.svg",
        "descricao": noticia
    })

# Passo 6: Salvar os dados do usuário atualizados (se necessário)
# Você pode optar por salvar a lista 'usuarios' em um arquivo ou atualizá-la em seu banco de dados.
# Exemplo: json.dump(usuarios, open('usuarios_atualizados.json', 'w'), indent=2)
