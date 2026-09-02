# Chunking

Este módulo implementa estratégias de chunking para processamento de documentos em corpus de texto. O sistema divide documentos em chunks menores para processamento posterior, suportando múltiplas estratégias de divisão.

## Arquivos

### chunk_strategies.py
Implementa as três estratégias principais de chunking:
- **chunk_fixed_window**: Divide documentos em janelas fixas de caracteres com sobreposição configurável
- **chunk_full_document**: Mantém cada documento inteiro como um único chunk
- **chunk_hierarchical_semantic**: Divide documentos de forma hierárquica baseada em estrutura semântica (FAQs por perguntas/respostas, documentos técnicos por seções H2)

Funções auxiliares incluem geração de IDs de chunks, resolução de fontes e injeção de cabeçalhos padronizados.

### lambda_function.py
Handler AWS Lambda que orquestra o pipeline completo de chunking em ambiente serverless. Pipeline:
1. Lê documentos JSONL do S3
2. Aplica as 3 estratégias de chunking
3. Padroniza payloads de saída
4. Escreve chunks no S3 em formato JSONL

Pode ser disparado por eventos S3 ou invocações manuais.

### payload_formatter.py
Responsável pela padronização dos payloads de saída das estratégias de chunking. Garante que todo chunk tenha:
- chunk_id formatado no padrão `{doc_family_id}_v{version_ordinal}_{strategy}_{chunk_index:03d}`
- Metadados completos incluindo strategy e chunk_index
- Estrutura consistente entre todas as estratégias

### process_chunks.py
Script para processamento local de chunks. Contém funções para:
- Ler arquivos JSONL do disco local
- Transformar estrutura de documentos do formato de ingestão para o formato esperado pelas estratégias
- Aplicar as 3 estratégias de chunking
- Escrever chunks em formato JSONL localmente

Pode ser executado via linha de comando com caminhos customizados.

### reader.py
Módulo de leitura de arquivos JSONL do S3. Implementa:
- Leitura de objetos S3
- Parse de JSONL linha por linha
- Handler Lambda para leitura disparada por eventos S3

### s3_writer.py
Módulo de escrita de chunks no S3. Funções:
- Conversão de lista de chunks para formato JSONL
- Upload de chunks para bucket S3
- Configuração de Content-Type apropriado

### upload_chunks_to_s3.py
Script utilitário para fazer upload dos 3 arquivos de chunks locais para o S3. Suporta:
- Upload dos arquivos gerados por process_chunks.py
- Configuração via variáveis de ambiente ou argumentos de linha de comando
- Logging detalhado do processo de upload

### __init__.py
Arquivo de inicialização do pacote Python (vazio).

## Tecnologias Utilizadas

- **Python 3**: Linguagem principal do projeto
- **AWS Lambda**: Funções serverless para execução do pipeline
- **AWS S3**: Armazenamento de documentos e chunks
- **boto3**: SDK da AWS para interação com serviços AWS
- **JSONL (JSON Lines)**: Formato de serialização (um JSON por linha)
- **Regex**: Processamento de texto para identificação de padrões (seções H2, perguntas em negrito)
- **logging**: Sistema de logging estruturado para monitoramento

## Estratégias de Chunking

### Fixed Window
Divide conteúdo em janelas fixas de caracteres (padrão: 500) com sobreposição (padrão: 100). Útil para processamento genérico onde a estrutura semântica não é crítica.

### Full Document
Mantém cada documento inteiro como um único chunk. Ideal para documentos curtos ou quando contexto completo é necessário.

### Hierarchical Semantic
Divide documentos baseado em estrutura semântica:
- FAQs: Divide por pares Pergunta/Resposta (identificadas por texto em negrito)
- Documentos técnicos: Divide por seções H2, respeitando integridade de tabelas e listas numeradas