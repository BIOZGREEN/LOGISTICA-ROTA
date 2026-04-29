# CIADOS - Registro de Atividades
## Sistema de Logística Efêmero - Roteirização de Notas Fiscais

**Data de Início:** 28/04/2026  
**Horário de Início:** Aproximadamente 16:00  
**Responsável:** Engenheiro de Software Sênior e Arquiteto de Dados

---

### 📅 28/04/2026 - Fase de Estruturação Inicial

**16:00 - Início do Projeto**
- Diretório de trabalho verificado e confirmado como vazio
- Iniciada a estruturação do sistema de logística efêmero conforme requisitos

**16:02 - Criação do config.py**
- Arquivo de configuração centralizado criado
- Definições incluídas:
  - Chave da API OpenRouteService (via variável de ambiente)
  - Localização base: Sorocaba (Rua XV de Novembro, 350, Centro)
  - Horário de trabalho: 08:00 às 16:30 (Segunda a Sexta)
  - Parâmetros: tempo médio de entrega (10 min), tempo de retorno/base (30 min), velocidade média (40 km/h)
  - Limite máximo de PDFs: 50 por lote

**16:04 - Criação do services/pdf_extractor.py**
- Módulo para extração de endereços de PDFs de Notas Fiscais
- Tecnologia utilizada: PyMuPDF (fitz)
- Funcionalidades implementadas:
  - Extração de texto diretamente do conteúdo binário (sem armazenamento em disco)
  - Padrões de regex para identificação de campos "Destinatário/Remetente"
  - Filtragem de resultados para eliminar ruídos e duplicatas
  - Processamento em lote com tratamento de erros individual por PDF
  - Retorno estruturado com endereço, página de origem e arquivo fonte

**16:05 - Criação do services/geocoder.py**
- Módulo de integração com OpenRouteService (API gratuita)
- Funcionalidades implementadas:
  - Classe OpenRouteServiceGeocoder com tratamento de chave API
  - Geocodificação individual de endereços com timeout e tratamento de exceções
  - Geocodificação em lote com rate limiting (pausa a cada 10 requisições)
  - Filtragem de resultados válidos vs inválidos
  - Instância global reutilizável para eficiência em ambiente serverless

**16:10 - Criação do services/route_optimizer.py**
- Núcleo do sistema de otimização com restrições de horário
- Algoritmos implementados:
  - Função haversine para cálculo de distâncias geográficas
  - Matriz de distâncias entre todos os pontos (base + entregas)
  - Algoritmo TSP guloso (Vizinho Mais Próximo) para otimização inicial
  - Simulação de rota com verificação de restrições de horário
  - Lógica crítica de divisão automática para múltiplos dias:
    * Verifica se retorno à base + conferência ocorre antes das 16:30
    * Quando excedido, divide otimamente entre dia atual e próximo dia útil
    * Pula automaticamente sábados e domingos
    * Recalcula rota otimizada para cada dia separado
  - Função principal processar_rotas() que orquestra todo o fluxo
  - Retorno detalhado com estatísticas de sucesso, taxa de geocodificação e dias necessários

**16:11 - Criação do api/rotas.py**
- Endpoint principal para Vercel Functions (AWS Lambda compatible)
- Funcionalidades implementadas:
  - Handler POST para recebimento de lote de PDFs
  - Suporte a payload JSON com PDFs em base64
  - Validação de limites (máximo 50 PDFs)
  - Orquestração completa: extração → geocodificação → otimização
  - Tratamento de erros com códigos HTTP apropriados
  - Headers CORS para permitir chamadas de frontend
  - Formato de resposta padronizado com sucesso/mensagem/dados/estatísticas
  - Função adaptadora para compatibilidade com Vercel

**16:11 - Criação do requirements.txt**
- Dependências mínimas especificadas:
  - PyMuPDF>=1.23.0 (para extração de PDF)
  - requests>=2.28.0 (para chamadas à OpenRouteService)

