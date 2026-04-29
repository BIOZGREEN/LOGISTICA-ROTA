#=============================================
# PLANEJAMENTO DE ROTA
# NOME: README.md
# DESCRIÇÃO: Documentação principal do projeto com instruções de instalação, uso e configuração
# AUTOR: SILVANO MORAES DE SOUZA
# VERSÃO: 1.0
#=============================================
# Bioz Logística - Otimizador de Rotas

Sistema completo para otimização de rotas logísticas com interface moderna e integração Google Maps.

## Funcionalidades

- ✅ Upload de PDFs e CSVs com dados de entrega
- ✅ Drag & Drop de arquivos
- ✅ Cálculo automático de rotas otimizadas (8h-16h, saindo de Sorocaba)
- ✅ Integração com Google Maps API (distâncias reais)
- ✅ Cálculo de economia (gasolina + horas extras)
- ✅ Exportação de rotas para CSV
- ✅ Visualização da rota completa no Maps
- ✅ Interface responsiva com Tailwind CSS

## Configuração

### 1. Instalar dependências
```bash
npm install
```

### 2. Configurar Google Maps API Key
Crie um arquivo `.env` baseado no `.env.example`:

```bash
cp .env.example .env
```

Edite o `.env` e adicione sua chave da API do Google Maps:
```
GOOGLE_MAPS_API_KEY=sua_chave_aqui
```

**Como obter a chave:**
1. Acesse [Google Cloud Console](https://console.cloud.google.com/)
2. Crie um projeto ou selecione existente
3. Ative as APIs: `Maps JavaScript API`, `Directions API`, `Distance Matrix API`
4. Crie credenciais (API Key)
5. Configure restrições de HTTP referrer para segurança

### 3. Executar localmente
```bash
node server.js
```

Acesse: http://localhost:3001

## Testando Localmente

1. Instale as dependências:
```bash
npm install
```

2. Crie o arquivo `.env` (opcional - sem ele usa dados simulados):
```
GOOGLE_MAPS_API_KEY=sua_chave_aqui
```

3. Execute o servidor:
```bash
node server.js
```
Ou use o arquivo `start.bat` (duplo clique).

4. Acesse no navegador:
```
http://localhost:3001
```

## Estrutura do Projeto

```
├── index.html          # Frontend (HTML + Tailwind + JS)
├── server.js           # Servidor Express local
├── api/
│   ├── process.js      # Processamento de PDFs/CSV
│   └── optimize.js    # Otimização de rotas (Google Maps)
├── package.json
├── vercel.json        # Configuração para deploy Vercel
└── .env               # Variáveis de ambiente (não commitar)
```

## Deploy na Vercel

1. Instale a CLI da Vercel:
```bash
npm i -g vercel
```

2. Faça login:
```bash
vercel login
```

3. Deploy:
```bash
vercel --prod
```

Ou conecte o repositório Git ao [Vercel Dashboard](https://vercel.com/dashboard) para deploy automático.

**Importante:** Adicione a `GOOGLE_MAPS_API_KEY` nas variáveis de ambiente da Vercel.

## Parâmetros da Rota

- **Origem:** Sorocaba, SP
- **Horário:** 08:00 às 16:00 (8h limite)
- **Velocidade média:** 60 km/h
- **Tempo por parada:** 10 minutos
- **Custo gasolina:** R$ 0,60/km (R$ 6,00/L a 10km/L)

## Solução de Problemas

**Erro: "Failed to fetch"**
- Verifique se o servidor está rodando: acesse http://localhost:3001
- Se a porta estiver em uso, mate o processo: `netstat -ano | findstr :3001` e depois `taskkill /PID <número> /F`

**Erro: EADDRINUSE**
- A porta já está em uso. Mude a porta no `server.js` ou finalize o processo anterior.

## Formato dos Arquivos

### CSV
O sistema procura automaticamente por colunas com nomes:
- `endereco`, `endereço`, `address`, `logradouro`, `rua`, `local`

Exemplo:
```csv
endereco,cidade,estado
"Av. Paulista, 1000",São Paulo,SP
"Rua Augusta, 500",São Paulo,SP
```

### PDF
Extrai automaticamente endereços usando regex (formatos comuns de endereços brasileiros).

## Tecnologias

- **Frontend:** HTML5, Tailwind CSS, Vanilla JavaScript
- **Backend:** Node.js, Express
- **Processamento:** pdf-parse, csv-parser, multer
- **APIs:** Google Maps (Directions, Distance Matrix)
- **Deploy:** Vercel

## Licença

MIT - Bioz Logística S.A.
