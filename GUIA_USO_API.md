# 📘 Guia Inicial: Como Usar e Integrar APIs no Seu Código

> Este guia foi escrito para quem está começando com APIs. Tudo está explicado de forma simples, com exemplos práticos que você pode copiar e testar.

---

## 1. O que é uma API?

Uma **API** (Application Programming Interface) é como um "garçom" entre dois programas.

- **Você** (cliente) faz um pedido para a API.
- A **API** leva esse pedido para o servidor, busca a informação e traz de volta para você.

Exemplo do dia a dia: quando você usa um app de clima, o app consulta uma API que devolve a temperatura da sua cidade.

---

## 2. Conceitos essenciais (vocabulário básico)

| Termo | O que significa |
|-------|-----------------|
| **Endpoint** | O endereço (URL) onde a API fica. Exemplo: `https://api.exemplo.com/usuarios` |
| **Requisição** | A mensagem que você envia para a API (pedido de dados, envio de dados, etc.) |
| **Resposta** | O que a API devolve para você depois de processar o pedido |
| **Método HTTP** | A ação que você quer fazer: `GET` (buscar), `POST` (enviar/criar), `PUT` (atualizar), `DELETE` (apagar) |
| **Status code** | Código numérico que diz se deu certo ou erro. Ex: `200` = ok, `404` = não encontrado, `500` = erro no servidor |
| **JSON** | Formato de texto usado para trocar dados. Parece com um objeto/dicionário: `{"nome": "Ana", "idade": 25}` |

---

## 3. Passo a passo para integrar uma API ao seu código

1. **Escolha a API** e leia a documentação dela (saberá o endpoint, se precisa de senha/token e que dados enviar).
2. **Monte a requisição**: URL + método (GET, POST...) + headers (cabeçalhos) + corpo (quando for enviar dados).
3. **Envie a requisição** com a ferramenta da sua linguagem (ex: `fetch` no JavaScript, `requests` no Python).
4. **Trate a resposta**: verifique o status code e leia os dados retornados (geralmente em JSON).
5. **Trate erros**: sempre preveja o que acontece se a internet cair ou a API retornar erro.

---

## 4. Exemplo prático 1 — JavaScript (navegador ou Node.js)