**16:12 - Criação do vercel.json**
- Configuração para deploy na plataforma Vercel
- Build configurado para usar @vercel/python
- Rota definida: /api/* → api/rotas.py
- Preparado para receber variável de ambiente OPENROUTESERVICE_API_KEY

**16:13 - Criação do README.md**
- Documentação completa do sistema
- Seções incluídas:
  - Visão geral da arquitetura
  - Descrição de cada módulo
  - Instruções de uso com exemplo de payload
  - Formato de resposta detalhado
  - Variáveis de ambiente necessárias
  - Limitações e considerações técnicas
  - Instruções para deploy na Vercel

**16:14 - Ajustes Finais no route_optimizer.py**
- Correção crítica na lógica de retorno à base:
  - Adicionado tempo explícito de conferência (30 minutos) após chegada à base
  - Verificação atualizada: retorna como "dentro do limite" apenas se o fim da conferência ocorrer antes das 16:30
  - Campo "fim_conferencia" adicionado às paradas de retorno para rastreabilidade
  - Tempo de serviço no retorno configurado como TEMPO_RETORNO_BASE_MIN (30 min)

---

### 📊 Resumo do Trabalho Concluído

✅ **8 arquivos fonte criados** além de documentação  
✅ **Arquitetura modular** separada por responsabilidades claras  
✅ **Total conformidade com requisitos**:
  - Recebe lote de até 50 PDFs de Notas Fiscais
  - Extrai endereço de destinatário/remetente sem armazenamento PDF
  - Utiliza OpenRouteService para geocodificação
  - Otimiza sequência de paradas (TSP) com restrição de horário crítica
  - Regra de negócio implementada: retorno à base para conferência antes das 16:30
  - Diluição automática para próximo dia útil quando necessário
  - Pula fins de semana automaticamente
✅ **Pronto para deploy serverless** na Vercel Functions  
✅ **Documentação completa** incluindo guia de uso e exemplo de integração  

---

### 🔧 Próximos Passos Recomendados

1. **Configurar variável de ambiente** na Vercel:
   - `OPENROUTESERVICE_API_KEY`: Obter chave gratuita em https://openrouteservice.org/dev/#/signup

2. **Testar com dados reais**:
   - Preparar amostras de PDFs de Notas Fiscais para validação
   - Verificar precisão da extração de endereços nos modelos específicos utilizados

3. **Monitorar uso da API gratuita**:
   - OpenRouteService tem limites diários na camada gratuita
   - Considerar implementação de cache simples para endereços repetidos

4. **Possíveis melhorias futuras**:
   - Substituir TSP guloso por algoritmo mais ótimo (ex: otimização 2-opt)
   - Adicionar suporte a janelas de tempo de entrega específicas
   - Implementar retry automático para falhas temporárias de geocodificação
   - Adicionar relatório de endereços não geocodificados para revisão manual

---

**Registro finalizado às 16:15 de 28/04/2026**  
Sistema pronto para validação e deploy conforme especificação original.

---

### 📅 28/04/2026 - Continuação: Tarefas 1 e 2

**16:16 - Tarefa 1: Criação do SQL DDL para Supabase**
- Arquivo: `supabase_schema.sql`
- Funcionalidades implementadas:
  - Extensão PostGIS habilitada para suporte geográfico
  - Tabela `entregas` criada com todos os campos necessários:
    * Identificação: id (UUID), pdf_origem, numero_nf, serie_nf
    * Dados originais: dados_extraidos (JSONB para auditoria)
    * Geolocalização: localizacao (GEOGRAPHY PostGIS), latitude, longitude
    * Status de geocodificação: geocodificado, tentativas, erros, confiança
    * Agendamento: data_agendada_entrega, turno_entrega, secuencia_rota, rota_id
    * Controle de status: EXTRAIDO, GEOCODIFICADO, ROTEIRIZADO, AGENDADO, CONCLUIDO, FALHA
  - Índices criados para performance (GIST para geolocalização, status, datas)
  - Trigger automático para atualização de timestamp (atualizado_em)
  - Comentários detalhados para documentação do schema

**16:18 - Tarefa 2: Script Python Principal com Maintenance-Zero**
- Arquivo: `main.py`
- Foco em "Maintenance-Zero" (sem estado persistente, processamento efêmero)
- Funcionalidades implementadas:
  - Tratamento robusto de erros em todas as etapas
  - Verificação de feriados brasileiros usando biblioteca `holidays`
  - Função `verificar_dia_util()` que considera: Seg-Sex + não feriado nacional
  - Função `proximo_dia_util()` que pula fins de semana E feriados automaticamente
  - Separação clara de endereços com sucesso vs falha na geocodificação
  - Estatísticas detalhadas de processamento
  - Modo CLI para processamento rápido: `python main.py <entrada.json> [saida.json]`
  - Tratamento de falhas individuais sem interromper o lote
  - Código de saída apropriado (0 mesmo com falhas parciais)

**16:19 - Atualização do requirements.txt**
- Adicionada dependência: `holidays>=0.26` para suporte a feriados brasileiros

**16:20 - Criação do supabase_integration.py (Opcional)**
- Módulo para integração com Supabase (caso deseje persistência)
- Classe `SupabaseClient` com métodos:
  - `salvar_entregas()`: Insere dados na tabela entregas
  - `buscar_entregas_pendentes()`: Recupera entregas não processadas
  - `atualizar_status_entrega()`: Atualiza status individual
- Função `testar_conexao_supabase()` para validação inicial
- Nota: Para o sistema "efêmero", este módulo é opcional

**16:21 - Atualização do CIADOS.MD**
- Registro detalhado de todas as atividades das Tarefas 1 e 2
- Documentação das decisões arquiteturais e técnicas

---

### 📊 Resumo Atualizado (16:21)

✅ **11 arquivos criados** (incluindo SQL e scripts Python)  
✅ **Tarefa 1 concluída**: SQL DDL para Supabase com geolocalização (PostGIS)  
✅ **Tarefa 2 concluída**: Script Python principal "Maintenance-Zero"  
✅ **Tratamento de feriados brasileiros** implementado via biblioteca `holidays`  
✅ **Tratamento de erros robusto** em todas as etapas do pipeline  
✅ **Separação clara** de sucessos vs falhas na geocodificação  

---

### 🔍 Destaques Técnicos das Tarefas 1 e 2

**SQL DDL (Tarefa 1):**
- Tipo `GEOGRAPHY(POINT, 4326)` do PostGIS para armazenar coordenadas geográficas
- Índice espacial GiST para consultas geográficas rápidas
- Campo `dados_extraidos` como JSONB para flexibilidade na auditoria
- Status tracking completo do ciclo de vida da entrega

**Script Python (Tarefa 2):**
- "Maintenance-Zero": Não mantém estado entre execuções
- Feriados brasileiros: Usa `holidays.Brazil()` que inclui feriados nacionais
- Pula sábado, domingo E feriados ao calcular próximo dia útil
- Relatório detalhado de endereços não encontrados para revisão manual
- Código de saída 0 para sucesso (mesmo parcial), facilitando automação

---

**Registro atualizado às 16:21 de 28/04/2026**  
Sistema agora possui persistência opcional (Supabase), tratamento completo de feriados e script de execução standalone.

---

### 📅 28/04/2026 - Criação de Exemplos e Testes

**16:22 - Criação de exemplo_entrada.json**
- Arquivo de exemplo mostrando o formato esperado de entrada
- Contém 2 PDFs mockados em base64 (estrutura mínima válida)
- Serve como template para usuários prepararem seus payloads

**16:23 - Criação de test_system.py**
- Suíte de testes básicos para validar o sistema
- Testes implementados:
  - `test_imports()`: Verifica se todos os módulos importam corretamente
  - `test_config()`: Valida configurações (horários, base, limites)
  - `test_holidays()`: Testa feriados brasileiros (Natal, Ano Novo, Tiradentes, etc.)
  - `test_haversine()`: Valida cálculo de distância (Sorocaba-SP ≈ 87km)
  - `test_route_optimizer()`: Testa TSP com pontos fictícios
  - `test_pdf_extractor_mock()`: Verifica carregamento do módulo
- Execução: `python test_system.py`
- Retorna código 0 (sucesso) ou 1 (falha) para automação

**16:24 - Atualização do requirements.txt**
- Adicionada dependência: `supabase>=2.0.0` para integração opcional

**16:25 - Criação de GUIA_USO.md**
- Guia prático e completo de uso do sistema
- Seções incluídas:
  - Pré-requisitos e instalação local
  - Configuração de variáveis de ambiente
  - Modo 1: Execução local via CLI com exemplo passo-a-passo
  - Modo 2: Deploy na Vercel (serverless) com instruções detalhadas
  - Modo 3: Integração opcional com Supabase
  - Testes do sistema com exemplo de saída esperada
  - Diagrama do fluxo completo de processamento
  - Regras de negócio implementadas (resumo)
  - Troubleshooting para erros comuns
  - Próximos passos recomendados

---

### 📊 Resumo Final das Tarefas (16:25)

✅ **14 arquivos criados** no total  
✅ **Tarefa 1 concluída**: SQL DDL para Supabase com PostGIS  
✅ **Tarefa 2 concluída**: Script Python "Maintenance-Zero" com feriados  
✅ **Exemplo prático** de entrada criado  
✅ **Suíte de testes** automatizada criada  
✅ **Guia de uso completo** documentado  
✅ **Tratamento de erros** robusto em todas as etapas  
✅ **Feriados brasileiros** implementados via biblioteca `holidays`  
✅ **Documentação** completa (README, GUIA_USO, CIADOS)  

---

### 🗂️ Estrutura Final do Projeto

```
LOGISTICA - ROTA/
├── config.py                    # Configurações centralizadas
├── main.py                     # Script principal CLI (Maintenance-Zero)
├── test_system.py              # Testes automatizados
├── supabase_integration.py     # Integração opcional com Supabase
├── supabase_schema.sql         # DDL SQL para Supabase + PostGIS
├── requirements.txt            # Dependências Python
├── vercel.json                 # Configuração para deploy Vercel
├── README.md                   # Documentação geral
├── GUIA_USO.md                 # Guia prático de uso
├── CIADOS.MD                   # Registro detalhado de atividades
├── exemplo_entrada.json        # Exemplo de payload de entrada
├── api/
│   └── rotas.py               # Endpoint Vercel Functions
└── services/
    ├── pdf_extractor.py       # Extração de endereços de PDFs
    ├── geocoder.py            # Geocodificação OpenRouteService
    └── route_optimizer.py     # Otimização TSP + restrições horário
```

---

**Registro finalizado às 16:25 de 28/04/2026**  
Sistema completo, testado (estruturalmente) e documentado. Pronto para:
1. Receber chave da OpenRouteService
2. Testes com PDFs reais de Notas Fiscais
3. Deploy na Vercel (se desejado)
4. Configuração opcional do Supabase

---

### 📅 28/04/2026 - Refinamento da Lógica de Cálculo de Tempo

**16:30 - Refinamento do config.py**
- Atualização de parâmetros conforme novas regras de negócio:
  - Velocidade média: 40 km/h → **60 km/h** (urbano/rodoviário misto)
  - Tempo de entrega: 10 min → substituído por `TEMPO_DESCARGA_MIN = 15` (fixo por nota)
  - Tempo de retorno à base mantido em 30 min
- Comentários atualizados para refletir os novos valores

**16:32 - Refinamento do services/route_optimizer.py**
- **Nova lógica de cálculo de janela de tempo:**
  - Para cada parada N, calcula: `Tempo_Total = Tempo_Acumulado + Deslocamento(N-1→N) + 15min(Descarga) + Deslocamento(N→Base)`
  - Se `Tempo_Total` ultrapassar 16:30, a parada N é movida para o próximo dia útil
- **Implementação da biblioteca holidays:**
  - Configurada para feriados nacionais e estaduais de SP: `holidays.Brazil(state='SP')`
  - Função `proximo_dia_util()` pula fins de semana E feriados (nacionais + SP)
- **Código idempotente:**
  - Mesma entrada sempre produz mesma saída
  - Cálculos baseados em datas determinísticas
- **Função `simular_rota_com_tempo()` refatorada:**
  - Recebe `data_base` como parâmetro para cálculos reproduzíveis
  - Retorna lista de paradas processadas e lista de excedentes
  - Verificação de janela feita antes de confirmar cada parada
- **Função `dividir_rota_por_dia()` atualizada:**
  - Processa endereços restantes recursivamente para próximos dias úteis
  - Respeita feriados de SP em cada iteração
- **Tratamento de erro robusto:** Excedentes são preservados para reprogramação

**16:35 - Validação das Alterações**
- Lógica de cálculo alinhada com requisitos:
  - Velocidade 60 km/h implementada
  - 15 minutos fixos de descarga por nota
  - Janela de verificação (N-1→N + descarga + N→Base) funcional
  - Corte às 16:30 respeitado
  - Feriados nacionais e de SP sendo pulados corretamente
- Código limpo e idempotente garantido

---

### 📊 Resumo da Atualização (16:35)

✅ **config.py atualizado**: Velocidade 60 km/h, descarga 15 min  
✅ **route_optimizer.py refinado**: Nova lógica de janela de tempo  
✅ **Biblioteca holidays**: Feriados nacionais + SP implementados  
✅ **Código idempotente**: Comportamento determinístico garantido  
✅ **Validação de janela**: Corte às 16:30 com reprogramação automática  

---

**Registro atualizado às 16:35 de 28/04/2026**  
Lógica de cálculo de tempo refinada conforme especificações. Sistema agora calcula janelas precisas e pula feriados de SP.