Vamos consumir uma API pública gratuita ([JSONPlaceholder](https://jsonplaceholder.typicode.com/)) que retorna posts fictícios.

### No navegador (HTML + JS)

```html
<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <title>Meu primeiro consumo de API</title>
</head>
<body>
  <h1>Posts da API</h1>
  <button onclick="carregarPosts()">Carregar posts</button>
  <div id="resultado"></div>

  <script>
    async function carregarPosts() {
      const endpoint = 'https://jsonplaceholder.typicode.com/posts';

      try {
        // 1. Faz a requisição GET para a API
        const resposta = await fetch(endpoint);

        // 2. Verifica se a resposta foi bem-sucedida
        if (!resposta.ok) {
          throw new Error('Erro na API: ' + resposta.status);
        }

        // 3. Converte a resposta para JSON (objeto JavaScript)
        const dados = await resposta.json();

        // 4. Exibe os 3 primeiros posts na tela
        const container = document.getElementById('resultado');
        container.innerHTML = '';

        dados.slice(0, 3).forEach(post => {
          container.innerHTML += `
            <div style="border:1px solid #ccc; padding:10px; margin:10px 0;">
              <strong>${post.title}</strong><br>
              ${post.body}
            </div>
          `;
        });
      } catch (erro) {
        document.getElementById('resultado').textContent =
          'Erro ao carregar: ' + erro.message;
      }
    }
  </script>
</body>
</html>
```

### No Node.js (versão 18+ tem `fetch` nativo)

```javascript
// arquivo: consumir-api.js
async function main() {
  const endpoint = 'https://jsonplaceholder.typicode.com/posts/1';

  try {
    const resposta = await fetch(endpoint);

    if (!resposta.ok) {
      throw new Error(`Erro HTTP: ${resposta.status}`);
    }

    const post = await resposta.json();
    console.log('Título:', post.title);
    console.log('Corpo:', post.body);
  } catch (erro) {
    console.error('Falha ao consumir a API:', erro.message);
  }
}

main();
```

Para rodar no Node.js:
```bash
node consumir-api.js
```

---

## 5. Exemplo prático 2 — Python (biblioteca `requests`)

### Instale a biblioteca (se ainda não tiver)

```bash
pip install requests
```

### Código simples (GET)

```python
# arquivo: consumir_api.py
import requests

def buscar_usuario():
    url = "https://jsonplaceholder.typicode.com/users/1"

    try:
        resposta = requests.get(url, timeout=10)  # timeout evita travamento

        # Verifica se deu certo (código 200)
        if resposta.status_code == 200:
            dados = resposta.json()  # converte para dicionário Python
            print("Nome:", dados["name"])
            print("Email:", dados["email"])
            print("Cidade:", dados["address"]["city"])
        else:
            print(f"A API respondeu com erro: {resposta.status_code}")

    except requests.exceptions.RequestException as erro:
        print("Erro ao conectar com a API:", erro)

if __name__ == "__main__":
    buscar_usuario()
```

Para rodar:
```bash
python consumir_api.py
```

### Enviando dados para a API (POST)

Muitas APIs exigem que você envie informações (ex: criar um usuário). Veja como enviar JSON:

```python
import requests

url = "https://jsonplaceholder.typicode.com/posts"

novo_post = {
    "title": "Meu primeiro post",
    "body": "Aprendendo a usar APIs em Python",
    "userId": 1
}

try:
    resposta = requests.post(url, json=novo_post, timeout=10)

    if resposta.status_code in (200, 201):
        print("Criado com sucesso!")
        print(resposta.json())
    else:
        print(f"Erro: {resposta.status_code} - {resposta.text}")

except requests.exceptions.RequestException as erro:
    print("Erro na requisição:", erro)
```

---

## 6. Boas práticas e segurança

| Dica | Por que fazer |
|------|---------------|
| **Use `timeout`** | Evita que seu programa fique esperando para sempre se a API demorar ou cair. |
| **Trate erros com `try/catch` ou `try/except`** | Sua aplicação não trava se a internet cair ou a API retornar erro. |
| **Não coloque senhas/tokens direto no código** | Use variáveis de ambiente (`.env`) para chaves secretas. |
| **Leia o `Content-Type`** | Confirme que a resposta é JSON antes de tentar converter. |
| **Respeite limites (rate limit)** | Não envie centenas de requisições por segundo; a API pode bloquear seu IP. |

Exemplo de variável de ambiente em Python:

```python
import os
TOKEN = os.getenv("MINHA_CHAVE_API")  # defina no sistema antes de rodar
```

---

## 7. Checklist rápido antes de integrar

- [ ] Tenho o endpoint correto da API?
- [ ] Sei qual método usar (GET, POST, PUT, DELETE)?
- [ ] Preciso de autenticação (token, senha, chave de API)?
- [ ] Sei qual formato de dados enviar/receber (geralmente JSON)?
- [ ] Tratei erros de conexão e de resposta?
- [ ] Usei timeout para não travar o programa?

---

## 8. Próximos passos

1. Teste os exemplos acima (eles usam APIs públicas e gratuitas).
2. Procure uma API de um serviço que você usa (clima, CEP, pagamentos, etc.) e leia a documentação dela.
3. Crie um pequeno projeto que consome essa API e exibe os dados na sua aplicação.

---

> 💡 Dica: se quiser testar APIs sem escrever código, use ferramentas como **Postman**, **Insomnia** ou a extensão **Thunder Client** (VS Code). Elas permitem enviar requisições e ver as respostas de forma visual.
